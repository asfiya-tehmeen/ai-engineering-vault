---
tags: [phase-2, pytorch]
---
# 06 — PyTorch

> PyTorch is the dominant framework for research and LLM work. It's on a ~2-month release cadence (2.14 as of Sep 2026 per the release blog — always check https://pytorch.org/blog/). Install per https://pytorch.org/get-started/locally/. JAX is the main alternative (esp. at Google/TPU); concepts transfer.

## 1. Tensors
```python
import torch
x = torch.tensor([[1., 2.], [3., 4.]])      # shape (2,2)
torch.zeros(3,4); torch.randn(2,3); torch.arange(10); torch.eye(3)
x.shape, x.dtype, x.device
x.to("cuda") / x.cuda() / x.to("mps")       # GPU / Apple Silicon
x.to(torch.bfloat16); x.float()
x @ x.T; x * x; x.sum(dim=0); x.mean(-1, keepdim=True)
x.view(4); x.reshape(-1); x.unsqueeze(0); x.squeeze(); x.permute(1,0); x.transpose(0,1)
torch.cat([a,b], dim=0); torch.stack([a,b]); x[:, 0]; x[x>2]
torch.einsum("bij,bjk->bik", A, B)
x.numpy(); torch.from_numpy(arr)             # share memory on CPU
```
- **Broadcasting** same as NumPy. **In-place ops** end with `_` (`x.add_(1)`) — can break autograd.
- **dtypes**: float32 (default), float16, **bfloat16** (same range as fp32, less precision — LLM standard), int64 (labels/indices), bool.
- **Device discipline:** all tensors in an op must be on the same device.

## 2. Autograd
```python
w = torch.tensor([2.0], requires_grad=True)
loss = ((w * 3 - 1) ** 2).sum()
loss.backward()           # populates w.grad
w.grad                    # accumulates across backward() calls → zero it each step
with torch.no_grad(): ... # no graph (inference, param updates)
x.detach()                # cut from graph
```
Dynamic computation graph (define-by-run). `loss.backward()` requires a scalar (or pass `gradient=`). `torch.autograd.grad` for explicit grads; `retain_graph`, `create_graph` (higher-order).

## 3. nn.Module
```python
import torch.nn as nn, torch.nn.functional as F

class MLP(nn.Module):
    def __init__(self, d_in, d_h, d_out, p=0.1):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(d_in, d_h), nn.GELU(), nn.Dropout(p),
            nn.Linear(d_h, d_h),  nn.GELU(), nn.Dropout(p),
            nn.Linear(d_h, d_out))
    def forward(self, x): return self.net(x)

model = MLP(784, 256, 10).to(device)
sum(p.numel() for p in model.parameters())      # parameter count
```
Key modules: `Linear, Conv2d, Embedding, LayerNorm, BatchNorm, Dropout, LSTM, MultiheadAttention, TransformerEncoderLayer`, containers (`Sequential, ModuleList, ModuleDict`). **Register submodules as attributes** (or ModuleList) so params are tracked. `nn.Parameter`, `register_buffer` (non-trainable state like running stats/masks). `model.train()` vs `model.eval()`.

## 4. Data: Dataset & DataLoader
```python
from torch.utils.data import Dataset, DataLoader
class MyDS(Dataset):
    def __init__(self, X, y): self.X, self.y = X, y
    def __len__(self): return len(self.X)
    def __getitem__(self, i): return self.X[i], self.y[i]
dl = DataLoader(MyDS(X, y), batch_size=64, shuffle=True, num_workers=4, pin_memory=True, drop_last=True)
```
`collate_fn` (variable-length padding), samplers, `IterableDataset` for streams, Hugging Face `datasets` for large corpora. Data loading often bottlenecks GPUs — profile.

