---
tags: [phase-2, exercises]
---
# 09 — Exercises (Phase 2)

🟢 easy · 🟡 medium · 🔴 hard. Rule: **implement from scratch first, then compare with the library.** Keep everything in `phase2-exercises/` with tests.

## A. Math by hand + NumPy (Note 01)
- [ ] 🟢 A1. Compute dot product, norm, cosine similarity of 3 vector pairs by hand; verify in NumPy.
- [ ] 🟢 A2. Multiply a (2×3)(3×2) matrix by hand; write the shape of every intermediate in an `A @ B @ C` chain.
- [ ] 🟢 A3. Differentiate by hand: `x³+2x`, `σ(x)`, `tanh(x)`, `ln(1+eˣ)`, `(wx−y)²` wrt `w`. Verify with finite differences.
- [ ] 🟡 A4. Derive `∂/∂z` of softmax-cross-entropy = `p − y`. Verify numerically.
- [ ] 🟡 A5. Implement SVD-based PCA on a dataset; compare with `sklearn.decomposition.PCA` (signs may flip).
- [ ] 🟡 A6. Low-rank approximation of an image with SVD at ranks 1,5,20,50; plot error vs rank.
- [ ] 🟡 A7. Simulate the CLT: average of n uniform variables; histogram for n=1,2,10,50.
- [ ] 🟡 A8. Bayes: P(disease)=0.1%, sensitivity 99%, specificity 95%. Compute P(disease|positive) analytically and by simulation.
- [ ] 🟡 A9. Compute entropy of a fair coin, biased coin, and the character distribution of a text file. Compute KL between two distributions; show asymmetry.
- [ ] 🔴 A10. Show numerically that minimizing cross-entropy equals maximizing log-likelihood; show Gaussian noise MLE equals MSE.
- [ ] 🔴 A11. Implement `numerically_stable_logsumexp` and softmax; demonstrate overflow in the naive version.
- [ ] 🔴 A12. Prove & verify: for a matrix `A`, the Frobenius norm² = sum of squared singular values.

## B. Classical ML from scratch (Notes 02–03)
- [ ] 🟢 B1. Linear regression by normal equation and by gradient descent; plot loss curve; compare to sklearn.
- [ ] 🟢 B2. Show effect of learning rate (0.001, 0.1, 1.5) on convergence; show effect of feature scaling.
- [ ] 🟡 B3. Logistic regression from scratch (sigmoid + BCE + GD), L2 option; decision boundary plot on 2D data.
- [ ] 🟡 B4. Polynomial regression degree 1–15 on noisy sine data; plot train vs val error (bias–variance U-curve); add Ridge and show it tames overfit.
- [ ] 🟡 B5. Implement k-NN with vectorized distance computation; evaluate vs k; show curse of dimensionality (distance concentration) experiment.
- [ ] 🟡 B6. k-means from scratch (k-means++ init); compare with sklearn; elbow + silhouette.
- [ ] 🟡 B7. Implement a decision tree (Gini, recursive splits, max_depth) for classification.
- [ ] 🔴 B8. Random forest from your tree via bootstrap + feature subsampling; show variance reduction vs a single tree.
- [ ] 🔴 B9. Gradient boosting for regression from scratch using shallow sklearn trees on residuals (`F += η·tree(residual)`).
- [ ] 🔴 B10. Naive Bayes (Multinomial) text classifier from scratch with Laplace smoothing; test on 20 Newsgroups subset.
- [ ] 🔴 B11. PCA vs UMAP/t-SNE on MNIST embeddings; describe what each preserves & misleads.

## C. Pipelines & evaluation (Notes 03–04)
- [ ] 🟢 C1. Build a sklearn `Pipeline` + `ColumnTransformer` on Titanic; 5-fold CV; confidence interval via bootstrap.
- [ ] 🟡 C2. Create **deliberate leakage** (scale before splitting; target-leaking feature) and quantify the inflated score vs correct pipeline.
- [ ] 🟡 C3. Imbalanced dataset (1% positives): compare accuracy, F1, PR-AUC, ROC-AUC; tune threshold for recall ≥ 0.9.
- [ ] 🟡 C4. Plot a reliability diagram; calibrate with Platt and isotonic; compute ECE before/after.
- [ ] 🟡 C5. Time-series split vs random split on a forecasting task; show how random split overestimates performance.
- [ ] 🟡 C6. Compare two models using a paired bootstrap on the same test set; report CI of the metric difference.
- [ ] 🔴 C7. Run LightGBM/XGBoost with Optuna (≥50 trials, pruning); report val vs test gap; discuss winner's curse.
- [ ] 🔴 C8. SHAP analysis on your best model; find one surprising feature and investigate whether it's leakage.
- [ ] 🔴 C9. Implement nDCG@k, MRR, Recall@k from scratch; unit-test on hand-made rankings.
- [ ] 🔴 C10. Slice analysis: compute metrics per subgroup; find a slice with ≥10 point drop; propose a fix.

