---
tags: [phase-2, qa, interview]
---
# 10 — Q&A (Phase 2)

> Answer aloud first. Click to reveal.

## Math & foundations
> [!question]- Q1. Why is the dot product a similarity measure?
> `a·b = ‖a‖‖b‖cosθ`. For normalized vectors it equals cosine of the angle: 1 same direction, 0 orthogonal, −1 opposite. Attention scores and embedding search both use it.

> [!question]- Q2. What does the gradient tell you?
> The direction of steepest increase of a function (a vector of partial derivatives). Gradient descent steps opposite to it to reduce the loss.

> [!question]- Q3. Explain the chain rule's role in backprop.
> The loss is a composition of layer functions; the chain rule says the derivative of a composition is the product of local derivatives. Backprop computes these products from output to input, reusing the upstream gradient at each node, so total cost ≈ one forward pass.

> [!question]- Q4. Why do we minimize negative log-likelihood?
> Likelihood of i.i.d. data is a product; log turns it into a sum (numerically stable, easy gradients) and is monotonic so the maximizer is unchanged. Negate to get a loss to minimize.

> [!question]- Q5. Relationship between MLE, cross-entropy, and KL divergence?
> Minimizing cross-entropy between the data distribution and the model = minimizing `KL(data‖model)` (since entropy of data is constant) = maximizing likelihood.

> [!question]- Q6. Why MSE for regression and cross-entropy for classification?
> They're the NLL under Gaussian-noise and Bernoulli/Categorical assumptions respectively. Cross-entropy with softmax also gives well-behaved gradients (`p−y`) that don't vanish when predictions are confidently wrong, unlike MSE on probabilities.

> [!question]- Q7. L1 vs L2 regularization — geometry & Bayesian view?
> L1 (diamond constraint) hits axes → sparse weights; Laplace prior. L2 (round) shrinks all weights smoothly; Gaussian prior (MAP). L2 has closed-form-friendly gradients; L1 non-differentiable at 0.

> [!question]- Q8. What is KL divergence and is it a distance?
> `KL(p‖q)=Σp log(p/q)`, extra bits from coding `p`-distributed data with a code for `q`. Non-negative, zero iff equal, asymmetric, no triangle inequality → not a metric.

> [!question]- Q9. What is perplexity?
> `exp` of the average per-token cross-entropy: the model's effective branching factor. Lower is better; only comparable with the same tokenization/dataset.

> [!question]- Q10. Eigenvectors/SVD in ML?
> PCA uses the eigenvectors of the covariance (or SVD of centered data) to find max-variance directions; SVD gives optimal low-rank approximations (compression, LSA, intuition for LoRA).

> [!question]- Q11. Explain Bayes' theorem with the rare-disease example.
> Even with a 99% sensitive test, if prevalence is 0.1% most positives are false positives, because the prior is tiny. Posterior = likelihood × prior / evidence. Always consider base rates.

> [!question]- Q12. What does the Central Limit Theorem say and why care?
> Sums/averages of many independent variables approach a Gaussian regardless of source distribution. Justifies confidence intervals, and explains why minibatch gradient noise is roughly Gaussian.

## ML fundamentals
> [!question]- Q13. Explain bias–variance tradeoff.
> Expected error = bias² (systematic error from wrong assumptions/too simple) + variance (sensitivity to training sample) + irreducible noise. Simple models: high bias/low variance; complex: low bias/high variance. Regularization, more data and ensembling reduce variance. (Modern overparameterized nets show double descent.)

> [!question]- Q14. How do you detect and fix overfitting?
> Detect: large train–val gap, val loss rising. Fix: more data/augmentation, regularization (L2, dropout), early stopping, simpler model, ensembling, reduce features, cross-validation for honest estimates.

> [!question]- Q15. Train/val/test — purposes?
> Train: fit parameters. Validation: tune hyperparameters/choose models/early stopping. Test: one-shot unbiased estimate of final performance. Reusing test for decisions leaks information.

