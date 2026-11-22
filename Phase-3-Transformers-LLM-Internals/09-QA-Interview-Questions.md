---
tags: [phase-3, qa, interview]
---
# 09 — Q&A (Phase 3)

> Answer aloud, then expand.

## Tokenization
> [!question]- Q1. What is BPE and why byte-level?
> Iteratively merge the most frequent adjacent symbol pair to build a subword vocabulary. Byte-level starts from the 256 bytes, so any Unicode string is representable (no unknown tokens) and the vocab stays bounded.

> [!question]- Q2. Why do LLMs struggle with counting letters or reversing words?
> They operate on subword tokens, not characters; the character composition isn't directly visible, and the model must have memorized spelling of each token.

> [!question]- Q3. Why does non-English text often cost more?
> Tokenizers trained mostly on English learn fewer merges for other scripts, so the same content takes more tokens → higher cost, less usable context, and sometimes worse quality.

> [!question]- Q4. What's a chat template and why does it matter?
> The exact serialization of roles/messages/special tokens the model was post-trained on. Using the wrong one degrades instruction-following, especially with open models.

> [!question]- Q5. Trade-off in vocab size?
> Larger vocab → shorter sequences and cheaper attention but bigger embedding/output matrices and sparser rare-token training.

## Architecture
> [!question]- Q6. Explain self-attention in plain terms.
> Each token forms a query, compares against all tokens' keys to get weights (softmax), and takes a weighted average of their values — a learned, content-based lookup over the context.

> [!question]- Q7. Why scale by √d_k?
> Dot product variance grows with `d_k`; unscaled scores push softmax into saturation with vanishing gradients. Dividing by √d_k keeps variance ~1.

> [!question]- Q8. Why a causal mask and how does training stay parallel?
> The mask blocks attention to future tokens so each position predicts the next token only from the past; all positions are computed in one pass (teacher forcing), unlike generation which is sequential.

> [!question]- Q9. Why multiple heads?
> Different heads can specialize in different relations (syntax, position, copying) in separate subspaces at the same total cost as one big head.

> [!question]- Q10. MHA vs MQA vs GQA vs MLA?
> They differ in KV sharing: MHA one KV per query head; MQA one KV for all; GQA groups of query heads share KV (best balance, widely used); MLA compresses KV into a low-rank latent to shrink the cache while keeping quality. All target smaller KV cache/faster decode.

> [!question]- Q11. How does RoPE work and why is it popular?
> It rotates query/key coordinate pairs by position-dependent angles so their dot product depends on relative offset. It encodes relative position without extra parameters and supports context extension (interpolation/YaRN).

> [!question]- Q12. Why Pre-LN/RMSNorm/SwiGLU?
> Pre-LN stabilizes deep training; RMSNorm is cheaper with similar quality; gated activations (SwiGLU) empirically improve quality per parameter.

> [!question]- Q13. Where are most parameters? What do MLPs do?
> In the MLPs (~⅔ per block); they act like key–value memories storing factual and feature transformations per position, while attention routes information across positions.

> [!question]- Q14. Explain Mixture of Experts and its trade-offs.
> A router sends each token to top-k of many expert MLPs; total params are large but active params per token are small. Pros: capacity per FLOP. Cons: must keep all experts in memory, routing/load balancing complexity, all-to-all communication, training instability.

> [!question]- Q15. Why is attention O(T²) a problem and what are the fixes?
> Compute/memory grow quadratically with sequence length. Fixes: FlashAttention (memory), sliding-window/sparse attention, GQA/MLA (cache), linear/state-space hybrids, ring attention for distribution, retrieval.

> [!question]- Q16. Encoder-only vs decoder-only vs encoder-decoder?
> Encoder-only (BERT): bidirectional representations for understanding/embeddings. Decoder-only (GPT): causal generation, now dominant. Encoder-decoder (T5): conditional seq2seq with cross-attention.

> [!question]- Q17. Estimate parameter count of a transformer.
> ≈ `12·L·D²` plus embeddings `V·D` (for 4D MLP, MHA). With gated MLP hidden `H`: per layer `4D² + 3DH`.

> [!question]- Q18. What is weight tying?
> Sharing the input embedding matrix with the output projection; saves V·D parameters and often helps small models.

## Pretraining & scaling
> [!question]- Q19. What's the pretraining objective and why is it powerful?
> Next-token cross-entropy on huge text. To predict well it must model syntax, facts, discourse, and reasoning patterns present in data; labels are free so it scales.

> [!question]- Q20. Training FLOPs formula?
> ≈ `6·N·D` (N params, D tokens; active params for MoE).

