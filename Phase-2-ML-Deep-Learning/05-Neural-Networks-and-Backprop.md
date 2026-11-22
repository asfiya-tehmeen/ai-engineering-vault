---
tags: [phase-2, deep-learning, backprop]
---
# 05 — Neural Networks & Backpropagation

## 1. From neuron to network
- **Artificial neuron:** `a = φ(wᵀx + b)` — weighted sum + nonlinearity `φ` (activation).
- **Layer:** many neurons: `h = φ(Wx + b)`, `W ∈ ℝ^{out×in}`. **MLP (feed-forward net):** stack layers: `x → h₁ → h₂ → … → ŷ`.
- **Why nonlinearity?** Stacked linear maps collapse to one linear map. Nonlinearity gives expressiveness.
- **Universal approximation theorem:** a wide-enough 1-hidden-layer net can approximate any continuous function on a compact set. (Says *can*, not *will learn* or *efficiently*. Depth gives **hierarchical feature reuse** — exponentially more efficient for compositional functions.)
- **Representation learning:** hidden layers learn features; the final layer is a (linear/softmax) classifier on those features. **Embeddings** are hidden representations.

## 2. Activation functions
| Function | Formula | Notes |
|---|---|---|
| Sigmoid | `1/(1+e⁻ˣ)` | output (0,1); saturates → vanishing grads; used for gates/binary outputs |
| Tanh | `(eˣ−e⁻ˣ)/(eˣ+e⁻ˣ)` | zero-centered; still saturates |
| **ReLU** | `max(0,x)` | cheap, sparse, no saturation for x>0; "dying ReLU" risk |
| Leaky ReLU / PReLU | `max(αx,x)` | fixes dying |
| **GELU** | `x·Φ(x)` | smooth; BERT/GPT-2 era |
| **SiLU/Swish** | `x·σ(x)` | smooth, used widely |
| **SwiGLU / GeGLU** | gated: `(xW₁ ⊙ SiLU(xW₂))W₃` | in LLaMA-style transformers' MLPs |
| Softmax | `eᶻⁱ/Σeᶻʲ` | output layer for categorical; subtract max for stability |
Output layer + loss pairing: regression → linear + MSE; binary → sigmoid + BCE (use `BCEWithLogits`); multiclass → logits + CrossEntropy (applies log-softmax internally).

## 3. Forward pass
For a 2-layer MLP, batch input `X (B,d)`:
```
Z1 = X @ W1 + b1      # (B,h)
A1 = relu(Z1)
Z2 = A1 @ W2 + b2     # (B,k)  logits
P  = softmax(Z2)
L  = -mean(log P[range(B), y])
```

## 4. Backpropagation (derive it once by hand)
Backprop = **chain rule applied efficiently** over a computation graph, reusing intermediate gradients (dynamic programming) — reverse-mode autodiff.

For the net above, with `dZ2 = (P − Y_onehot)/B`:
```
dW2 = A1ᵀ @ dZ2          # (h,k)
db2 = sum(dZ2, axis=0)
dA1 = dZ2 @ W2ᵀ          # (B,h)
dZ1 = dA1 * (Z1 > 0)     # ReLU backward
dW1 = Xᵀ @ dZ1
db1 = sum(dZ1, axis=0)
```
**Pattern:** `local gradient × upstream gradient`. Each op knows how to compute its local backward; the framework chains them.
- Cache forward activations (memory cost!) → **activation checkpointing** trades compute for memory (important for big models).
- **Verify with numerical gradient checking** (`(f(θ+ε)−f(θ−ε))/2ε`).
- **Autograd engines** (micrograd, PyTorch): record ops in a DAG during forward, traverse in reverse topological order.

### Tiny autograd (micrograd idea) — you should build this (Exercise)
```python
class Value:
    def __init__(self, data, _children=(), _op=''):
        self.data, self.grad = data, 0.0
        self._backward = lambda: None
        self._prev = set(_children)
    def __add__(self, o):
        o = o if isinstance(o, Value) else Value(o)
        out = Value(self.data + o.data, (self, o), '+')
        def _backward():
            self.grad += out.grad; o.grad += out.grad
        out._backward = _backward; return out
    def __mul__(self, o):
        o = o if isinstance(o, Value) else Value(o)
        out = Value(self.data * o.data, (self, o), '*')
        def _backward():
            self.grad += o.data * out.grad; o.grad += self.data * out.grad
        out._backward = _backward; return out
    def backward(self):
        topo, seen = [], set()
        def build(v):
            if v not in seen:
                seen.add(v); [build(c) for c in v._prev]; topo.append(v)
        build(self); self.grad = 1.0
        for v in reversed(topo): v._backward()
```
(Note `+=`: gradients **accumulate** when a node is used multiple times.)

