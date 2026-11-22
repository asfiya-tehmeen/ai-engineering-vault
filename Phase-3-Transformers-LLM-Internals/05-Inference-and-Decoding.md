---
tags: [phase-3, inference, decoding, kv-cache]
---
# 05 — Inference, Decoding & Serving

> This note explains **latency, cost, rate limits, context-length pricing, prompt caching** — i.e., everything about how you'll *use* LLMs in production.

## 1. Autoregressive generation
```
prompt tokens → [PREFILL: process all prompt tokens in parallel] → first logits
loop: sample token → append → [DECODE: one token at a time, reuse cache] → … until EOS/max_tokens/stop
```
- **Prefill** is **compute-bound** (big matmuls, parallel over prompt) → determines **time-to-first-token (TTFT)**.
- **Decode** is **memory-bandwidth-bound**: each new token must read *all weights* (and the KV cache) from GPU memory for tiny compute → determines **time-per-output-token (TPOT)** / tokens/sec.
- Rule of thumb: decode speed per sequence ≈ `memory_bandwidth / bytes_read_per_token`. A 70B bf16 model (140 GB) on ~3 TB/s HBM ≈ ≤ ~20 tok/s per GPU-equivalent for a single sequence → **batching** is what makes serving economical.

## 2. The KV cache
Without caching, generating token `t` recomputes K,V for all previous tokens (O(T²) total recompute). **Cache K and V per layer** and only compute them for the new token.
**Size:** `KV bytes = 2 · L · n_kv_heads · d_head · T · bytes_per_elem · batch`
Example (LLaMA-3-8B-like: L=32, kv_heads=8, d_head=128, bf16): `2·32·8·128·2 = 131,072 B ≈ 128 KB/token` → 8k context ≈ 1 GB per sequence; 128k ≈ 16 GB per sequence. **This is why GQA/MLA/quantized KV/sliding windows matter, and why long-context requests cost more and cut concurrency.**
- KV cache memory, not compute, often caps **batch size/concurrency**.
- **Prefix caching:** if many requests share a prompt prefix (system prompt, tools, documents), reuse its KV → skip prefill. **This is exactly what API "prompt caching" is** (Phase 4): cheaper & faster cached tokens. Design prompts with **stable prefix first, variable content last**.
- **KV-cache quantization** (fp8/int8/4-bit), **eviction/compression** (H2O, SnapKV, streaming), **offloading** to CPU/disk/remote.

