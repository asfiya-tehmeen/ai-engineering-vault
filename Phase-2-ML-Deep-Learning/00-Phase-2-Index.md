---
tags: [phase-2, index]
---
# Phase 2 — ML & Deep Learning Foundations

> [!abstract] Why this phase exists
> LLMs are neural networks trained with gradient descent on a loss function. If you only know the API, you can't reason about *why* a model fails, why fine-tuning overfits, why evals lie, why embeddings cluster, or why a "better" model is slower. This phase gives you the model of the machine: **data → model → loss → gradients → update → evaluation**. Everything in Phases 3–9 is a specialization of it.

**Duration:** 5–8 weeks · **Prereq:** [[../Phase-1-Foundations/00-Phase-1-Index|Phase 1]] (Python, NumPy, pandas)

## Topic map
| # | Note | Core ideas |
|---|---|---|
| 01 | [[01-Math-Foundations]] | linear algebra, calculus, probability, statistics, information theory (entropy, cross-entropy, KL) |
| 02 | [[02-ML-Fundamentals]] | supervised/unsupervised, loss, gradient descent, generalization, bias–variance, regularization, data splits, leakage |
| 03 | [[03-Classical-ML-Algorithms]] | linear/logistic regression, kNN, trees, forests, boosting, SVM, naive Bayes, clustering, PCA, UMAP, scikit-learn |
| 04 | [[04-Evaluation-and-Metrics]] | accuracy/precision/recall/F1/ROC/PR, regression metrics, ranking metrics, calibration, CV, tuning, error analysis |
| 05 | [[05-Neural-Networks-and-Backprop]] | perceptron, MLP, activations, backprop (derived), init, normalization, dropout, optimizers, LR schedules |
| 06 | [[06-PyTorch]] | tensors, autograd, nn.Module, training loop, DataLoader, GPU, mixed precision, checkpointing, debugging |
| 07 | [[07-Deep-Architectures]] | CNNs, RNN/LSTM/GRU, seq2seq, embeddings (word2vec), autoencoders/VAEs/GANs/diffusion, transfer learning, self-supervised & contrastive learning |
| 08 | [[08-Practical-ML-Engineering]] | reproducibility, experiment tracking, GPUs/cloud, Hugging Face intro, MLOps, responsible ML |
| 09 | [[09-Exercises]] | 70+ exercises incl. implement-from-scratch |
| 10 | [[10-QA-Interview-Questions]] | 60 Q&A |
| 11 | [[11-Resources-and-Practice-Links]] | courses, books, problem sites |
| 12 | [[12-Projects-and-Checkpoint]] | 5 projects + self-assessment |

## Weekly plan (8 weeks; compress if experienced)
| Week | Focus | Output |
|---|---|---|
| 1 | Note 01 (math) + 3B1B videos | notebook: vectors/matrices/derivatives/probability by hand + NumPy |
| 2 | Note 02, 03 (linear/logistic, gradient descent from scratch) | linear & logistic regression in pure NumPy |
| 3 | Note 03 (trees/boosting/clustering/PCA) + Note 04 | Kaggle Titanic / House Prices end-to-end with proper CV |
| 4 | Note 05 (backprop by hand) | micrograd-style autograd engine + tiny MLP |
| 5 | Note 06 (PyTorch) | MNIST/Fashion-MNIST classifier with clean training loop |
| 6 | Note 07 (CNN, RNN, embeddings, transfer learning) | fine-tune a pretrained vision/text model |
| 7 | Note 08 + exercises + Q&A | experiment tracking, reproducibility report |
| 8 | Capstone | [[12-Projects-and-Checkpoint\|Capstone: from-scratch deep learning library + tracked experiments]] |

## Completion checklist
- [ ] Compute gradients by hand for a 2-layer net and verify with autograd
- [ ] Explain bias–variance, overfitting, regularization with a plot you generated
- [ ] Never leak test data (can name 5 leakage patterns)
- [ ] Choose the right metric and justify it for an imbalanced problem
- [ ] Implement logistic regression + a tiny MLP + autograd from scratch
- [ ] Write a clean PyTorch training loop (train/val, scheduler, clip, AMP, checkpoint) without a tutorial
- [ ] Explain cross-entropy, softmax, KL, perplexity, and why LLMs minimize next-token cross-entropy
- [ ] Explain what Adam/AdamW does and why LR warmup is used
- [ ] Fine-tune a pretrained model and report results with variance across seeds
- [ ] Capstone on GitHub with README

> [!tip] How deep to go on math
> You need *fluency, not a PhD*. Target: read an equation in a paper and know what each shape/term is, and implement it. If a derivation stalls you for >1 hour, implement + verify numerically (finite differences / autograd), then return later.

Back to [[../00-Roadmap]].
