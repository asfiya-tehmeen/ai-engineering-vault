---
tags: [phase-9, quantization, distillation, moe, architectures, hardware]
---
# 02 — Efficient Models: Quantization, Distillation, MoE & New Architectures

> **Source note:** my search returned solid material on **NVFP4 and quantization-aware distillation (QAD)** (NVIDIA/Red Hat, early 2026; vendor-reported numbers) and a **30B-A3B hybrid-MoE** model example, but **nothing on diffusion LMs and little on SSM/Mamba trends**. Treat those parts as established-concept summaries, not current-state reports. Builds on [[../Phase-3-Transformers-LLM-Internals/02-Transformer-Architecture]] and [[../Phase-3-Transformers-LLM-Internals/05-Inference-and-Decoding]].

## 1. Why efficiency is the frontier (for engineers)
Inference dominates lifetime cost for deployed models; decode is **memory-bandwidth-bound**; memory capacity limits model/context size; energy and hardware supply constrain scale. Every technique here trades **quality ↔ memory ↔ speed ↔ complexity**. Your job: know the lever, its failure modes, and how to **evaluate the served artifact**.

## 2. Hardware mental model: the roofline
- **Arithmetic intensity** = FLOPs ÷ bytes moved. A kernel is **compute-bound** if intensity > the hardware's *ridge point* (peak FLOPs ÷ memory bandwidth), else **memory-bound**.
- Modern data-center GPUs have ridge points on the order of **hundreds of FLOPs/byte** (e.g., ~1 PFLOP/s dense bf16 vs ~3–5 TB/s HBM → ridge ≈ 200–300; *verify specs for your part*).
- **Prefill** (big matmuls) is usually compute-bound; **decode** (matrix–vector per token, batch small) has intensity ~O(1–10) → **memory-bound** → speed ≈ bandwidth ÷ bytes read per token. This is *why* quantizing weights (fewer bytes) speeds decode almost linearly, and why batching helps.
```python
def decode_tokens_per_sec(params_B, bytes_per_param, hbm_TBps, kv_bytes_per_token=0, ctx=0, batch=1):
    bytes_read = params_B*1e9*bytes_per_param + kv_bytes_per_token*ctx*batch     # weights + KV per step
    return batch * (hbm_TBps*1e12) / bytes_read                                  # aggregate tokens/s upper bound
# 70B @ bf16 (2 B) on 3.35 TB/s, batch 1 ≈ 24 tok/s; @ 4-bit (0.5 B) ≈ 96 tok/s (upper bounds, ignoring overhead)
```
- **FlashAttention** is IO-aware tiling: keep Q/K/V blocks in on-chip SRAM, compute softmax online (running max/sum) → avoids materializing the T×T matrix. Know the **online softmax** trick (Phase 3 exercise B12).
- **Kernel languages:** CUDA/CUTLASS (max control), **Triton** (Python-like GPU kernels), compilers (`torch.compile`/Inductor, XLA, TVM). Learn to read a profile (nsys/torch profiler) before writing kernels.