> [!question]- Q21. What is Chinchilla scaling and why do open models violate it?
> For fixed compute, balance params and tokens (~20 tokens/param) to minimize loss. Labs overtrain smaller models on far more tokens since inference cost across many requests dominates training cost, making smaller high-quality models cheaper overall.

> [!question]- Q22. Data quality steps in pretraining?
> Extraction, language ID, heuristic & classifier quality filtering, exact/near dedup, PII/safety filtering, decontamination, domain mixing/upsampling, annealing on high-quality data.

> [!question]- Q23. Why bf16 over fp16?
> bf16 keeps fp32's exponent range (no loss-scaling headaches/overflow) with lower mantissa precision, which training tolerates.

> [!question]- Q24. What causes loss spikes and fixes?
> Bad batches, high LR, attention logit growth/numerical issues. Fixes: grad clipping, lower LR/β₂, QK-norm, z-loss, data skipping & rollbacks, better init.

> [!question]- Q25. Explain data/tensor/pipeline parallelism.
> DP splits batches (FSDP/ZeRO shard states), TP splits matrices within a layer, PP splits layers into stages; modern training combines them plus expert/sequence parallelism.

> [!question]- Q26. What's model collapse?
> Degradation when models are trained recursively on their own outputs — loss of diversity/tails and accumulated errors; mitigate with real data mixing and filtering.

> [!question]- Q27. Are "emergent abilities" real?
> Some sharp transitions are artifacts of discontinuous metrics, but qualitative capability changes with scale (ICL, CoT) are real; evaluate with continuous metrics.

## Post-training
> [!question]- Q28. SFT vs RLHF — what does each add?
> SFT teaches format/behavior from demonstrations; RLHF/preference optimization shapes responses toward what humans prefer (helpfulness, tone, safety) by optimizing a reward with KL regularization.

> [!question]- Q29. Explain the reward model loss.
> Bradley–Terry: maximize `σ(r(x,y_w) − r(x,y_l))`, i.e., minimize `−log σ(r_w − r_l)`, so the preferred response scores higher.

> [!question]- Q30. Why the KL penalty in RLHF?
> Keeps the policy near the reference/SFT model to avoid reward hacking and degeneration into outputs that exploit RM flaws.

> [!question]- Q31. How does DPO avoid RL?
> It reparameterizes the KL-regularized RLHF objective so the optimal policy's implicit reward is `β log(π/π_ref)`; this gives a supervised logistic loss over preference pairs using policy and reference log-probs.

> [!question]- Q32. What is GRPO and why used for reasoning?
> Sample a group of responses per prompt, reward each (often verifiable), normalize rewards within the group as advantages, and update with a clipped policy gradient + KL. No value network, so cheaper and simpler than PPO; good for RLVR.

> [!question]- Q33. What is RLVR and its limits?
> RL with programmatically verifiable rewards (math answers, unit tests). Limits: only for checkable domains, reward hacking via loopholes, sparse reward, costly rollouts, entropy collapse, unclear generalization to non-verifiable tasks.

> [!question]- Q34. What is reward hacking—examples?
> Policy exploits proxy rewards: verbosity/flattery to please RMs, hard-coding unit-test outputs, editing tests, exploiting environment bugs, special-casing answers.

> [!question]- Q35. What is sycophancy and why does it arise?
> Tendency to agree with the user's stated views; arises because human raters/rewards favor agreeable responses. Mitigate with data/rubrics, adversarial evals, prompting.

> [!question]- Q36. What is distillation?
> Train a smaller student on a teacher's outputs (text or logits) to transfer capability cheaply; on-policy variants have the teacher grade student samples.

> [!question]- Q37. Why do reasoning models help, and what's the cost?
> They spend extra tokens on serial reasoning/self-verification, improving multi-step tasks; cost: higher latency and output-token spend, risk of overthinking and unfaithful traces.

## Inference
> [!question]- Q38. Prefill vs decode?
> Prefill processes the prompt in parallel (compute-bound, sets TTFT); decode generates one token per step reusing KV cache (memory-bandwidth-bound, sets tokens/sec).

> [!question]- Q39. Compute KV-cache size.
> `2 × layers × kv_heads × head_dim × seq_len × bytes × batch`. E.g., 32 layers, 8 KV heads, 128 dim, bf16 → 128 KB/token.

> [!question]- Q40. Why does batching improve cost efficiency?
> Decode reads all weights once per step regardless of batch size, so more concurrent sequences amortize that memory traffic → higher tokens/s per GPU (until KV memory/latency limits).

> [!question]- Q41. What's prompt/prefix caching and how to exploit it?
> Reuse KV of identical prefixes across requests to skip prefill; put static content (system prompt, tools, documents) first and dynamic content last; avoid changing early tokens.