## 5. Training dynamics & problems
- **Vanishing gradients:** sigmoid/tanh derivatives <1 multiplied across many layers → ~0 signal in early layers. Fixes: ReLU-family, **residual connections**, normalization, good init, gated RNNs.
- **Exploding gradients:** repeated multiplication >1 → NaN. Fix: **gradient clipping** (`clip_grad_norm_`), lower LR, normalization, careful init.
- **Dead ReLUs**, **saturation**, **loss spikes** (LLM training instabilities; bf16, QK-norm, z-loss).
- **Initialization:** random small weights break symmetry; scale to preserve variance — **Xavier/Glorot** (tanh/linear: `Var=2/(fan_in+fan_out)`), **He/Kaiming** (ReLU: `Var=2/fan_in`). Zero biases. Zero-init the last layer of residual branches in some nets.
- **Normalization:** 
  - **BatchNorm** (normalize over batch per feature; running stats at inference; great for CNNs; issues with small batches/sequences).
  - **LayerNorm** (normalize over features per example; used in Transformers), **RMSNorm** (LLaMA-style, cheaper), GroupNorm, InstanceNorm. Pre-LN vs Post-LN.
- **Residual connections** `y = x + F(x)`: gradient highway → trains 100+ layer nets. (ResNet; Transformers.)
- **Dropout:** zero units with prob `p` during training (scale by `1/(1−p)`), disabled in eval (`model.eval()`!). Acts like ensembling.
- **Weight decay**, **label smoothing**, **data augmentation**, **early stopping**, **mixup/cutmix**.

## 6. Optimizers
| Optimizer | Update idea | Notes |
|---|---|---|
| SGD | `θ−ηg` | simple; noisy |
| SGD+Momentum | velocity `v=βv+g` | smooths, speeds along ravines; strong for CNNs |
| RMSProp | divide by running RMS of grads | per-parameter LR |
| **Adam** | momentum (m) + RMS (v) with bias correction: `θ−η·m̂/(√v̂+ε)` | default for most; β₁=0.9, β₂=0.999 (0.95 for LLMs) |
| **AdamW** | Adam + *decoupled* weight decay | standard for transformers/LLMs |
| Adafactor, Lion, Sophia, **Muon** | memory-lean / newer | active research |
| L-BFGS | quasi-Newton | small problems |
- **Learning-rate schedules:** step decay, cosine decay, **linear warmup + cosine/linear decay** (warmup stabilizes Adam early on; standard for transformers), one-cycle, constant with warmup (WSD – warmup-stable-decay in newer LLM training). LR range test to find good LR.
- **Batch size ↔ LR:** larger batch → larger LR (linear/sqrt scaling heuristic); **gradient accumulation** simulates big batches on small memory.
- **Mixed precision** (fp16/bf16 forward, fp32 master weights; loss scaling for fp16) — big speedups.
- Second-order intuition: curvature, condition number; why Adam helps.

## 7. Loss landscape intuition
Non-convex with many saddle points; wide/flat minima generalize better; **overparameterization + SGD noise** → implicit regularization; mode connectivity; lottery ticket hypothesis; grokking (delayed generalization). Don't over-theorize — measure.

## 8. Debugging neural nets (Karpathy's recipe, condensed)
1. Become one with the data (look at it!). 2. Build the simplest end-to-end pipeline + dumb baseline. 3. **Overfit a single batch** (loss→~0). 4. Verify initial loss ≈ `−log(1/num_classes)`. 5. Check gradients/shapes/train-vs-eval mode. 6. Add complexity one thing at a time. 7. Regularize; tune; ensemble. 8. Plot everything (loss, grad norms, weight/activation histograms, update-to-weight ratio ≈ 1e-3).
Common bugs: forgot `zero_grad`, wrong dim in softmax/loss, forgot `model.eval()` / `torch.no_grad()`, labels shape/dtype wrong, data not shuffled, normalization mismatch between train/inference, LR too high, double softmax before CrossEntropy.

## 9. Key architectures preview
CNNs (grids/images), RNNs (sequences), **Transformers (everything now)** — all in [[07-Deep-Architectures]] and Phase 3.

## 10. Self-check
1. Why can't a deep net of only linear layers beat linear regression? 2. Derive `∂L/∂z = p − y`. 3. Why scale activations by `1/(1−p)` in dropout? 4. What does Adam do that SGD doesn't? 5. Why warmup? 6. Why LayerNorm in transformers vs BatchNorm? 7. Why does reverse-mode autodiff cost ~ one forward pass regardless of parameter count?

Next → [[06-PyTorch]]