## 3. Quantization
### 3.1 Basics
Map floats to low-bit integers: `q = round(x/s + z)`, `x̂ = s(q − z)`. **Scale/zero-point** per tensor, per channel, or per group (e.g., 64–128 weights). **Symmetric** (z=0) vs **asymmetric**. Error ≈ `s²/12` per element — shrink `s` by grouping and by handling outliers.
```python
import numpy as np
def quantize_int8_per_channel(W):                         # W: (out, in)
    s = np.abs(W).max(axis=1, keepdims=True) / 127.0 + 1e-12
    q = np.clip(np.round(W / s), -127, 127).astype(np.int8)
    return q, s
def dequantize(q, s): return q.astype(np.float32) * s
# Try per-tensor vs per-channel vs per-group; add an outlier column (×50) and watch per-tensor error explode.
```
### 3.2 What to quantize
| Target | Notes |
|---|---|
| **Weights only (W8A16, W4A16)** | biggest memory + decode speed win; activations stay bf16/fp16 |
| **Weights + activations (W8A8, FP8, W4A4/NVFP4)** | faster matmuls on supporting hardware; activation outliers make this harder |
| **KV cache (int8/FP8/4-bit)** | cuts long-context memory; test long-context quality |
| **Optimizer states/gradients (8-bit optimizers)** | training memory |
### 3.3 Methods (post-training quantization, PTQ)
- **LLM.int8()** — mixed-precision decomposition for **outlier features** (a few channels with huge magnitude dominate error).
- **GPTQ** — layer-wise, uses approximate second-order info (Hessian) to minimize output error when rounding weights.
- **AWQ** — protects "salient" weight channels identified by **activation magnitudes**; per-channel scaling.
- **SmoothQuant** — migrates activation outliers into weights via per-channel scaling so W8A8 works.
- **GGUF k-quants / bitsandbytes NF4** — practical formats for local/QLoRA (NF4 = 4-bit NormalFloat, designed for normally distributed weights).
- **Rotations/Hadamard transforms** (QuaRot/SpinQuant-style) spread outliers before quantizing.
### 3.4 Low-precision floating point
- **FP8** (E4M3/E5M2): used for inference and increasingly **training** (per-tensor/block scaling). **FP4 variants** — **MXFP4** (microscaling: shared scale per block of 32) and **NVFP4** (NVIDIA; small blocks with higher-precision scale factors) — target next-gen accelerators.
- **Reported (vendor/Red Hat, early 2026):** NVFP4 PTQ recovers roughly **95–98% of BF16 accuracy** on 7B–14B models across tasks, with MoE models (e.g., large Qwen/Llama-4-class) described as robust; hardware claims of up to ~2× FLOPs utilization on Blackwell tensor cores. *Vendor numbers — verify on your tasks.*
### 3.5 QAT and QAD
- **Quantization-aware training (QAT):** simulate quantization in the forward pass with **straight-through estimators**, fine-tune so weights adapt. Strong but needs training data/compute; can be fragile with multi-stage post-training (SFT→RL→merging).
- **Quantization-aware distillation (QAD)** (NVIDIA, Jan 2026): train the **quantized student to match the full-precision teacher's output distribution (KL loss)** instead of re-running the original post-training objectives; reported to recover near-BF16 accuracy on models that went through multi-stage pipelines, where plain QAT struggles. Concept to remember: **distillation as a way to recover accuracy after compression.**
### 3.6 Evaluating quantized models (don't skip)
Compare on **your task evals**, not just perplexity: long-context, tool-use/JSON validity, multilingual, safety/refusals, code. Test on the **served engine** (kernel/numerics differ). Check **outlier-sensitive** layers and **degradation under long generation**. Report speed *and* quality together.

## 4. Pruning & sparsity
Unstructured pruning (zero out weights; needs sparse kernels), **structured pruning** (remove heads/channels/layers; real speedups), **2:4 semi-structured sparsity** (hardware-supported on some GPUs), layer dropping/merging, and **prune + distill** recipes (e.g., compress a large model into a smaller family via pruning followed by distillation). Gains are real but often smaller than quantization for the engineering effort; distillation recovers quality.

## 5. Distillation (deep dive)
- **Objective:** match teacher's **soft targets**: `L = α·CE(y, p_s) + (1−α)·T²·KL(p_t^T ‖ p_s^T)` (temperature `T`); for LLMs, **sequence-level** (SFT on teacher generations) or **token-level KL on logits**.
- **Variants:** response distillation (black-box teachers — check provider ToS), **logit/feature distillation** (open teachers), **on-policy distillation** (student samples, teacher scores/corrects — reduces train–test mismatch; e.g., GKD/MiniLLM-style), **multi-teacher**, **self-distillation** (compress a long-reasoning mode into short), **reasoning distillation** (R1-style traces).
- **What transfers well:** format, style, skills present in the teacher's outputs; **limits:** small students can't absorb everything; capability gaps remain on rare knowledge and hardest reasoning.
- **Practice:** filter teacher outputs by graders/tests; balance difficulty; avoid contaminating eval sets; evaluate on held-out *and* real-traffic.