> [!question]- Q16. Name five data-leakage patterns.
> Fit scaler/imputer/vectorizer before splitting; duplicates across splits; target leakage features; future information in time series; same entity (user/patient) in train and test; tuning on test.

> [!question]- Q17. Why standardize features?
> Distance-based (kNN, SVM, k-means) and gradient-based models are sensitive to scale; standardizing improves conditioning (faster, stable GD) and makes regularization treat features equally. Trees don't need it.

> [!question]- Q18. Gradient descent vs SGD vs mini-batch?
> GD uses all data per step (exact, slow); SGD one sample (noisy, fast, escapes sharp minima); mini-batch balances noise and hardware efficiency. Noise has regularizing effects.

> [!question]- Q19. What happens with too high / too low learning rate?
> High: oscillation or divergence (NaN). Low: slow convergence/stuck in plateaus. Use LR finder, schedules, warmup.

> [!question]- Q20. Parameter vs hyperparameter?
> Parameters are learned from data (weights); hyperparameters are set before/around training (LR, depth, λ, batch size, architecture) and tuned on validation data.

> [!question]- Q21. Generative vs discriminative?
> Discriminative models learn `P(y|x)` (decision boundary); generative learn `P(x)` or `P(x,y)` and can sample. LLMs are generative autoregressive models.

> [!question]- Q22. How handle class imbalance?
> Right metrics (PR-AUC, recall@precision), class weights/focal loss, resampling (careful—do inside CV folds), threshold tuning, collect more minority data, anomaly-detection framing, calibrated probabilities.

> [!question]- Q23. How handle missing data?
> Understand mechanism (MCAR/MAR/MNAR); impute (median/mode/model-based) inside the pipeline; add missing-indicator features; use models that handle NaNs (LightGBM/XGBoost); drop only when justified.

## Classical algorithms
> [!question]- Q24. Why does logistic regression use cross-entropy, and why is it convex?
> Cross-entropy is the Bernoulli NLL; composition with a linear function and sigmoid gives a convex loss in `w`, so GD finds the global optimum (no local minima).

> [!question]- Q25. How does a decision tree choose splits?
> Greedily picks the feature/threshold giving maximum impurity reduction (Gini/entropy/variance), recursively, until stopping criteria.

> [!question]- Q26. Random forest vs gradient boosting?
> RF: parallel deep trees on bootstrap samples, averaged → lowers variance. GBM: sequential shallow trees each fitting the gradient/residuals of the ensemble → lowers bias; more tunable, usually higher accuracy but riskier to overfit.

> [!question]- Q27. Why do tree-based models dominate tabular data?
> Handle mixed types, missing values, nonlinearities/interactions, irregular target functions, and are robust to uninformative features and scale; deep nets lack these inductive biases and need more tuning. (Gap narrowing with newer tabular foundation models.)

> [!question]- Q28. k-means weaknesses?
> Needs `k`, assumes spherical equal-size clusters, sensitive to init/outliers/scale, local optima. Alternatives: GMM, DBSCAN/HDBSCAN.

> [!question]- Q29. PCA: what and when?
> Linear projection onto orthogonal directions of max variance; for compression, denoising, decorrelation, visualization. Assumes variance = importance; needs scaling; components may be hard to interpret.

> [!question]- Q30. t-SNE/UMAP caveats?
> Great for visual exploration of clusters, but distances between clusters and cluster sizes are not meaningful; results depend on hyperparameters/seed; don't use them as features carelessly.

> [!question]- Q31. What is the kernel trick?
> Compute inner products in a high-dimensional feature space via a kernel function without constructing the features, enabling nonlinear boundaries in SVMs.

> [!question]- Q32. Explain bagging vs boosting vs stacking.
> Bagging: parallel models on resamples, average (variance↓). Boosting: sequential, correct previous errors (bias↓). Stacking: a meta-learner combines base predictions (use out-of-fold preds).

