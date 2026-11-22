---
tags: [phase-2, math]
---
# 01 — Math Foundations for ML

> Strategy: learn each idea **geometrically first**, then as formula, then in NumPy. Watch 3Blue1Brown for intuition (links in [[11-Resources-and-Practice-Links]]).

## Part A — Linear algebra
### Objects
- **Scalar** (number), **vector** `x ∈ ℝⁿ` (list of numbers = a point/arrow), **matrix** `A ∈ ℝ^{m×n}`, **tensor** (n-dim array). Shapes are everything: *most ML bugs are shape bugs.*
- A vector can be a data point, a set of weights, or an **embedding** (a meaning encoded as coordinates).

### Operations
| Op | Formula | ML meaning |
|---|---|---|
| Dot product | `a·b = Σ aᵢbᵢ = ‖a‖‖b‖cosθ` | similarity; a neuron's weighted sum |
| Norms | `‖x‖₂ = √Σxᵢ²`, `‖x‖₁ = Σ|xᵢ|` | length; L2/L1 regularization |
| Cosine similarity | `a·b / (‖a‖‖b‖)` | semantic search (Phase 5) |
| Matrix–vector | `y = Wx` | a linear layer |
| Matrix–matrix | `C = AB`, `Cᵢⱼ = Σₖ AᵢₖBₖⱼ`, shapes `(m,k)(k,n)→(m,n)` | batched layers, attention `QKᵀ` |
| Transpose | `(AB)ᵀ = BᵀAᵀ` | |
| Hadamard | elementwise `A ⊙ B` | gating |
| Outer product | `abᵀ` | rank-1 matrix (LoRA is low-rank!) |

### Key concepts
- **Linear transformation**: matrices rotate/scale/shear space. A neural layer = linear map + nonlinearity.
- **Basis, span, linear independence, rank**: rank = how many independent directions. **Low-rank** structure is why LoRA (Phase 7) and PCA work.
- **Inverse, determinant, solving `Ax=b`**; ill-conditioned matrices → numerical instability.
- **Eigenvalues/eigenvectors**: `Av = λv` — directions a matrix only stretches. PCA = eigenvectors of the covariance matrix.
- **SVD**: `A = UΣVᵀ` — universal decomposition; best low-rank approximation (Eckart–Young). Used for PCA, compression, latent semantic analysis, LoRA intuition.
- **Orthogonality, projections**: least squares = projecting `b` onto column space of `A`; normal equation `w = (XᵀX)⁻¹Xᵀy`.
- **Positive (semi)definite** matrices ↔ convex quadratic bowls; Hessians & curvature.
- **Norm balls & regularization geometry**: L1 ball has corners → sparsity; L2 ball is round → shrinkage.
- **Matrix calculus** (see below) — gradient of `‖Xw − y‖²` is `2Xᵀ(Xw − y)`.

### In NumPy
```python
import numpy as np
A = np.random.randn(5, 3); x = np.random.randn(3)
A @ x; A.T @ A
U, S, Vt = np.linalg.svd(A, full_matrices=False)
np.allclose(A, U @ np.diag(S) @ Vt)
w, v = np.linalg.eigh(A.T @ A)         # symmetric → eigh
```
**Shape discipline:** write shapes in comments: `# (B, T, D) @ (D, H) -> (B, T, H)`. Learn `einsum`: `np.einsum('bij,bjk->bik', A, B)`.

## Part B — Calculus
- **Derivative** `f'(x)`: slope/sensitivity. Rules: sum, product, quotient, **chain rule**: `d/dx f(g(x)) = f'(g(x)) g'(x)` ← **the** rule behind backprop.
- Common: `d/dx xⁿ = nxⁿ⁻¹`, `eˣ`, `ln x → 1/x`, `sigmoid σ' = σ(1−σ)`, `tanh' = 1−tanh²`, `ReLU' = 1[x>0]`.
- **Partial derivatives & gradient**: `∇f = [∂f/∂x₁, …, ∂f/∂xₙ]` points in the direction of steepest ascent. **Gradient descent moves opposite to it.**
- **Jacobian** (vector→vector derivative matrix), **Hessian** (second derivatives; curvature).
- **Chain rule for vectors**: `∂L/∂x = Jᵀ ∂L/∂y` — "vector–Jacobian product" (what autograd computes).
- **Taylor approximation**: `f(x+δ) ≈ f(x) + ∇fᵀδ + ½δᵀHδ` — why learning rates must be small and why second-order methods exist.
- **Convexity**: convex → every local min is global (linear/logistic regression). Neural nets are **non-convex**, yet SGD works well in practice (saddle points, flat minima, overparameterization).
- **Numerical gradient check**: `(f(x+ε) − f(x−ε)) / 2ε` to verify your analytic gradient. Do this.
- **Automatic differentiation**: forward mode vs **reverse mode** (backprop). Reverse mode is efficient when outputs ≪ inputs (one scalar loss, millions of params).

