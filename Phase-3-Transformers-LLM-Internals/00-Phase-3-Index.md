---
tags: [phase-3, index]
---
# Phase 3 — Transformers & LLM Internals

> [!abstract] Why this phase exists
> You'll spend your career *using* LLMs through APIs. The engineers who debug well, estimate cost/latency, choose models, and design around failure are the ones who know **what's inside**: tokens, attention, KV cache, sampling, post-training. This phase makes the model a glass box.

**Duration:** 5–7 weeks · **Prereq:** [[../Phase-2-ML-Deep-Learning/00-Phase-2-Index|Phase 2]] (backprop, PyTorch, cross-entropy, AdamW)

> [!warning] A note on freshness
> The *fundamentals* here (tokenization, attention, KV cache, pretraining → post-training) are stable. The *model landscape* (names, sizes, benchmark leaders) changes monthly. I've marked fast-moving items and point to live trackers rather than freezing numbers. Verify any specific model claim at the sources in [[10-Resources-and-Practice-Links]].

## Topic map
| # | Note | Core ideas |
|---|---|---|
| 01 | [[01-Tokenization]] | BPE, byte-level, vocab, special tokens, chat templates, tokenizer pitfalls, cost |
| 02 | [[02-Transformer-Architecture]] | attention, multi-head, MQA/GQA/MLA, RoPE, norms, SwiGLU, residual stream, MoE, sliding window, hybrids/SSMs |
| 03 | [[03-Pretraining-and-Scaling]] | objective, data pipeline, scaling laws, Chinchilla vs overtraining, parallelism, precision, instabilities, data wall |
| 04 | [[04-Post-Training-and-Alignment]] | SFT, RLHF, reward models, PPO, DPO, GRPO/RLVR, reasoning models, distillation, safety |
| 05 | [[05-Inference-and-Decoding]] | prefill/decode, KV cache math, sampling, batching, paged attention, speculative decoding, quantization, serving engines |
| 06 | [[06-Context-Capabilities-and-Limits]] | context windows, in-context learning, hallucination, reasoning, multimodal, benchmarks, open vs closed, choosing models |
| 07 | [[07-Build-GPT-From-Scratch]] | full nanoGPT-style implementation walkthrough |
| 08 | [[08-Exercises]] | 70+ exercises |
| 09 | [[09-QA-Interview-Questions]] | 60 Q&A |
| 10 | [[10-Resources-and-Practice-Links]] | papers, courses, trackers, practice |
| 11 | [[11-Projects-and-Checkpoint]] | 5 projects + self-assessment |

## Weekly plan (6 weeks)
| Week | Focus | Output |
|---|---|---|
| 1 | Note 01 + Karpathy tokenizer video | BPE tokenizer from scratch (minbpe-style) |
| 2 | Note 02 + Illustrated Transformer + Annotated Transformer | attention & transformer block in PyTorch, shape-annotated |
| 3 | Note 07 + Karpathy "Let's build GPT" | train a char-level GPT; then BPE GPT on a real corpus |
| 4 | Note 03 + 05 | back-of-envelope calculators (params, FLOPs, memory, KV cache); KV-cache implementation |
| 5 | Note 04 | SFT a small model; DPO run; GRPO toy on arithmetic |
| 6 | Note 06 + exercises + Q&A | capstone + model-selection memo |

## Completion checklist
- [ ] Tokenize text by hand with BPE; explain why "strawberry" counting fails and why numbers/code/other languages cost more tokens
- [ ] Write multi-head causal self-attention from memory, with correct shapes and masking
- [ ] Explain MQA/GQA/MLA, RoPE, RMSNorm, SwiGLU, MoE routing and *why* each exists
- [ ] Compute params, training FLOPs (6ND), inference memory and KV-cache size for a given config
- [ ] Explain Chinchilla vs overtraining and why open models train on far more tokens than "optimal"
- [ ] Explain pretraining → SFT → preference tuning (RLHF/DPO) → RLVR reasoning training, with the loss for each
- [ ] Explain prefill vs decode, why decode is memory-bandwidth-bound, and how batching/paging/speculation help
- [ ] Explain temperature/top-p/min-p and when to use each
- [ ] Explain why hallucination happens structurally; list mitigations
- [ ] Train a small GPT end-to-end and sample from it
- [ ] Capstone shipped on GitHub

Back to [[../00-Roadmap]].