## 5. The canonical training loop
```python
opt = torch.optim.AdamW(model.parameters(), lr=3e-4, weight_decay=0.01)
sched = torch.optim.lr_scheduler.OneCycleLR(opt, max_lr=3e-4, total_steps=epochs*len(train_dl))
scaler_dtype = torch.bfloat16

for epoch in range(epochs):
    model.train()
    for xb, yb in train_dl:
        xb, yb = xb.to(device, non_blocking=True), yb.to(device, non_blocking=True)
        with torch.autocast(device_type="cuda", dtype=scaler_dtype):      # mixed precision
            logits = model(xb)
            loss = F.cross_entropy(logits, yb)
        opt.zero_grad(set_to_none=True)
        loss.backward()
        torch.nn.utils.clip_grad_norm_(model.parameters(), 1.0)
        opt.step(); sched.step()

    model.eval()
    correct = total = 0
    with torch.no_grad():
        for xb, yb in val_dl:
            pred = model(xb.to(device)).argmax(-1).cpu()
            correct += (pred == yb).sum().item(); total += len(yb)
    print(epoch, loss.item(), correct/total)
    torch.save({"model": model.state_dict(), "opt": opt.state_dict(), "epoch": epoch}, "ckpt.pt")
```
**Order matters:** `zero_grad → forward → loss → backward → (clip) → step`. Use `loss.item()` sparingly in tight loops (GPU sync). Gradient accumulation: divide loss by `accum_steps`, step every N batches.

## 6. Checkpointing & reproducibility
- Save `state_dict()` (not whole model); also optimizer, scheduler, step, RNG state for exact resume. **safetensors** format (safe, fast; Hugging Face standard) — avoid untrusted `pickle`-based `.pt` files (code execution risk).
- Seeds: `torch.manual_seed`, `numpy`, `random`; `torch.use_deterministic_algorithms(True)` (slower); data-loader worker seeds. Perfect reproducibility across hardware is not guaranteed.

## 7. Performance
- **GPU basics:** CUDA cores/SMs, VRAM, memory-bandwidth vs compute-bound, tensor cores (need fp16/bf16 + aligned shapes). `nvidia-smi`, `torch.cuda.memory_summary()`.
- **Mixed precision** (autocast + bf16), **`torch.compile(model)`** (graph capture + kernel fusion via Inductor/Triton), **`torch.backends.cuda.matmul.allow_tf32`**, fused optimizers (`fused=True`), **FlashAttention** via `F.scaled_dot_product_attention`, channels-last for CNNs, `pin_memory`, larger batch until VRAM-bound, **gradient checkpointing**, profiling with `torch.profiler`.
- **Distributed:** `DistributedDataParallel (DDP)` (replicate model, shard data), **FSDP / FSDP2** (shard params/grads/optimizer), DeepSpeed ZeRO, tensor/pipeline parallelism (Phase 9). `torchrun`, NCCL.
- **Memory budget (training, mixed precision + Adam):** ≈ 16–18 bytes/param (weights + grads + optimizer states) + activations. A 7B model ≈ >100 GB to fully fine-tune naïvely → why LoRA/QLoRA exist (Phase 7).

## 8. Common errors & fixes
| Error | Cause |
|---|---|
| `RuntimeError: mat1 and mat2 shapes cannot be multiplied` | shape mismatch — print shapes |
| `Expected all tensors on same device` | forgot `.to(device)` |
| `CUDA out of memory` | smaller batch, bf16, grad checkpointing, accumulate, `del`/`empty_cache` rarely helps real issue |
| `NaN loss` | LR too high, bad data (inf/NaN), log(0), fp16 overflow; clip grads; `torch.autograd.set_detect_anomaly(True)` |
| `element 0 of tensors does not require grad` | calling backward on detached/no_grad tensor |
| Loss not decreasing | LR wrong, labels wrong, forgot `zero_grad`, data not normalized, bug in model |
| Eval worse than train by a lot | forgot `eval()`, leakage differences, overfit |

## 9. Ecosystem
- **torchvision / torchaudio / torchtext(legacy)**, **Lightning** / **Accelerate** (training boilerplate), **timm** (vision models), **Hugging Face Transformers/Datasets/PEFT/TRL** (Phases 3, 7), **ONNX / TorchScript / ExecuTorch** (export), **TensorBoard / W&B**, **torchmetrics**, **einops** (readable tensor ops: `rearrange(x, 'b h w c -> b c h w')`).
- **JAX/Flax**, **TensorFlow/Keras 3 (multi-backend)** — know they exist.
- **Notebooks & compute:** Google Colab, Kaggle (free GPUs/TPUs), Lightning Studios, Modal, RunPod, Lambda, Vast.ai; local GPU / Apple Silicon (MPS).

## 10. Mini-reference: shape cheat sheet
`Linear: (B,*,in)→(B,*,out)` · `Conv2d: (B,C,H,W)` · `Embedding: (B,T)→(B,T,D)` · `Attention: Q,K,V (B,h,T,d)` · `CrossEntropy: logits (B,C), target (B,) int64` (or `(B,C,T)` with target `(B,T)`).

Next → [[07-Deep-Architectures]]
