---
tags: [phase-3, exercises]
---
# 08 — Exercises (Phase 3)

🟢 easy · 🟡 medium · 🔴 hard. Keep code in `phase3-exercises/`. Verify against reference implementations (tiktoken, HF Transformers, PyTorch) wherever possible.

## A. Tokenization (Note 01)
- [ ] 🟢 A1. Use tiktoken (`o200k_base`, `cl100k_base`) to tokenize: English paragraph, Hindi paragraph, Python code, JSON, numbers (`1234567`). Table tokens/char; explain the differences.
- [ ] 🟢 A2. Show `"hello"`, `" hello"`, `"Hello"`, `"HELLO"` are different tokens.
- [ ] 🟡 A3. Hand-run BPE on `"aaabdaaabac"` for 3 merges; then verify with your code.
- [ ] 🟡 A4. Implement byte-level BPE `train / encode / decode` (≈100 lines); roundtrip test on Unicode, emoji, code.
- [ ] 🟡 A5. Add GPT-style regex pre-tokenization & special tokens; match tiktoken on 1000 samples (or explain mismatches).
- [ ] 🟡 A6. Explain via tokens why LLMs fail at "how many r's in strawberry". Test 3 models; try adding spaces between letters.
- [ ] 🔴 A7. Train tokenizers (vocab 4k/16k/64k) on a corpus; plot compression (bytes/token) vs vocab; compute embedding-matrix param cost.
- [ ] 🔴 A8. Find or construct "glitch tokens" (rare tokens) in a vocab by looking for tokens with unusually low embedding norm/usage in a small model.
- [ ] 🔴 A9. Build a prompt-cost estimator: given text + model, return token count and $ using tiktoken/provider count API; compare to billed usage on 10 API calls.

## B. Attention & the Transformer (Note 02)
- [ ] 🟢 B1. Compute attention by hand for T=3, d=2 (given Q,K,V); verify with NumPy. Show the causal mask effect.
- [ ] 🟢 B2. Plot attention heatmaps from a pretrained small model (GPT-2) for 3 sentences; identify a previous-token head or induction-like pattern.
- [ ] 🟡 B3. Implement scaled dot-product attention in NumPy and PyTorch (no `scaled_dot_product_attention`); match `F.scaled_dot_product_attention` to 1e-5.
- [ ] 🟡 B4. Show why √d_k matters: compute softmax saturation/gradient norm with and without scaling at d=64, 512, 4096.
- [ ] 🟡 B5. Implement multi-head attention with explicit `view/transpose`; annotate every shape.
- [ ] 🟡 B6. Implement RoPE; test that `q_m · k_n` depends only on `m−n`.
- [ ] 🟡 B7. Implement RMSNorm and SwiGLU; swap into your GPT; compare loss curves.
- [ ] 🟡 B8. Parameter counter: given `(L, D, h, kv_heads, H, V, tied?)` return params; check against HF `model.num_parameters()` for 3 real configs.
- [ ] 🔴 B9. Implement GQA and MQA; measure KV-cache size and decode speed vs MHA.
- [ ] 🔴 B10. Implement sliding-window attention mask + one global layer pattern; verify receptive field growth across layers.
- [ ] 🔴 B11. Implement a top-2 MoE layer with load-balancing loss; show expert utilization histograms; compare active vs total params.
- [ ] 🔴 B12. Implement FlashAttention's online-softmax (tiled) in pure PyTorch; match naive attention numerically; explain memory savings.
- [ ] 🔴 B13. Reproduce an **induction head** experiment: repeat random token sequences and show loss on the 2nd repetition drops sharply for a small trained transformer.

## C. Build & train GPT (Note 07)
- [ ] 🟢 C1. Train the char-level GPT; report train/val loss, sample at T=0.2/0.8/1.2.
- [ ] 🟡 C2. Ablations (each run 3 seeds): no residual, no LayerNorm, no positional embeddings, 1 vs 6 layers, dropout 0 vs 0.2. Write what broke and why.
- [ ] 🟡 C3. Overfit one batch; verify initial loss ≈ ln(V).
- [ ] 🟡 C4. Switch to BPE (tiktoken GPT-2) and a larger corpus; report bits-per-byte instead of loss to compare across tokenizers.
- [ ] 🟡 C5. Implement gradient accumulation, cosine LR+warmup, checkpoint/resume; show resumed run is identical.
- [ ] 🔴 C6. Load GPT-2 (124M) weights from Hugging Face into your model; match logits; then generate.
- [ ] 🔴 C7. Reproduce GPT-2 124M training on a FineWeb-Edu shard (follow build-nanogpt); evaluate HellaSwag.
- [ ] 🔴 C8. Add `torch.compile`, bf16, fused AdamW, Flash attention; report tokens/sec and MFU.

