---
tags: [phase-3, projects, checkpoint]
---
# 11 — Projects & Phase Checkpoint

> Repos need a README with **Problem · Approach · Results (with variance) · What broke · What I'd do next**. Log runs (W&B). Publish at least Projects 1, 3, and 5.

## Project 1 — `bpe-lab`: tokenizer from scratch *(Note 01)*
Byte-level BPE with regex pre-tokenization, special tokens, save/load, and a CLI. Compare with tiktoken on 1000 samples (token-for-token where possible). Add a **cost explorer** web page/CLI: paste text → token counts across 3 tokenizers + estimated $ at configurable prices. Write up multilingual token inflation with a chart.

## Project 2 — `tinyformer`: GPT from scratch + modern upgrades *(Notes 02, 07)*
- Phase A: char-level GPT (Shakespeare).
- Phase B: BPE + a real corpus (FineWeb-Edu shard / TinyStories). Report **bits-per-byte**.
- Phase C (upgrades with ablations, 3 seeds each): RMSNorm, RoPE, SwiGLU, GQA, QK-norm; plot loss vs steps & wall-clock.
- Phase D: KV-cache generation + sampling suite (temp/top-k/top-p/min-p).
- Phase E: load GPT-2 weights, match logits.
- Deliverables: clean repo, tests (shapes, causality, cache-equivalence), results table, blog-style write-up.

## Project 3 — `llm-calc`: back-of-envelope engine *(Notes 03, 05)*
Web/CLI calculator: input a model config (or HF model ID → fetch `config.json`) → outputs **parameters (total/active), weights memory by dtype, training FLOPs/time/cost, KV cache vs context & concurrency, max concurrency per GPU, estimated tok/s (bandwidth bound), $/M tokens self-hosted vs API**. Validate against 5 real models' published specs and measured vLLM runs. (This is a genuine FDE tool for sizing customer deployments.)

## Project 4 — Post-training lab *(Note 04)*
1. SFT a small open model on a narrow task (e.g., customer-support triage JSON extraction) with LoRA; measure before/after with a held-out eval.
2. DPO on preference pairs (tone/format/safety) — show what changed and what regressed.
3. **GRPO on a verifiable toy task** (arithmetic/countdown/unit-test code): plot reward, length, entropy; document at least one reward-hacking incident and your fix.
4. Compare to the "just prompt a bigger model" baseline on cost/quality. Honest conclusion: when is fine-tuning worth it?

## Project 5 (Capstone) — `inference-bench`: serve, measure, optimize *(Notes 05, 06)*
- Serve one small open model on **vLLM** (and optionally SGLang/llama.cpp/Ollama).
- Benchmark TTFT, TPOT, throughput, p95 across concurrency (1→64), prompt lengths (128→8k), output lengths, **quantization levels**, prefix caching on/off, speculative decoding on/off.
- Produce a report with plots, the "knee" of throughput vs latency, cost per million tokens vs an API price, and **recommendations** for two customer scenarios (low-latency chat vs high-throughput batch).
- Include a **long-context study**: lost-in-the-middle & multi-needle accuracy vs position/length for one API model and one open model.
- This mirrors real FDE work: sizing, benchmarking, and justifying an infra choice with data.

## Phase 3 Self-assessment (0–3; need ≥2 before Phase 4)
| Skill | 0–3 |
|---|---|
| Hand-run BPE; explain tokenization-caused failures | |
| Count tokens/cost for a prompt; explain cross-provider differences | |
| Write causal multi-head attention from memory with shapes | |
| Explain RoPE, RMSNorm, SwiGLU, GQA/MLA, MoE and why each exists | |
| Count parameters & active parameters of a config | |
| Compute training FLOPs/memory & inference memory/KV cache | |
| Explain Chinchilla vs overtraining | |
| Describe the pretraining data pipeline & its levers | |
| Explain SFT → RLHF/DPO → RLVR with losses | |
| Explain GRPO & reward hacking | |
| Explain prefill vs decode; why decode is memory-bound | |
| Explain continuous batching, paged attention, prefix caching, speculative decoding | |
| Choose sampling params for a task | |
| Explain hallucination causes & mitigations | |
| Explain long-context limits & context rot | |
| Pick a model with a defensible process | |
| Train + sample a GPT end-to-end | |
| Run & benchmark a local inference server | |

## Mock interview (60 min)
1. 10 min — "Explain what happens from API call to first token, in detail." (tokenize → prefill → KV cache → decode loop → sampling → stream)
2. 15 min — implement causal self-attention (or top-p sampling) live; explain shapes & mask.
3. 15 min — "Customer wants to self-host a 70B model for 200 concurrent users with 8k contexts. Size it." (weights, KV cache, GPUs, batching, quantization, latency targets)
4. 10 min — "Our fine-tuned model got worse; debug." (template, masking, LR, data, eval leakage)
5. 10 min — "Explain to a CFO why output tokens cost more and how to cut the bill."

## What you've unlocked
You now understand the engine well enough to *predict* its behavior: cost, latency, failure modes, and where prompts/tools/retrieval must compensate. Phase 4 puts it to work through APIs: prompting, structured outputs, tool calling, streaming, caching, and context engineering.

⏸️ **Pause point.** Say the word and I'll build **Phase 4 — Building with LLM APIs** (I'll research the current Anthropic/OpenAI/Google API features first: structured outputs, tool use, prompt caching, batch, extended thinking, multimodal, context engineering).

Back to [[00-Phase-3-Index]] · [[../00-Roadmap]]