## Evaluation
> [!question]- Q33. Precision vs recall? When prioritize each?
> Precision = TP/(TP+FP) (trustworthiness of positives); recall = TP/(TP+FN) (coverage). Prioritize recall when misses are costly (disease, fraud), precision when false alarms are costly (auto-actions, spam deletion).

> [!question]- Q34. ROC-AUC vs PR-AUC?
> ROC-AUC is the probability a random positive outranks a random negative; insensitive to prevalence and can look great with heavy imbalance. PR-AUC focuses on the positive class and reflects the precision cost of false positives—better for rare events.

> [!question]- Q35. Your model has 99% accuracy. Is that good?
> Not necessarily — check class balance and baseline (majority class), per-class metrics, calibration, slices, and cost of errors.

> [!question]- Q36. What is calibration; how to fix?
> Predicted probabilities match empirical frequencies. Diagnose with reliability diagrams/ECE/Brier; fix with temperature scaling, Platt, isotonic regression on held-out data.

> [!question]- Q37. How do you know model A is really better than model B?
> Same test set, paired comparison with confidence intervals (paired bootstrap/McNemar), multiple seeds, check slices, test practical significance and online A/B test.

> [!question]- Q38. Why cross-validation?
> Gets a lower-variance estimate of generalization and uses data efficiently, esp. with small datasets; must be applied to entire pipeline and respect structure (group/time).

> [!question]- Q39. How would you do error analysis?
> Sample failures, read them, categorize root causes (label noise, ambiguity, missing signal, OOD, bug), quantify categories, fix the biggest first (data/labels/features/rules/model), re-evaluate on fixed set.

> [!question]- Q40. Explain nDCG.
> Discounted cumulative gain sums graded relevance with a log-position discount, normalized by the ideal ranking; rewards putting highly relevant items early.

## Neural networks
> [!question]- Q41. Why do we need nonlinear activations?
> Without them, a stack of linear layers is equivalent to one linear layer; nonlinearity gives expressive power to approximate complex functions.

> [!question]- Q42. Explain backpropagation.
> Forward pass computes and caches activations and the loss; backward pass applies the chain rule from the loss to each parameter, reusing upstream gradients layer by layer; gradients then feed an optimizer.

> [!question]- Q43. Vanishing & exploding gradients: causes and fixes?
> Repeated multiplication of Jacobians with norms <1 or >1 across depth/time. Fixes: ReLU-family activations, proper init (He/Xavier), normalization, residual connections, gated RNNs, gradient clipping, lower LR.

> [!question]- Q44. What's special about softmax + cross-entropy?
> Convert logits to probabilities and measure NLL; gradient w.r.t. logits is `p − y`; use log-softmax/logsumexp for numerical stability (frameworks fuse them — pass raw logits).

> [!question]- Q45. Adam vs SGD?
> Adam keeps running averages of gradient (momentum) and squared gradient (per-parameter scale), giving fast, robust convergence with little tuning; SGD+momentum can generalize slightly better in some vision tasks but needs LR tuning. AdamW fixes weight decay coupling and is standard for transformers.

> [!question]- Q46. Why warmup and LR decay?
> Early on, Adam's second-moment estimates and the random init are unreliable and large LR can destabilize; warmup ramps LR. Decay lets the model settle into a minimum.

> [!question]- Q47. BatchNorm vs LayerNorm?
> BN normalizes across the batch per feature (depends on batch stats; running averages at inference; great for CNNs). LN normalizes across features per example (batch-independent; good for sequences/transformers). RMSNorm drops mean-centering.

> [!question]- Q48. How does dropout work and why scale at train time?
> Randomly zero activations with prob `p` during training to prevent co-adaptation; inverted dropout scales kept activations by `1/(1−p)` so expected activations match at inference (when dropout is off).