> [!question]- Q42. Explain speculative decoding.
> A cheap draft proposes several tokens; the target verifies them in one forward pass and accepts the longest matching prefix (rejection-sampling rule preserves the target distribution), yielding speedups when acceptance is high.

> [!question]- Q43. Top-k vs top-p vs min-p vs temperature?
> Temperature rescales logits; top-k keeps k tokens; top-p keeps smallest cumulative-mass set; min-p keeps tokens above a fraction of the top probability. Truncation methods cut the unreliable tail; use low T for factual/extraction, higher for creative.

> [!question]- Q44. Why is output more expensive than input?
> Output tokens are generated sequentially in memory-bound decode steps, while input tokens are processed in parallel at high compute efficiency; capacity and latency costs differ.

> [!question]- Q45. What does quantization trade?
> Lower precision weights/KV reduce memory and bandwidth (faster, cheaper, fits bigger models) at small, task-dependent quality loss; outliers and low-bit regimes need careful methods (AWQ/GPTQ, FP8).

> [!question]- Q46. What are PagedAttention and continuous batching?
> Paged KV cache blocks avoid fragmentation and enable sharing; continuous batching inserts/removes sequences each step so GPUs stay busy despite varying lengths.

> [!question]- Q47. How does structured/constrained decoding work?
> At each step, compute allowed tokens given a grammar/schema state and mask all others, ensuring syntactically valid output; doesn't ensure truthfulness.

> [!question]- Q48. Why might T=0 outputs differ between runs?
> Floating-point non-associativity with different batching/kernels, MoE routing, hardware/versions → tiny logit differences flip argmax near ties.

## Context, limits, practice
> [!question]- Q49. What is "lost in the middle"?
> Models utilize information at the start/end of long contexts better than the middle; mitigations: placement, retrieval/reranking, shorter cleaner context.

> [!question]- Q50. Why do LLMs hallucinate?
> They model plausible text, not truth; rewarded for answering; long-tail facts poorly learned; no native uncertainty mechanism; SFT on unknown facts encourages guessing. Mitigate with grounding, tools, abstention, verification, evals.

> [!question]- Q51. How does few-shot prompting work without training?
> The transformer performs in-context learning: patterns in the prompt condition its next-token distribution (induction-like circuits), effectively adapting behavior at inference time.

> [!question]- Q52. When would you choose open-weight over API?
> Data cannot leave the environment, need customization/fine-tuning, predictable high-volume cost, latency control, offline/edge, or licensing/vendor-risk reasons — if you can staff the ops.

> [!question]- Q53. How do you pick a model for a task?
> Define task & constraints → build an eval set → test candidates from strongest to cheapest → compare quality/latency/cost/privacy → consider routing, caching, fine-tuning → monitor and re-evaluate as models change.

> [!question]- Q54. Why are public benchmarks insufficient?
> Contamination, saturation, harness differences, mismatch with your domain/format; use private task-specific evals.

> [!question]- Q55. Is chain-of-thought a faithful explanation?
> Not necessarily; it can be a post-hoc narrative that omits actual influences; monitor behavior and verify outputs independently.

> [!question]- Q56. How does a multimodal LLM ingest an image?
> A vision encoder splits the image into patches → embeddings → projector/adapter → tokens fed into the LLM alongside text (token count scales with resolution).

> [!question]- Q57. Estimate memory to serve a 70B model at 4-bit with 32k context, 16 concurrent users.
> Weights ≈ 35 GB; KV per token (GQA, ~80 layers × 8 kv × 128 × 2 × bytes) at bf16 ≈ ~320 KB → 32k ≈ 10 GB/seq → 16 seqs ≈ 160 GB → KV dominates; need KV quantization, fewer concurrent seqs, or multi-GPU. (Show your arithmetic and assumptions — interviewers want the method.)

> [!question]- Q58. What would you check first if a fine-tuned open model behaves worse than the base?
> Chat template/tokenizer mismatch, loss masking, learning rate too high/forgetting, data quality/duplicates, over-training epochs, eval contamination, special tokens/EOS handling, LoRA merge errors, decoding params.

> [!question]- Q59. Why can't we just stuff everything in a 1M-token context?
> Cost/latency scale with tokens, accuracy degrades with distractors/position, KV memory limits concurrency, and updating/permissions/citations are easier with retrieval; use long context when it measurably helps.

> [!question]- Q60. How would you explain transformers to a non-technical executive in 60 seconds?
> "It predicts the next word using everything written so far, having learned patterns from a huge amount of text; extra training taught it to follow instructions and be helpful. It's a powerful pattern-completer — great drafting, summarizing, reasoning — but it can state wrong things confidently, so we pair it with your data, tools, checks, and human review."

Next → [[10-Resources-and-Practice-Links]]