> [!warning] API reality check (added in Phase 4)
> Everything in this section is how sampling works *inside the model/serving stack* and is directly controllable on open-weight models (vLLM, Ollama, llama.cpp) and many providers. **However, the Claude API has removed `temperature`/`top_p`/`top_k` on current models** (non-default values return a 400; Anthropic's docs say requests sending them 400 on every model from 2026-10-21), and reasoning models at other providers also restrict them. For hosted APIs, steer behavior with prompts, effort/reasoning settings, and structured outputs instead — see [[../Phase-4-Building-with-LLM-APIs/01-API-Fundamentals]].

## 3. Decoding strategies (turning logits into tokens)
Logits `z` → probabilities `softmax(z/T)`.
| Method | How | Effect / use |
|---|---|---|
| **Greedy** | argmax | deterministic-ish, repetitive; good for extraction/classification |
| **Temperature T** | divide logits by T | T→0 sharper (greedy), T>1 flatter/more random |
| **Top-k** | sample from k most likely | fixed cutoff |
| **Top-p (nucleus)** | smallest set with cumulative prob ≥ p | adaptive cutoff (p≈0.9–0.95) |
| **Min-p** | keep tokens with prob ≥ p_min × p_max | scales with confidence; robust at high T |
| Typical / eta / epsilon sampling | information-theoretic cutoffs | |
| **Repetition/frequency/presence penalties** | down-weight seen tokens | reduce loops |
| **Beam search** | keep top-b sequences | good for translation/ASR, poor for open-ended chat (generic text) |
| **Best-of-N / majority vote / reranking** | sample N, pick by judge/verifier/vote | quality ↑, cost × N |
| **Contrastive / speculative / lookahead** | efficiency or quality tweaks | |
- **Non-determinism:** even at T=0, outputs can vary due to floating-point non-associativity, batching, MoE routing, kernel selection. Don't assume bit-exact reproducibility; use seeds where offered and test for variance.
- **Logprobs** (some APIs): per-token confidence → classification scores, uncertainty heuristics, calibration, detecting guesses.
- Some reasoning APIs fix or restrict sampling params; read provider docs.

## 4. Constrained / structured decoding
Mask invalid tokens at each step so output **must** match a grammar/JSON Schema/regex (Outlines, XGrammar, llguidance, lm-format-enforcer; provider "structured outputs" features). Guarantees *syntactic* validity, not *semantic* correctness. Slight quality tradeoffs possible; schema design matters (Phase 4).

## 5. Speeding up inference
| Technique | Idea |
|---|---|
| **FlashAttention (1/2/3)** | IO-aware tiled attention; avoids materializing T×T; big speed/memory win |
| **Continuous (in-flight) batching** | add/remove sequences from the running batch every step → high GPU utilization |
| **PagedAttention (vLLM)** | KV cache in fixed-size blocks (like OS paging) → no fragmentation, prefix sharing |
| **Chunked prefill** | interleave prefill chunks with decode to protect latency |
| **Speculative decoding** | small *draft* model (or MTP/Medusa/EAGLE heads) proposes k tokens; big model verifies in one parallel pass; accept matching prefix → same output distribution, 2–3× faster decode |
| **Quantization** | weights int8/FP8/4-bit (GPTQ, AWQ, GGUF k-quants, bitsandbytes NF4, FP4/MXFP4); activations & KV cache too; trade tiny quality for 2–4× memory/speed |
| **Distillation / smaller models** | cheapest speedup |
| **MoE** | fewer active params per token |
| **Model/tensor/pipeline parallel** | fit big models across GPUs |
| **Disaggregated prefill/decode** | separate GPU pools tuned for each phase |
| **Prompt/prefix & semantic caching** | skip repeated work |
| **Torch.compile / CUDA graphs / custom kernels (Triton, CUTLASS)** | reduce overhead |

## 6. Serving stacks
- **vLLM** (PagedAttention, continuous batching, prefix caching, chunked prefill, quantization, speculative decoding, distributed) — most popular open engine.
- **SGLang** (RadixAttention prefix reuse; strong for agents, structured outputs, repeated context).
- **TensorRT-LLM / Triton / NVIDIA Dynamo** (NVIDIA-optimized), **TGI** (HF), **llama.cpp / Ollama / LM Studio / MLX** (local, CPU/Apple Silicon, GGUF), **Ray Serve**, cloud-managed (Bedrock, Vertex, Azure AI Foundry, Together, Fireworks, Baseten, Modal, Groq/Cerebras/SambaNova specialized hardware).
- Expose **OpenAI-compatible APIs** so apps swap backends easily.

## 7. Metrics that matter
**TTFT**, **TPOT / inter-token latency**, **end-to-end latency** (≈ TTFT + TPOT×output_tokens), **throughput** (tokens/s/GPU), **goodput** (throughput meeting latency SLOs), **p50/p95/p99**, **concurrency**, **cost per million tokens**, **cache hit rate**, **GPU utilization/MFU**, **queue time**.
Trade-off triangle: **latency ↔ throughput ↔ cost**.

## 8. Cost & latency estimation recipes
- **Cost/request** = `in_tokens·price_in + cached_in·price_cached + out_tokens·price_out (+ thinking tokens billed as output)`. Output tokens cost several × input tokens; **reasoning tokens are output tokens**.
- **Latency** ≈ network + queue + TTFT(prompt_len) + out_tokens × TPOT. Cutting **output length** is the biggest latency lever.
- **Streaming** doesn't reduce total time but improves perceived latency.
- **Self-hosting breakeven:** GPU $/hr ÷ (tokens/s × 3600) vs API price; utilization is everything (idle GPUs destroy economics).
- **Memory fit:** weights (N×bytes) + KV cache (per-seq × concurrency) + activations + overhead ≤ VRAM. Example: 70B at 4-bit ≈ 35 GB weights + KV.

## 9. Hardware primer
GPUs: NVIDIA A100/H100/H200/B200 class (HBM bandwidth & capacity are key), consumer RTX for local; AMD MI300-series; Google TPUs; AWS Trainium/Inferentia; Groq/Cerebras (SRAM/wafer-scale, very fast decode). Interconnect: NVLink/NVSwitch, InfiniBand. Apple Silicon unified memory → good local inference (MLX). **Memory capacity & bandwidth dominate LLM inference.**

## 10. Run one locally (do this in week 4)
```bash
# Ollama
ollama run llama3.2        # or any small model from the library
# vLLM (Linux + NVIDIA GPU)
pip install vllm && vllm serve Qwen/Qwen2.5-1.5B-Instruct   # OpenAI-compatible server at :8000
```
```python
from openai import OpenAI            # works against local servers
client = OpenAI(base_url="http://localhost:8000/v1", api_key="x")
```
Then measure TTFT/TPOT while varying batch/concurrency, prompt length, quantization.

## Self-check
1. Why is decode memory-bound? 2. Compute KV cache for a given config. 3. How does speculative decoding preserve the output distribution? 4. What is prefix caching and how should you structure prompts to benefit? 5. Why do output tokens cost more than input tokens? 6. Why can T=0 still be non-deterministic?

Next → [[06-Context-Capabilities-and-Limits]]
