---
tags: [phase-3, pretraining, scaling]
---
# 03 — Pretraining & Scaling Laws

## 1. The objective
**Next-token prediction** (autoregressive LM): maximize `Σₜ log P(xₜ | x<ₜ)` ⇒ minimize cross-entropy. Self-supervised: the text *is* the label. Training on trillions of tokens makes the model compress the distribution of human text → knowledge, syntax, reasoning heuristics.
- Metrics: loss/perplexity/bits-per-byte on held-out data; downstream benchmark suites.
- Alternatives: masked LM (BERT), span corruption (T5), **fill-in-the-middle (FIM)** for code, multi-token prediction.

## 2. Data (where most of the quality comes from)
Pipeline for a web-scale corpus:
1. **Sources:** web crawl (Common Crawl), books, code (GitHub), academic papers, Wikipedia, forums, math, multilingual, synthetic.
2. **Extraction & cleaning:** HTML→text, language ID, boilerplate removal.
3. **Quality filtering:** heuristics (length, symbol ratio), classifiers (educational value; e.g., FineWeb-Edu style), perplexity filters.
4. **Dedup:** exact + near-dup (MinHash/LSH), sub-document dedup → reduces memorization & improves efficiency.
5. **Safety/PII/toxicity filtering; copyright & licensing considerations.**
6. **Decontamination** vs benchmarks.
7. **Mixing & curriculum:** domain proportions (web/code/math), upsampling high-quality data, **annealing** on high-quality data at the end of training, long-context extension stage.
8. **Tokenization & packing:** concatenate documents with EOS, pack into fixed-length sequences; document masking to avoid cross-document attention.
- Open data recipes to study: FineWeb / FineWeb-Edu, DCLM, Dolma, RedPajama, The Stack, OLMo/Nemotron-CC data pipelines.
- **Data > architecture** for quality; "data-centric" work is the main lever of open labs.

## 3. Training recipe (typical)
- Optimizer **AdamW** (β₁=0.9, β₂≈0.95, wd≈0.1), **grad clip 1.0**, LR warmup then cosine or **WSD (warmup-stable-decay)**, large global batch (millions of tokens, often ramped), **bf16** (FP8 increasingly at scale), no dropout (single-epoch, data-rich).
- **Initialization & scaling rules:** μP / µTransfer to transfer hyperparameters from small proxies to large models.
- **Training loop at scale:** thousands of GPUs, weeks–months, checkpoints, failure recovery (hardware faults are routine), loss-spike handling (rewind & skip data, lower LR).

## 4. Parallelism (how big models train)
| Strategy | Splits | Notes |
|---|---|---|
| **Data parallel (DP/DDP)** | batch | replicate model |
| **ZeRO / FSDP** | optimizer states, grads, params | memory-efficient DP |
| **Tensor parallel (TP)** | each matmul across GPUs | needs fast interconnect (NVLink) |
| **Pipeline parallel (PP)** | layers across GPUs | bubbles; micro-batching |
| **Sequence/context parallel** | sequence dimension | long context (ring attention) |
| **Expert parallel (EP)** | MoE experts | all-to-all comms |
"3D/4D parallelism" combines them. Libraries: Megatron-LM, DeepSpeed, PyTorch FSDP2/torchtitan, NeMo, JAX/MaxText. **MFU** (model FLOPs utilization; 35–55% is good) measures efficiency.

## 5. Back-of-the-envelope math (learn to do this fast)
- **Params** `N` (see Note 02).
- **Training FLOPs** ≈ `6·N·D` (D = tokens). (2ND forward + 4ND backward.) For MoE use *active* params.
- **Time** ≈ FLOPs / (GPUs × peak FLOPs × MFU). Example: 8B model × 15T tokens → 7.2×10²³ FLOPs; at 400 TFLOP/s effective ×(~16k GPUs equivalent)… know the order of magnitude: millions of GPU-hours.
- **Training memory** (mixed precision + Adam) ≈ **16–20 bytes/param** + activations (before sharding).
- **Inference weights** ≈ `N × bytes/param` (bf16=2, int8=1, 4-bit=0.5).
- **Cost** ≈ GPU-hours × $/GPU-hour.