> [!question]- Q49. What are residual connections for?
> `y = x + F(x)` gives gradients an identity path, easing optimization of very deep nets and letting layers learn refinements.

> [!question]- Q50. Your loss doesn't decrease. Debug steps?
> Verify data/labels, overfit one batch, check initial loss ≈ `ln(C)`, LR (try 10× up/down), `zero_grad`, loss/logit shapes, `train()/eval()`, normalization, gradient norms/NaNs, simplify model, check data pipeline augmentations.

> [!question]- Q51. Why does batch size matter?
> Larger batch → less gradient noise, better hardware efficiency but may need higher LR/warmup and can generalize worse; smaller → noisier, regularizing, slower per epoch. Gradient accumulation decouples effective batch from memory.

## PyTorch & systems
> [!question]- Q52. Why `optimizer.zero_grad()` each step?
> PyTorch accumulates gradients in `.grad` across backward calls; without clearing, gradients from previous batches add up.

> [!question]- Q53. `model.eval()` vs `torch.no_grad()`?
> `eval()` switches layer behavior (dropout off, BN uses running stats); `no_grad()` disables graph building to save memory/compute. Use both for inference.

> [!question]- Q54. How to fit a bigger model/batch in GPU memory?
> bf16/fp16 mixed precision, gradient accumulation, activation/gradient checkpointing, smaller batch, 8-bit optimizers, LoRA/QLoRA, FSDP/ZeRO sharding, offloading, shorter sequences, FlashAttention.

> [!question]- Q55. What does `torch.compile` do?
> Captures the model into a graph and compiles fused, optimized kernels (via TorchDynamo + Inductor/Triton), reducing overhead and memory traffic; first run has compile cost; dynamic shapes may trigger recompilation.

> [!question]- Q56. DDP vs FSDP?
> DDP replicates the full model on each GPU and averages gradients (simple, memory-heavy). FSDP shards parameters/gradients/optimizer states across GPUs, gathering as needed → fits larger models at some communication cost.

## Architectures & practice
> [!question]- Q57. Why CNNs for images?
> Locality + translation equivariance + weight sharing → fewer parameters, spatial hierarchy of features; good inductive bias for grid data.

> [!question]- Q58. Why did Transformers replace RNNs?
> RNNs process sequentially (hard to parallelize) and struggle with long-range dependencies; attention connects any two positions directly and parallelizes over sequence length, scaling better with compute/data.

> [!question]- Q59. When fine-tune vs train from scratch vs feature-extract?
> Little data/similar domain → feature extraction or light fine-tuning; moderate data → full fine-tune with small LR; huge, very different domain with compute → from scratch/continued pretraining.

> [!question]- Q60. What is training–serving skew?
> Differences between how features are computed in training vs production (code, data freshness, missing handling), causing degraded live performance; prevent with shared code paths, feature stores, and monitoring.

> [!question]- Q61. What's contrastive learning/InfoNCE?
> Learn embeddings by pulling positive pairs together and pushing negatives apart using a softmax over similarities with temperature; powers CLIP and text embedding models used in RAG.

> [!question]- Q62. Explain a VAE's loss.
> ELBO = reconstruction likelihood − `KL(q(z|x)‖p(z))`; the KL regularizes the latent distribution toward the prior so sampling from the prior gives valid outputs; reparameterization trick enables gradients.

> [!question]- Q63. How does diffusion generate images?
> Forward process gradually adds Gaussian noise; a network learns to predict the noise (or score/velocity) at each timestep; sampling starts from noise and iteratively denoises, optionally guided by text conditioning (classifier-free guidance).

> [!question]- Q64. How would you explain a model's 85% precision / 60% recall to a non-technical customer?
> "When it flags a case, it's right 17 out of 20 times; it catches 6 of every 10 real cases. We can trade: catch more at the cost of more false alarms — here's the dollar impact of each setting." Tie to their cost of misses vs false alarms.

Next → [[11-Resources-and-Practice-Links]]
