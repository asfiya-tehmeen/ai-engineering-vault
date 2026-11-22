---
tags: [phase-2, ml]
---
# 02 — Machine Learning Fundamentals

## 1. What is ML?
Traditional programming: `rules + data → answers`. ML: `data + answers → rules (a model)`. A model is a **parameterized function** `ŷ = f(x; θ)`; **learning** = finding `θ` that makes predictions good on *new* data.

### Paradigms
| Paradigm | Signal | Examples |
|---|---|---|
| **Supervised** | labeled `(x, y)` | classification, regression, translation |
| **Unsupervised** | no labels | clustering, density estimation, dimensionality reduction |
| **Self-supervised** | labels derived from data itself | next-token prediction (LLMs), masked LM, contrastive learning |
| **Semi-supervised** | few labels + many unlabeled | pseudo-labeling |
| **Reinforcement learning** | reward from environment | games, robotics, **RLHF/RLVR for LLMs** |
| **Weak/Distant supervision** | noisy programmatic labels | data programming |

### Task types
Regression (continuous), binary/multiclass/multilabel classification, ranking, sequence labeling, generation, clustering, anomaly detection, recommendation, forecasting.

## 2. The ML pipeline
`Problem framing → data collection → labeling → EDA → splitting → features → model → loss/optimization → evaluation → error analysis → iterate → deploy → monitor → retrain`.
> Most real-world gains come from **better data/framing/evaluation**, not fancier models. (True for LLM apps too — Phase 7.)

## 3. Core formalism
- Data: `{(xᵢ, yᵢ)}ᵢ₌₁ⁿ` drawn i.i.d. from unknown distribution `P(x,y)`.
- Model class `f(x;θ)`; **loss** `ℓ(ŷ, y)` per example; **empirical risk** `R̂(θ) = (1/n) Σ ℓ(f(xᵢ;θ), yᵢ)`.
- **Goal:** minimize **true risk** `E[ℓ]` on unseen data, but we only optimize empirical risk → **generalization gap**.
- **i.i.d. assumption** and **distribution shift** (covariate shift, label shift, concept drift) — why models rot in production.

## 4. Loss functions
| Task | Loss | Notes |
|---|---|---|
| Regression | MSE (`(ŷ−y)²`) | sensitive to outliers; Gaussian MLE |
| | MAE | robust; median regression |
| | Huber / smooth-L1 | MSE near 0, MAE far |
| Binary class | Binary cross-entropy `−[y log p + (1−y) log(1−p)]` | Bernoulli MLE |
| Multiclass | Categorical cross-entropy `−log p_y` | with softmax |
| Margin | Hinge `max(0, 1 − y·s)` | SVM |
| Ranking | pairwise/listwise, triplet, contrastive (InfoNCE) | embeddings |
| Imbalance | weighted CE, focal loss | |
**Loss ≠ metric.** Optimize a differentiable *surrogate* (cross-entropy); evaluate what you care about (F1, revenue, human preference).

## 5. Gradient descent (the engine)
`θ ← θ − η ∇θ R̂(θ)`
- **Batch GD** (all data), **SGD** (one example), **mini-batch SGD** (typical: 32–4096; LLMs use millions of tokens per batch).
- **Learning rate η**: too high → diverge/oscillate; too low → slow/stuck. Often the most important hyperparameter.
- **Epoch** = one pass over data; **iteration/step** = one update.
- **Momentum**: accumulate velocity `v ← βv + g; θ ← θ − ηv`. **Nesterov**. **RMSProp**, **Adam** (momentum + per-parameter adaptive scaling), **AdamW** (decoupled weight decay) — see [[05-Neural-Networks-and-Backprop]].
- **Feature scaling** (standardize) dramatically speeds up GD (conditioning).
- **Convergence diagnostics:** plot loss curves (train & val). Spiky → LR too high / bad batch; flat → LR too low / dead units.

### From scratch: linear regression
```python
import numpy as np
def fit_linreg(X, y, lr=0.1, steps=1000):
    n, d = X.shape
    Xb = np.c_[np.ones(n), X]            # add bias column
    w = np.zeros(d + 1)
    for _ in range(steps):
        grad = (2/n) * Xb.T @ (Xb @ w - y)
        w -= lr * grad
    return w
```
Closed form (normal equation) `w = (XᵀX)⁻¹Xᵀy` — fine for small `d`; GD scales.

## 6. Generalization: the central tension
- **Underfitting** (high bias): model too simple; both train & val error high. Fix: more capacity, better features, train longer.
- **Overfitting** (high variance): memorizes noise; train error low, val error high. Fix: more data, regularization, simpler model, augmentation, early stopping, ensembling.
- **Bias–variance decomposition (squared error):** `E[(y−f̂)²] = Bias² + Variance + Irreducible noise`.
- **Modern twist — double descent:** very over-parameterized models can generalize *better* past the interpolation threshold; why giant neural nets work. Classical U-curve isn't the whole story.
- **Capacity**: VC dimension (theory), parameter count (practice). **Scaling laws** (Phase 3): loss falls predictably with data/params/compute.