## 6. Mixture-of-Experts (MoE) in depth
- **Router:** `g(x) = softmax(W_r x)`; **top-k** experts per token; combine expert outputs with gate weights. Variants: sigmoid gating, **shared experts** (always-on), **fine-grained experts** (many small), expert-choice routing.
- **Load balancing:** auxiliary loss (`N·Σ f_i·P_i` where `f_i` = fraction of tokens to expert i, `P_i` = mean router prob) to prevent **expert collapse**; or **auxiliary-loss-free** bias adjustment (DeepSeek-V3-style); capacity factors/token dropping.
- **Parallelism:** **expert parallel** (experts across GPUs; all-to-all communication is a bottleneck); inference needs enough **batch** to keep experts busy.
- **Trade-offs:** total params ≫ active params (e.g., "30B-A3B" ≈ 30B total, ~3B active) → large capacity per FLOP but **memory must hold all experts**; routing instability; harder fine-tuning/quantization; communication costs; specialization is more syntactic/token-level than "topic experts."
- **Training tricks:** router z-loss, jitter/noise, expert dropout; upcycling dense → MoE.
- **Hybrid MoE** (MoE + attention variants + SSM layers) appears in recent open releases.

## 7. Beyond standard attention (concepts)
- **Linear attention & kernelized/recurrent forms:** replace `softmax(QKᵀ)V` with feature maps so attention becomes a recurrent state update `S_t = S_{t−1} + φ(k_t) v_tᵀ` → **O(T)** time, **fixed-size state** (no growing KV cache). Quality gap historically; improved with **gating/delta rules** (e.g., DeltaNet-style updates).
- **State-space models (S4, Mamba/Mamba-2):** input-dependent ("selective") recurrences computed with parallel scans; strong long-sequence efficiency; weaker at exact recall/copying/in-context retrieval than attention.
- **Hybrids:** interleave a few attention layers with many SSM/linear layers to get **fixed-state efficiency + attention's precise retrieval** — an active direction in 2025–26 releases.
- **Sparse/sliding-window + global attention**, **MLA** (compressed KV), **attention sinks**, **ring attention** for sequence parallel ([[../Phase-3-Transformers-LLM-Internals/02-Transformer-Architecture]]).
- **Memory-augmented/test-time-trained layers** (e.g., layers that update parameters at inference to store context) — research frontier ([[05-Data-Continual-Learning-and-Open-Problems]]).
- **Diffusion language models:** generate/refine many tokens in parallel by iterative denoising (masked/discrete diffusion) — potential for faster generation and editing; quality/serving maturity relative to autoregressive models is **unsettled** (my search returned nothing current — verify before relying on any claim).
- **Multi-token prediction (MTP)** and **speculative decoding** variants (Medusa/EAGLE) to speed decode.

## 8. Efficient training notes
bf16/FP8 mixed precision; activation checkpointing; ZeRO/FSDP; sequence/context parallel; MoE expert parallel; **MFU** as the efficiency metric; data-loading bottlenecks; **muP/hyperparameter transfer**; optimizer choices (AdamW vs Lion/Muon-family — evaluate carefully); **WSD schedules**; low-precision training stability (loss spikes, QK-norm, z-loss).

## 9. Lab ideas (do at least two)
1. **Quantization bake-off:** FP16 vs INT8 vs INT4 (GPTQ/AWQ/bnb-NF4/GGUF) on a 1–8B model: memory, tok/s, perplexity, **and** your task suite; analyze outlier layers.
2. **Per-tensor vs per-channel vs per-group** NumPy study with synthetic outliers; plot error vs group size.
3. **QAD toy:** quantize a small model to 4-bit; recover quality via KL distillation from the bf16 teacher; compare with plain QAT/PTQ.
4. **Distillation:** teacher → student (response-level vs logit-level vs on-policy) on a narrow task; cost/quality table.
5. **MoE from scratch:** top-2 router + load-balancing loss in your tinyformer; visualize expert utilization and collapse without the aux loss.
6. **Roofline calculator:** predict tok/s for several models/GPUs; validate against measurements with vLLM.
7. **Linear attention vs softmax attention:** implement both; test recall tasks (needle/copy) to see the retrieval gap; try a tiny hybrid.

## Self-check
1. Explain why decode is memory-bound using the roofline. 2. Per-tensor vs per-group quantization — why does group size matter with outliers? 3. What problem do SmoothQuant/AWQ/LLM.int8 each address? 4. Describe QAD vs QAT. 5. Write the MoE load-balancing loss and explain expert collapse. 6. Why do hybrids combine SSM/linear layers with attention?

Next → [[03-RL-for-LLMs-and-Alignment-Frontier]]