## D. Neural nets & backprop (Note 05)
- [ ] 🟢 D1. Forward pass of a 2-layer MLP in NumPy on a toy batch.
- [ ] 🟡 D2. Full backprop in NumPy (no autograd); numerical gradient check passes (rel. error <1e-6).
- [ ] 🟡 D3. Train your NumPy MLP on XOR / spirals / MNIST subset; plot decision boundaries & loss.
- [ ] 🟡 D4. Compare activations (sigmoid, tanh, ReLU) on a 10-layer net: plot gradient magnitudes per layer (vanishing demo).
- [ ] 🟡 D5. Compare init schemes (zeros, small random, Xavier, He): activation stats per layer.
- [ ] 🟡 D6. Implement SGD, Momentum, RMSProp, Adam; race them on a 2D "Rosenbrock/ravine" function; animate paths.
- [ ] 🔴 D7. **Build micrograd**: scalar autograd `Value` with +, *, **, tanh/relu, exp, log; train a small MLP; compare grads to PyTorch.
- [ ] 🔴 D8. Implement BatchNorm and LayerNorm forward/backward by hand; verify against `torch.nn`.
- [ ] 🔴 D9. Implement dropout (inverted) and show train/eval behavior difference.
- [ ] 🔴 D10. Implement a learning-rate warmup + cosine schedule from scratch; verify against PyTorch scheduler.

## E. PyTorch (Note 06)
- [ ] 🟢 E1. Tensor drill: reshape/permute/broadcast/einsum 20 tiny puzzles (e.g., batched pairwise distances without loops).
- [ ] 🟢 E2. Re-implement linear regression using `autograd` & manual SGD, then with `nn.Linear` + `optim`.
- [ ] 🟡 E3. MNIST/Fashion-MNIST MLP: ≥97%/≥88% test acc; plot curves; save/load checkpoint & resume.
- [ ] 🟡 E4. Custom `Dataset` + `collate_fn` for variable-length sequences with padding & masks.
- [ ] 🟡 E5. Add AMP (bf16), grad clipping, gradient accumulation; measure speed/memory change.
- [ ] 🟡 E6. Overfit a single batch to ~0 loss; then deliberately introduce 5 bugs (forgot zero_grad, double softmax, no eval(), wrong dim, labels float) and observe symptoms.
- [ ] 🔴 E7. Train a CNN on CIFAR-10 to ≥85% (augmentation, BN, OneCycle); then ResNet-style with skip connections ≥ 90%.
- [ ] 🔴 E8. Use `torch.profiler` to find the bottleneck (data loader vs GPU); fix it; report before/after.
- [ ] 🔴 E9. Add `torch.compile`; benchmark speedup honestly (warmup, sync).
- [ ] 🔴 E10. Run single-node DDP on 2 GPUs (Kaggle/Colab multi-GPU or cloud) with `torchrun`.

## F. Architectures & transfer (Note 07)
- [ ] 🟡 F1. Implement 2D convolution with loops, then via `unfold`/im2col + matmul; compare to `F.conv2d`.
- [ ] 🟡 F2. Fine-tune pretrained ResNet/ViT on a small custom image dataset; compare feature-extraction vs full fine-tune.
- [ ] 🟡 F3. LSTM character-level language model on Shakespeare; sample text with temperature 0.2/1.0/1.5.
- [ ] 🟡 F4. Train word2vec (skip-gram + negative sampling) on a small corpus; evaluate analogies/nearest neighbors.
- [ ] 🔴 F5. Implement a VAE on MNIST; interpolate in latent space; explain the KL term effect (β-VAE).
- [ ] 🔴 F6. Train a small contrastive model (SimCLR-lite or sentence-pair InfoNCE); show retrieval improving.
- [ ] 🔴 F7. Implement a tiny diffusion model on 2D toy data or MNIST (DDPM): forward noising + noise prediction net + sampling.
- [ ] 🔴 F8. Knowledge distillation: distill a big MLP/CNN into a small one; compare with training small from scratch.

## G. Practical engineering (Note 08)
- [ ] 🟢 G1. Set up W&B or MLflow; log a sweep of 12 runs; build a comparison report.
- [ ] 🟡 G2. Write a Hydra/YAML-driven training script; reproduce a run exactly from logged config + seed.
- [ ] 🟡 G3. Package a model behind a FastAPI endpoint with pre/post-processing in one object; load test with `locust`/`hey`.
- [ ] 🟡 G4. Compute drift on a dataset (PSI & KS test) between "train" and "production-like" slices.
- [ ] 🔴 G5. Write a one-page **model card** for your best model (data, metrics by slice, limits, risks).
- [ ] 🔴 G6. Estimate GPU memory for training a 1B-param model (fp32/bf16, Adam) on paper; then check with a tiny model's actual `memory_allocated`.

## H. Implement-from-scratch interview set (use [[11-Resources-and-Practice-Links]] sites)
Softmax · cross-entropy · logistic regression · k-means · k-NN · decision-tree split · PCA · conv2d · BatchNorm · Adam · LSTM cell · attention (preview) · AUC computation · cosine search top-k · sampling with temperature/top-k.

## Cadence
Daily: 1 exercise. Weekly: 1 mini-write-up (what surprised me). Revisit [[10-QA-Interview-Questions]] every Sunday.