## 6. Scaling laws
- **Kaplan et al. (2020):** loss falls as a power law in params, data, compute: `L(N) ∝ N^−α`.
- **Chinchilla (Hoffmann et al., 2022):** for a fixed compute budget, params and tokens should scale together — roughly **~20 tokens per parameter** was compute-*optimal for training*. Earlier giants (GPT-3 175B on 300B tokens) were undertrained.
- **Overtraining (inference-optimal) regime:** because you train once but serve billions of requests, labs train *smaller* models on *far more* tokens than Chinchilla-optimal (e.g., open 8B-class models trained on ~15T tokens ≈ ~1,900 tokens/param; 100× Chinchilla ratio) to get a cheaper-to-serve model at a given quality. Loss still improves, with diminishing returns.
- **Data-constrained scaling:** high-quality text is finite. Repeating data up to ~4 epochs is nearly as good as fresh; beyond that returns decay. 2026 work proposes extensions of Chinchilla with repetition/overfitting terms; synthetic-token terms are proposed but not settled. Treat claims about the "data wall" as contested.
- **Other scaling axes:** *post-training/RL compute* and **test-time (inference) compute** (reasoning models) have their own scaling behavior — the main frontier axis since 2024–25.
- **Emergent abilities:** capabilities that seem to appear suddenly with scale; later work shows many "emergences" are artifacts of discontinuous metrics (use continuous metrics). Real capability jumps (in-context learning, chain-of-thought) still occur.
- **Limits of laws:** extrapolation errors, loss ≠ capabilities, data quality shifts the curve.

## 7. Precision & numerics
fp32 → fp16 (loss scaling needed) → **bf16** (same exponent range as fp32; standard) → **FP8** (H100+; per-tensor/block scaling; DeepSeek-V3 popularized FP8 training) → FP4/MXFP4 emerging. Master weights & optimizer states stay higher precision. **Loss spikes** from bad data, LR too high, attention logit growth (fix: QK-norm, z-loss, lower β₂).

## 8. Long-context & other training stages
Stage 1 (short context, most tokens) → **context extension** (RoPE scaling + continued training on long docs) → **annealing** on curated data → then post-training ([[04-Post-Training-and-Alignment]]). **Mid-training** (reasoning/code/math-heavy data injections before post-training) is a 2025–26 trend.

## 9. Synthetic data
LLM-generated textbooks/Q&A/code/reasoning traces (Phi series, Nemotron, Cosmopedia), rephrasing web data, rejection-sampled verified solutions. Risks: **model collapse** (training on own outputs degrades diversity/tails), amplified errors/bias, benchmark leakage. Mitigation: verification, diversity, mixing with real data.

## 10. Memorization, contamination, licensing
Models memorize rare duplicated sequences; extraction attacks exist; PII/copyright issues → dedup, filtering, legal review. **Benchmark contamination** inflates scores — prefer private/held-out/live evals (Phase 7).

## 11. Who trains what (landscape — verify live)
Closed frontier labs (OpenAI, Anthropic, Google DeepMind, xAI…), open-weight (Meta Llama, DeepSeek, Qwen/Alibaba, Mistral, Google Gemma, OpenAI gpt-oss, Moonshot Kimi, Zhipu GLM, Microsoft Phi, Ai2 OLMo (fully open data/code), NVIDIA Nemotron, Hugging Face SmolLM). Trackers: Raschka's LLM Architecture Gallery, Epoch AI, LMArena, Artificial Analysis (links in [[10-Resources-and-Practice-Links]]).

## Self-check
1. Estimate FLOPs & GPU-hours for 7B on 2T tokens. 2. Why do open labs overtrain? 3. What is MFU? 4. Why bf16 over fp16? 5. What does dedup buy you? 6. Why can't we just keep scaling web text?

Next → [[04-Post-Training-and-Alignment]]