## 7. Regularization toolbox
| Technique | Idea |
|---|---|
| L2 / weight decay | penalize `‖w‖²`; shrinks weights; Gaussian prior |
| L1 (Lasso) | penalize `‖w‖₁`; sparse weights/feature selection |
| Elastic net | L1+L2 |
| Early stopping | stop when val loss rises |
| Dropout | randomly zero activations during training |
| Data augmentation | create label-preserving variants |
| Label smoothing | soft targets reduce overconfidence |
| Ensembling/bagging | average many models → lower variance |
| Noise injection, mixup, stochastic depth | |
| Smaller/simpler model, pruning | |
| **More (and better) data** | the best regularizer |

## 8. Data splitting & validation (get this right!)
- **Train / validation / test.** Train fits parameters; **validation** tunes hyperparameters/selects models; **test** is touched **once** at the end for an unbiased estimate. Typical 70/15/15 or 80/10/10; with big data, tiny val/test fractions suffice.
- **Stratified** splits preserve class ratios. **Group** splits (all rows from one user/patient/doc in the same fold). **Time-based** splits for temporal data (train on past, test on future — never shuffle!).
- **k-fold cross-validation**: rotate validation folds; average. Nested CV for unbiased tuning estimates on small data.
- **Data leakage** — the silent killer. Patterns:
  1. Preprocessing (scaler/imputer/encoder/TF-IDF) fit on all data before splitting → fit on train only (use `Pipeline`).
  2. Duplicates/near-duplicates across splits.
  3. Target leakage: a feature computed from/after the label (e.g., "refund_issued" predicting fraud).
  4. Temporal leakage: future information.
  5. Group leakage: same entity in train and test.
  6. Test-set tuning: repeatedly peeking at test → overfit to test (also *benchmark contamination* for LLMs).
- **Baselines first**: majority class, mean predictor, simple linear/logistic model, previous-period forecast. If your fancy model barely beats the baseline, investigate.

## 9. Feature engineering (classical ML)
- Numeric: scaling (standardize / min-max / robust), log transform for skew, binning, interactions, polynomial features.
- Categorical: one-hot, ordinal, target/mean encoding (careful: leakage), hashing, **learned embeddings**.
- Text: bag-of-words, **TF-IDF**, n-grams → later **embeddings/transformers**.
- Dates: hour/day-of-week/cyclical (sin/cos), time since event.
- Missing data: MCAR/MAR/MNAR; impute (mean/median/model), add "was_missing" flag.
- Outliers, class imbalance (resampling, class weights, SMOTE, threshold tuning).
- Dimensionality: curse of dimensionality (distances concentrate; need exponentially more data).
- Deep learning largely **learns features** from raw data (representation learning), reducing manual feature work.

## 10. Hyperparameters vs parameters
- **Parameters** `θ`: learned by optimization (weights). **Hyperparameters**: set by you (LR, depth, regularization strength, layers, batch size).
- Search: grid, **random search** (usually beats grid), Bayesian optimization (Optuna), successive halving/Hyperband, population-based training. Tune on **validation**, never test.

## 11. Generative vs discriminative
- **Discriminative** models `P(y|x)` (logistic regression, SVM, most NNs). **Generative** models `P(x)` or `P(x,y)` (naive Bayes, VAEs, GANs, diffusion, **LLMs**). An LLM is a generative model of `P(next token | previous tokens)`.

## 12. Parametric vs non-parametric
Parametric: fixed number of params (linear models, NNs). Non-parametric: complexity grows with data (kNN, kernel methods, decision trees, Gaussian processes).

## 13. Probabilistic view & uncertainty
- Model outputs should be **calibrated probabilities**. Aleatoric (data noise) vs **epistemic** (model ignorance) uncertainty. LLM "confidence" ≠ correctness (Phase 3/7).
- **Out-of-distribution (OOD)** detection, abstention, conformal prediction.

## 14. No Free Lunch & inductive bias
No algorithm is best on all problems. **Inductive bias** = built-in assumptions (CNN: locality/translation equivariance; RNN: sequence order; Transformer: pairwise attention; trees: axis-aligned splits). Choosing a model = choosing assumptions that match your data.

## 15. Common beginner mistakes
Not scaling features for GD/kNN/SVM · leakage · tuning on test · accuracy on imbalanced data · no baseline · ignoring seeds/variance · shuffling time series · evaluating only aggregate metric (check slices) · training on label noise without noticing.

Next → [[03-Classical-ML-Algorithms]]
