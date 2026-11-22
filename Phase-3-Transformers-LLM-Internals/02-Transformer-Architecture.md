---
tags: [phase-3, transformer, attention]
---
# 02 — The Transformer Architecture

> Origin: *Attention Is All You Need* (Vaswani et al., 2017). Today's LLMs are **decoder-only transformers** with a handful of refinements. Learn the original, then the modern deltas.

## 1. Big picture (decoder-only LLM)
```
token ids (B,T)
 → Embedding lookup            (B,T,D)
 → [ TransformerBlock ] × L    (B,T,D)    ← residual stream
        block: x = x + Attn(Norm(x));  x = x + MLP(Norm(x))
 → final Norm
 → Unembedding (LM head)       (B,T,V)  logits  (often tied with embedding weights)
 → softmax → next-token distribution; loss = cross-entropy vs tokens shifted by 1
```
- **Residual stream** view (Anthropic's circuits framing): each layer *reads from* and *writes to* a shared D-dimensional stream; attention moves information **between positions**, MLPs process **within** a position (and store much factual "knowledge").

## 2. Self-attention
Intuition: each token asks a **query** ("what am I looking for?"), every token offers a **key** ("what do I contain?") and a **value** ("what do I give if chosen?"). Similarity(query, key) → weights → weighted sum of values. A *soft, differentiable dictionary lookup*.

**Scaled dot-product attention:**
```
Q = X Wq,  K = X Wk,  V = X Wv                 # (B,T,D) → (B,T,d_k)
scores = Q Kᵀ / √d_k                           # (B,T,T)
scores = scores.masked_fill(causal_mask, -inf)  # token t sees only ≤ t
A = softmax(scores, dim=-1)                    # rows sum to 1
out = A V                                      # (B,T,d_v)
```
- **Why `/√d_k`:** dot products grow with dimension, pushing softmax into saturated (tiny-gradient) regions; scaling keeps variance ≈ 1.
- **Causal mask:** enables training on all positions in parallel while preserving autoregressive semantics.
- **Complexity:** time/memory **O(T²·d)** — the reason long contexts are expensive. FlashAttention removes the O(T²) *memory* by tiling in SRAM (still O(T²) compute).
- **Permutation equivariance:** attention alone has no notion of order → need positional information.

### Multi-head attention (MHA)
Split D into `h` heads of size `d_k=D/h`; each head learns different relations (syntax, coreference, positional, induction…); concat then project with `Wo`.
```python
class CausalSelfAttention(nn.Module):
    def __init__(s, D, h):
        super().__init__(); s.h = h; s.qkv = nn.Linear(D, 3*D, bias=False); s.o = nn.Linear(D, D, bias=False)
    def forward(s, x):
        B, T, D = x.shape
        q, k, v = s.qkv(x).chunk(3, dim=-1)
        q, k, v = (t.view(B, T, s.h, D//s.h).transpose(1, 2) for t in (q, k, v))   # (B,h,T,dk)
        y = F.scaled_dot_product_attention(q, k, v, is_causal=True)                 # fused/Flash kernel
        return s.o(y.transpose(1, 2).reshape(B, T, D))
```
- **Induction heads** (copy-pattern completion `[A][B] … [A] → [B]`) underpin in-context learning (Phase 9 interpretability).

## 3. KV-sharing variants (inference efficiency)
During generation, K and V for past tokens are cached ([[05-Inference-and-Decoding]]). Fewer KV heads ⇒ smaller cache, faster decoding.
| Variant | KV heads | Notes |
|---|---|---|
| **MHA** | = query heads | original |
| **MQA** (multi-query) | 1 | smallest cache; some quality loss |
| **GQA** (grouped-query) | g groups (e.g., 8) | **dominant default** (LLaMA 2/3, Mistral, Qwen, Gemma…) |
| **MLA** (multi-head latent attention, DeepSeek) | low-rank *latent* compressed KV, up-projected on use | much smaller cache with MHA-like quality; used by DeepSeek V2/V3 lineage and models derived from it |
| Sparse/sliding/local attention | restrict which tokens attend | see below |

## 4. Positional information
- **Sinusoidal** (original), **learned absolute** (GPT-2/BERT).
- **RoPE (rotary)** — *modern default*: rotate query/key vectors in 2D planes by angle ∝ position, so the dot product depends on **relative** distance. `θᵢ = base^(−2i/d)`. Extends context via scaling tricks: **position interpolation, NTK-aware scaling, YaRN**, adjusting `base`.
- **ALiBi**: add linear distance penalty to attention scores; good extrapolation.
- **NoPE** (no explicit position; causal mask leaks order) works in some settings; some recent models interleave layers with/without RoPE.
- Why length generalization is hard: the model hasn't *trained* on far positions.

## 5. Normalization & MLP
- **Pre-LN** (norm *before* sublayer) trains more stably than original post-LN. **RMSNorm** (no mean-centering; cheaper) is standard. Some models add **QK-Norm** (normalize q,k) to stabilize large-scale training.
- **MLP / FFN:** `D → 4D → D` (≈ 2/3·4D with gated variants). **SwiGLU**: `(SiLU(xW₁) ⊙ xW₃)W₂`. ~⅔ of block parameters live in MLPs; they act like key–value memories.
- No biases in most modern LLMs (some exceptions, e.g., attention bias in certain models).
- **Residual scaling/init** for deep nets.

## 6. Parameter counting
Per layer (MHA, no GQA): attention `4D²`, MLP (SwiGLU, hidden `H`) `3DH`. Total ≈ `L·(4D² + 3DH) + V·D` (+V·D if untied). Rough rule: **params ≈ 12·L·D²** for standard 4D MLP. E.g., D=4096, L=32 → ~6.4B (+embeddings) ≈ "7B class".

## 7. Mixture-of-Experts (MoE) — now the default for large open models
- Replace the dense MLP with **N expert MLPs** + a **router**; each token activates **top-k experts** (e.g., 2 of 8, or 8 of 256 + shared expert(s)).
- **Total params ≫ active params**: capacity of a huge model, per-token compute of a small one. Naming like "30B-A3B" = 30B total, ~3B active.
- Components: router (linear + softmax/sigmoid + top-k), **load-balancing** (auxiliary loss or aux-loss-free bias updates), **shared experts**, **fine-grained experts**, capacity/dropping, expert parallelism across GPUs.
- Trade-offs: **memory** must hold *all* experts; inference batching/routing complexity; training instability; communication (all-to-all).
- Examples (verify details in live sources): Mixtral (8×7B), DeepSeek-V3 lineage, Qwen3-MoE, gpt-oss, Llama 4 (alternating dense/MoE), Kimi K2.

## 8. Long-context architecture tricks
- **Sliding-window (local) attention** layers mixed with occasional **global** layers (e.g., Gemma-style 5 local : 1 global; gpt-oss alternates) → cache & compute scale with window, not full T.
- **Attention sinks:** initial tokens soak up attention mass; streaming-LLM keeps them in cache; some models learn explicit sink parameters.
- **Sparse attention** (e.g., DeepSeek Sparse Attention), **chunked/segment** attention, **ring attention** for distributed long sequences.
- **Hybrid linear-attention / state-space** layers (Mamba/SSM, DeltaNet, RWKV-style) interleaved with attention — fixed-size state, linear cost. Growing in 2025–26 releases; watch Raschka's gallery for who uses what.

## 9. Other architecture families
- **Encoder-only (BERT):** bidirectional; for embeddings/classification/retrieval (embedding models are often encoder- or decoder-derived).
- **Encoder–decoder (T5, Whisper):** seq2seq, cross-attention.
- **Vision transformers (ViT)** and **multimodal** (image patches → tokens → LLM).
- **Diffusion transformers (DiT)** for images/video.
- **Diffusion language models** (parallel denoising of text) — research (Phase 9).
- **Multi-token prediction (MTP):** train to predict several next tokens; enables speculative-style speedups.

## 10. Why does it work? (mental models)
- Attention = content-addressable memory over context; MLP = learned key–value store; depth = iterative refinement; scale = more circuits/features (superposition: more features than dimensions).
- Next-token prediction on diverse data forces learning syntax, facts, reasoning patterns, world models *as far as they help prediction*.
- Parallelizable training (unlike RNNs) → scaled on GPUs → the scaling-law era.

## 11. Cheat-sheet: shapes
`x:(B,T,D)` · `q,k,v:(B,h,T,d)` · `scores:(B,h,T,T)` · `MLP: D→H→D` · `logits:(B,T,V)` · `loss = CE(logits[:, :-1], tokens[:, 1:])`.

## Self-check
1. Why mask? 2. Why √d_k? 3. What does each head see? 4. MHA→GQA: what shrinks? 5. RoPE vs learned absolute? 6. Dense 7B vs "30B-A3B" MoE: compare memory, FLOPs/token. 7. Where do most parameters live?

Next → [[03-Pretraining-and-Scaling]]
