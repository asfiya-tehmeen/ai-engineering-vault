---
tags: [phase-3, project, pytorch, gpt]
---
# 07 — Build a GPT From Scratch (walkthrough)

> Goal: ~150 lines of PyTorch that train a decoder-only transformer on text and sample from it. Type it, don't paste it. Pair with Karpathy's "Let's build GPT" and nanoGPT. Then extend with the "modern" upgrades at the end.

## 0. Setup
```python
import math, time, torch, torch.nn as nn, torch.nn.functional as F
from dataclasses import dataclass
device = "cuda" if torch.cuda.is_available() else ("mps" if torch.backends.mps.is_available() else "cpu")
torch.manual_seed(1337)
```

## 1. Data (char-level first, BPE later)
```python
text = open("input.txt", encoding="utf-8").read()           # e.g., tiny-shakespeare
chars = sorted(set(text)); stoi = {c:i for i,c in enumerate(chars)}; itos = {i:c for c,i in stoi.items()}
encode = lambda s: [stoi[c] for c in s]; decode = lambda ids: "".join(itos[i] for i in ids)
data = torch.tensor(encode(text), dtype=torch.long)
n = int(0.9*len(data)); train_data, val_data = data[:n], data[n:]

def get_batch(split, B, T):
    d = train_data if split == "train" else val_data
    ix = torch.randint(len(d) - T - 1, (B,))
    x = torch.stack([d[i:i+T] for i in ix]); y = torch.stack([d[i+1:i+T+1] for i in ix])   # y = x shifted by 1
    return x.to(device), y.to(device)
```

## 2. Model
```python
@dataclass
class Config:
    vocab_size: int; block_size: int = 256; n_layer: int = 6; n_head: int = 6; n_embd: int = 384; dropout: float = 0.1

class Attention(nn.Module):
    def __init__(s, c):
        super().__init__(); assert c.n_embd % c.n_head == 0
        s.qkv = nn.Linear(c.n_embd, 3*c.n_embd, bias=False); s.proj = nn.Linear(c.n_embd, c.n_embd, bias=False)
        s.h, s.drop = c.n_head, c.dropout
    def forward(s, x):
        B, T, C = x.shape
        q, k, v = s.qkv(x).split(C, dim=2)
        q, k, v = (t.view(B, T, s.h, C//s.h).transpose(1, 2) for t in (q, k, v))       # (B,h,T,dk)
        y = F.scaled_dot_product_attention(q, k, v, is_causal=True, dropout_p=s.drop if s.training else 0.0)
        return s.proj(y.transpose(1, 2).contiguous().view(B, T, C))

class MLP(nn.Module):
    def __init__(s, c):
        super().__init__(); s.fc = nn.Linear(c.n_embd, 4*c.n_embd, bias=False); s.proj = nn.Linear(4*c.n_embd, c.n_embd, bias=False); s.drop = nn.Dropout(c.dropout)
    def forward(s, x): return s.drop(s.proj(F.gelu(s.fc(x))))

class Block(nn.Module):
    def __init__(s, c):
        super().__init__(); s.ln1 = nn.LayerNorm(c.n_embd); s.attn = Attention(c); s.ln2 = nn.LayerNorm(c.n_embd); s.mlp = MLP(c)
    def forward(s, x):
        x = x + s.attn(s.ln1(x))          # communication between positions
        x = x + s.mlp(s.ln2(x))           # computation within a position
        return x

class GPT(nn.Module):
    def __init__(s, c):
        super().__init__(); s.c = c
        s.tok = nn.Embedding(c.vocab_size, c.n_embd); s.pos = nn.Embedding(c.block_size, c.n_embd)
        s.drop = nn.Dropout(c.dropout); s.blocks = nn.ModuleList(Block(c) for _ in range(c.n_layer))
        s.ln_f = nn.LayerNorm(c.n_embd); s.head = nn.Linear(c.n_embd, c.vocab_size, bias=False)
        s.head.weight = s.tok.weight                            # weight tying
        s.apply(s._init)
    def _init(s, m):
        if isinstance(m, nn.Linear): nn.init.normal_(m.weight, 0, 0.02)
        elif isinstance(m, nn.Embedding): nn.init.normal_(m.weight, 0, 0.02)
    def forward(s, idx, targets=None):
        B, T = idx.shape
        x = s.drop(s.tok(idx) + s.pos(torch.arange(T, device=idx.device)))
        for b in s.blocks: x = b(x)
        logits = s.head(s.ln_f(x))                              # (B,T,V)
        loss = None if targets is None else F.cross_entropy(logits.view(-1, logits.size(-1)), targets.view(-1))
        return logits, loss
    @torch.no_grad()
    def generate(s, idx, max_new, temperature=1.0, top_k=None):
        for _ in range(max_new):
            logits, _ = s(idx[:, -s.c.block_size:])
            logits = logits[:, -1, :] / max(temperature, 1e-6)
            if top_k: v, _ = torch.topk(logits, top_k); logits[logits < v[:, [-1]]] = -float("inf")
            idx = torch.cat([idx, torch.multinomial(F.softmax(logits, -1), 1)], dim=1)
        return idx
```
**Sanity checks:** initial loss ≈ `ln(vocab)` (e.g., ln 65 ≈ 4.17). Overfit one batch to ~0. Count params; confirm shapes.