## D. Pretraining & scaling (Note 03)
- [ ] 🟢 D1. Write `llm_math.py`: params, training FLOPs (6ND), GPU-hours at a given MFU, weights memory by dtype, Adam training memory.
- [ ] 🟡 D2. Compute Chinchilla-optimal tokens for 1B/7B/70B; compare with actual token counts of 3 public open models (from their reports); compute tokens/param.
- [ ] 🟡 D3. Train 5 model sizes (0.5M–20M params) on the same data, plot loss vs params and vs compute on log–log axes; fit a power law.
- [ ] 🟡 D4. Show effect of data dedup: train on corpus with/without duplicates; compare val loss & memorization (exact-copy rate in samples).
- [ ] 🔴 D5. Quality-filter experiment: train a tiny classifier to score "educational value", filter a corpus, train two models, compare benchmarks.
- [ ] 🔴 D6. Estimate the cost of training a hypothetical 70B on 15T tokens (GPU count, weeks, $). List assumptions.
- [ ] 🔴 D7. Reproduce a loss spike (high LR, no clipping) and fix with clipping/QK-norm/lower β₂.

## E. Post-training (Note 04)
- [ ] 🟡 E1. SFT a 0.5B–1.5B model (e.g., Qwen2.5-0.5B) on ~1–5k instruction examples with TRL + LoRA; compare before/after on 20 held-out prompts.
- [ ] 🟡 E2. Verify assistant-only loss masking: print which tokens contribute to loss.
- [ ] 🟡 E3. Implement the Bradley–Terry reward-model loss; train an RM on a small preference dataset (e.g., UltraFeedback/HH subset); report pairwise accuracy.
- [ ] 🔴 E4. Implement DPO loss from scratch (log-prob of chosen/rejected under policy & reference); train; compare with TRL's DPOTrainer.
- [ ] 🔴 E5. GRPO toy: train a small model on an arithmetic/countdown task with a programmatic reward; plot reward, response length, and entropy; document reward hacking you observe.
- [ ] 🔴 E6. Best-of-N vs RL: for the same model, compare pass@1 after RL with best-of-16 from the base; discuss.
- [ ] 🔴 E7. Distill reasoning traces from a stronger model into a small one via SFT; evaluate on held-out problems.

## F. Inference & decoding (Note 05)
- [ ] 🟢 F1. Implement temperature, top-k, top-p, min-p sampling from logits; visualize the truncated distributions on a real next-token distribution.
- [ ] 🟢 F2. KV-cache calculator for 5 real model configs at 8k/32k/128k context.
- [ ] 🟡 F3. Implement KV caching in your GPT; verify equal logits; plot tokens/sec vs sequence length with/without cache.
- [ ] 🟡 F4. Measure TTFT & TPOT of a local model (Ollama/vLLM) vs prompt length and output length; fit `latency ≈ a + b·in + c·out`.
- [ ] 🟡 F5. Load-test vLLM with increasing concurrency; plot throughput vs p95 latency (find the knee).
- [ ] 🟡 F6. Quantize a model (GGUF/AWQ/bnb 4-bit); compare memory, speed, and quality on 50 prompts + perplexity.
- [ ] 🔴 F7. Implement speculative decoding with a small draft model (greedy case); verify identical outputs to the target; report speedup vs acceptance rate.
- [ ] 🔴 F8. Implement a simple continuous-batching scheduler simulation (requests arriving over time) and compare with static batching on throughput/latency.
- [ ] 🔴 F9. Implement constrained decoding: mask logits so output matches a regex / small JSON grammar; compare validity with unconstrained sampling.
- [ ] 🔴 F10. Demonstrate non-determinism at T=0 (batch-size changes) and explain.

## G. Context & limits (Note 06)
- [ ] 🟢 G1. Build 10 hallucination probes (obscure facts, fake citations, nonexistent APIs); run on 2 models; categorize failures.
- [ ] 🟡 G2. Lost-in-the-middle experiment: place a fact at 10 positions in a 10k–30k-token context; plot accuracy vs position for one API model.
- [ ] 🟡 G3. Needle-in-haystack vs multi-needle aggregation task; compare difficulty; plot vs context length.
- [ ] 🟡 G4. Few-shot ablation: 0/1/3/8 examples, shuffled orders, label-balance changes; quantify sensitivity with CIs.
- [ ] 🟡 G5. CoT vs no-CoT vs reasoning-model on 30 multi-step problems; compute accuracy, tokens, cost, latency.
- [ ] 🔴 G6. Write a **model-selection memo** for a hypothetical customer: requirements → 3 candidate models → eval results → cost/latency → recommendation & risks.
- [ ] 🔴 G7. Test faithfulness of CoT: insert a biasing hint in the prompt, check whether the stated reasoning mentions it when it changes the answer.

## H. Interview coding set (use [[10-Resources-and-Practice-Links]] sites)
Attention · multi-head attention · causal mask · positional encodings · LayerNorm/RMSNorm · softmax with temperature · top-p sampling · beam search · BPE · KV cache · cross-entropy/perplexity · LoRA layer · cosine similarity search · rotary embedding · transformer block · MoE router.

## Cadence
3 exercises/week + the GPT build + weekly Q&A review ([[09-QA-Interview-Questions]]).
