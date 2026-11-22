---
tags: [phase-2, projects, checkpoint]
---
# 12 — Projects & Phase Checkpoint

> Each repo: README with **Problem · Data · Baseline · Method · Results table (mean±std over seeds) · Error analysis · Limitations · How to run**. Track runs in W&B/MLflow.

## Project 1 — Tabular end-to-end: churn/fraud/price prediction *(Notes 02–04)*
- Dataset: Kaggle (Telco Churn, Credit-Card Fraud, House Prices, or a customer-like dataset).
- Do: EDA → leakage-safe `Pipeline` → baseline (dummy + logistic) → LightGBM + Optuna → calibration → threshold chosen by a *cost matrix* → SHAP → slice analysis → one-page "customer-facing" summary explaining the result in business terms.
- Deliverable: reproducible script + report. (Practices FDE-style communication.)

## Project 2 — From-scratch toolkit *(Notes 01–03, 05)*
NumPy-only library: linear/logistic regression, k-means, PCA, decision tree, MLP with backprop + optimizers (SGD, Adam), with gradient checks and unit tests. Compare each to scikit-learn on real data; document discrepancies.

## Project 3 — `micrograd++` autograd engine *(Note 05)*
Scalar then tensor-level reverse-mode autodiff in Python/NumPy: ops (+, ×, matmul, relu, tanh, exp, log, softmax), topological backward, `Module`/`Linear`/`MLP`, SGD/Adam, train on moons/MNIST. Verify vs PyTorch to 1e-6. Write a blog-style README explaining backprop with your own diagrams.

## Project 4 — Vision or text transfer-learning with rigorous evaluation *(Notes 04, 06, 07)*
- Choose: image classifier (custom 5–10 class dataset you scrape/label) **or** text classifier (support-ticket/news categories).
- Do: train from scratch vs feature-extract vs fine-tune; ablations; 3–5 seeds; confusion matrix; failure gallery; calibration; compute/latency table; model card.
- Serve via FastAPI + Docker (from Phase 1); load test.

## Project 5 (Capstone) — Mini deep-learning stack + tracked research log *(everything)*
1. Implement training loop in PyTorch with: config files, AMP, grad clipping, warmup+cosine, checkpoint/resume, seeding, W&B logging.
2. Train a small **character-level GPT-style or LSTM LM** on a text corpus (follow Karpathy's makemore/nanoGPT approach — you'll finish the Transformer part in Phase 3).
3. Run a **12+ run sweep** (LR, width, depth, dropout); write findings with plots, and a "what I'd try next".
4. Add `torch.compile` + profiling; report speed/memory.
5. Publish repo + a 1–2 page write-up.

## Phase 2 Self-assessment (0–3; need ≥2 before Phase 3)
| Skill | 0–3 |
|---|---|
| Matrix/vector ops, shapes, broadcasting, einsum | |
| Derive gradients for linear/logistic/softmax-CE/MLP | |
| Explain MLE ↔ cross-entropy ↔ KL | |
| Bias–variance, regularization, double descent (explain with a plot) | |
| Correct splits; spot 5 types of leakage | |
| Pick & defend a metric for imbalanced/ranking/regression tasks | |
| Bootstrapped CIs & paired comparisons | |
| Calibration (diagnose & fix) | |
| Trees/boosting hyperparameters & when they win | |
| PCA/UMAP/clustering: use & caveats | |
| Implement backprop by hand & gradient-check | |
| Explain init, normalization, residuals, dropout, optimizers, warmup | |
| Write a PyTorch training loop from memory | |
| Debug NaN/no-learning/overfit/shape/device issues | |
| Mixed precision, grad accumulation, checkpointing | |
| CNN/RNN/embeddings/autoencoder/diffusion concepts | |
| Transfer learning & fine-tuning trade-offs | |
| Experiment tracking & reproducibility | |
| Training–serving skew, drift, monitoring | |
| Communicate model performance to a non-technical stakeholder | |

## Mock interview (60 min)
1. 10 min — "Walk me through training a classifier end to end; what can go wrong?" (leakage, metric, calibration, drift)
2. 20 min — implement softmax + cross-entropy + one training step in NumPy/PyTorch; explain gradient.
3. 15 min — debugging: "loss is NaN after 200 steps" and "val accuracy 99% but production 70%".
4. 15 min — system design: "ML system to route customer-support tickets for a bank" (data, labels, metrics, privacy, rollout, monitoring).

## What you've unlocked
You can now read the Transformer paper and understand every word: embeddings, residuals, LayerNorm, softmax, cross-entropy, AdamW, warmup, bf16. Phase 3 opens the box on LLMs themselves.

⏸️ **Pause point.** Tell me when you're ready and I'll build **Phase 3 — Transformers & LLM Internals** (tokenization, attention, architecture, pretraining, scaling laws, post-training: SFT/RLHF/DPO/GRPO, inference mechanics, sampling, context, multimodal, nanoGPT build, plus exercises, Q&A, resources, projects).

Back to [[00-Phase-2-Index]] · [[../00-Roadmap]]