### Worked example: gradient of MSE for linear regression
`L(w) = (1/n)‖Xw − y‖²` → `∇_w L = (2/n) Xᵀ(Xw − y)`. Update: `w ← w − η∇L`.

### Worked example: softmax + cross-entropy gradient (memorize this)
`p = softmax(z)`, `L = −log p_y` → `∂L/∂z = p − onehot(y)`. Beautiful and numerically stable; used in every classifier and every LLM.

## Part C — Probability
- **Sample space, events, axioms.** `P(A∪B)=P(A)+P(B)−P(A∩B)`.
- **Conditional probability** `P(A|B)=P(A∩B)/P(B)`; **independence**; **chain rule** `P(x₁..xₙ)=Π P(xᵢ|x₍<ᵢ₎)` ← *autoregressive language modeling is exactly this factorization.*
- **Bayes' theorem**: `P(θ|D) = P(D|θ)P(θ)/P(D)` — posterior ∝ likelihood × prior. Base-rate fallacy (rare disease + 99% test).
- **Random variables, PMF/PDF/CDF, expectation `E[X]`, variance `Var(X)=E[X²]−E[X]²`, covariance, correlation**. Linearity of expectation.
- **Distributions you must know:** Bernoulli, Binomial, Categorical (→ softmax outputs), Multinomial, Poisson, Uniform, **Gaussian** `N(μ,σ²)` (why: CLT, max entropy given mean/var), Exponential, Beta, Dirichlet, Student-t, heavy tails/Zipf (word frequencies!).
- **Multivariate Gaussian**, covariance matrix, Mahalanobis distance.
- **Law of large numbers, Central Limit Theorem** — why averages stabilize and why we can compute confidence intervals; why minibatch gradients are unbiased noisy estimates.
- **Maximum Likelihood Estimation (MLE)**: choose params maximizing `P(data|θ)`. Equivalently minimize negative log-likelihood (NLL).
  - Gaussian noise → MSE loss. Bernoulli → binary cross-entropy. Categorical → cross-entropy.
  - *Pretraining an LLM = MLE on text.*
- **MAP estimation**: MLE + prior. Gaussian prior ⇒ L2 regularization; Laplace prior ⇒ L1.
- **Bayesian vs frequentist** viewpoints (know the difference at a high level).
- **Sampling:** inverse CDF, rejection sampling, MCMC (Metropolis–Hastings), importance sampling; **sampling from a categorical with temperature** (Phase 3 decoding).
- **Markov chains**, hidden Markov models (historical context).

## Part D — Statistics
- Descriptive stats, histograms, quantiles (p95/p99), robust stats (median/MAD).
- **Estimators**: bias, variance, consistency. Sample variance uses `n−1` (Bessel).
- **Hypothesis testing**: null, p-value (probability of data this extreme *given null* — not P(null true)), Type I/II errors, power, multiple-comparison problem (Bonferroni, FDR).
- **Confidence intervals** & **bootstrap** (resample to estimate uncertainty of any metric — crucial for eval metrics on small test sets).
- **A/B testing** basics: randomization, sample size, novelty effects, guardrail metrics.
- Correlation vs causation; confounders; Simpson's paradox; selection bias; survivorship bias.
- **Paired tests / paired bootstrap** for comparing two models on the same examples.

## Part E — Information theory (shows up constantly in LLMs)
- **Surprise/information**: `I(x) = −log p(x)`.
- **Entropy**: `H(p) = −Σ p log p` (expected surprise; uncertainty).
- **Cross-entropy**: `H(p,q) = −Σ p(x) log q(x)` — expected bits using code for `q` when truth is `p`. **Training loss for classification and next-token prediction.**
- **KL divergence**: `KL(p‖q) = H(p,q) − H(p) ≥ 0`, asymmetric, not a distance. Appears in VAEs, RLHF/PPO penalties, distillation, DPO derivation.
- **Mutual information**, **Jensen–Shannon**, **perplexity** `= exp(cross-entropy)` — "effective branching factor"; the classic LM metric. Bits-per-byte.
- **Minimizing cross-entropy ⇔ minimizing KL ⇔ MLE.** Memorize this equivalence.

## Part F — Optimization theory (light)
- Objective, constraints, convex vs non-convex, local/global minima, saddle points.
- **Gradient descent**, learning rate, convergence on quadratics (condition number → slow zig-zag; why momentum/Adam help).
- **Lagrange multipliers / duality** (appears in SVMs; skim).
- **Stochastic** optimization: noise helps escape sharp minima; batch size tradeoffs.

## Self-check (answer aloud)
1. Shape of `XᵀX` if `X` is `(n,d)`? 2. Gradient of `½‖w‖²`? 3. Why does `p − y` appear for softmax-CE? 4. Why is `log` used in likelihood? 5. Difference between `KL(p‖q)` and `KL(q‖p)`? 6. What does the chain rule have to do with backprop? 7. Why does L2 regularization equal a Gaussian prior?

Next → [[02-ML-Fundamentals]]