## 3. Training
```python
cfg = Config(vocab_size=len(chars)); model = GPT(cfg).to(device)
print(sum(p.numel() for p in model.parameters())/1e6, "M params")
opt = torch.optim.AdamW(model.parameters(), lr=3e-4, betas=(0.9, 0.95), weight_decay=0.1)
B, T, steps, warmup = 64, cfg.block_size, 5000, 200
def lr_at(it): 
    if it < warmup: return 3e-4 * (it+1) / warmup
    p = (it - warmup) / (steps - warmup); return 3e-5 + 0.5 * (3e-4 - 3e-5) * (1 + math.cos(math.pi * p))

@torch.no_grad()
def evaluate():
    model.eval(); out = {}
    for split in ("train", "val"):
        out[split] = sum(model(*get_batch(split, B, T))[1].item() for _ in range(20)) / 20
    model.train(); return out

for it in range(steps):
    for g in opt.param_groups: g["lr"] = lr_at(it)
    xb, yb = get_batch("train", B, T)
    with torch.autocast(device_type=device if device != "mps" else "cpu", dtype=torch.bfloat16, enabled=(device == "cuda")):
        _, loss = model(xb, yb)
    opt.zero_grad(set_to_none=True); loss.backward()
    torch.nn.utils.clip_grad_norm_(model.parameters(), 1.0); opt.step()
    if it % 250 == 0: print(it, evaluate())

print(decode(model.generate(torch.zeros((1,1), dtype=torch.long, device=device), 500, temperature=0.8, top_k=40)[0].tolist()))
```
Expect Shakespeare-ish gibberish with correct structure after minutes on a GPU; val loss ~1.5 for a 10M-param char model.

## 4. Upgrade path ("modernize" it — each is an exercise)
1. **BPE tokenizer** (tiktoken GPT-2 or your own) + a bigger corpus (FineWeb-Edu sample / OpenWebText shard) → follow Karpathy's `build-nanogpt`.
2. **RMSNorm** instead of LayerNorm. 3. **RoPE** instead of learned positions. 4. **SwiGLU MLP.** 5. **GQA** (`n_kv_head < n_head`). 6. **QK-norm.**
7. **KV-cache generation** (see Exercise).
8. **Mixed precision + `torch.compile` + fused AdamW + gradient accumulation** for speed.
9. **Data loader with sharded token files** (memmap `.bin`), packing with EOS.
10. **Checkpoint/resume + W&B logging.**
11. **MoE layer** (top-2 router + load-balance loss).
12. **Evaluate**: val loss/bits-per-byte; HellaSwag-style multiple-choice scoring via log-likelihood.
13. **Fine-tune** your GPT on instruction data (SFT with assistant-only loss mask), then DPO a tiny model with TRL.

## 5. KV-cache sketch (exercise starter)
```python
# In Attention.forward(x, cache=None): compute q,k,v for the new tokens; if cache: k = cat(cache.k, k), v = cat(cache.v, v)
# Return (y, (k, v)). In generate(): first pass processes the full prompt (prefill); then feed only the last token each step (decode).
# Verify: logits with and without cache match to ~1e-5, and measure tokens/sec improvement vs T.
```

## 6. Compare with a reference
Load HF GPT-2 weights into your architecture (nanoGPT's `from_pretrained`) and check logits match → proves your implementation is exactly GPT-2.

## 7. Related reading while you build
The Illustrated Transformer · The Annotated Transformer · Karpathy: nanoGPT, build-nanogpt, llm.c · Raschka: *Build a Large Language Model (From Scratch)* · bbycroft.net/llm (3D visualization) · Stanford CS336 (Language Modeling from Scratch).

Next → [[08-Exercises]]
