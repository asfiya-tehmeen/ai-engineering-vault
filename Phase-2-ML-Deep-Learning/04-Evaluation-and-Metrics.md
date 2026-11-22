---
tags: [phase-2, evaluation, metrics]
---
# 04 — Evaluation & Metrics

> **Evals are the most under-rated skill in AI engineering.** Every habit here (representative test sets, right metric, slices, uncertainty, error analysis) carries directly into LLM evals in Phase 7.

## 1. Classification metrics
**Confusion matrix** (binary): TP, FP, FN, TN.

| Metric | Formula | Use / trap |
|---|---|---|
| Accuracy | (TP+TN)/all | misleading on imbalance (99% negatives → 99% by predicting all-negative) |
| **Precision** | TP/(TP+FP) | "when I flag it, am I right?" — cost of false alarms (spam filter, content moderation) |
| **Recall (TPR, sensitivity)** | TP/(TP+FN) | "did I catch them all?" — cost of misses (cancer, fraud) |
| Specificity (TNR) | TN/(TN+FP) | |
| **F1** | 2PR/(P+R) | harmonic mean; Fβ weights recall (β>1) or precision (β<1) |
| ROC-AUC | area under TPR vs FPR | threshold-free; optimistic on heavy imbalance |
| **PR-AUC / Average Precision** | area under precision–recall | better for rare positives |
| Log loss (cross-entropy) | `−mean log p_y` | evaluates probabilities, punishes confident mistakes |
| Brier score | mean `(p−y)²` | calibration + discrimination |
| MCC | balanced correlation | good single number for imbalance |
| Cohen's κ | agreement beyond chance | also for human-labeler agreement |

- **Multiclass/multilabel averaging:** *macro* (mean over classes, equal weight), *micro* (pool all), *weighted*. Choose consciously; macro exposes poor performance on rare classes.
- **Thresholds:** `predict_proba` + choose threshold by business cost: minimize `cost_FP·FP + cost_FN·FN`, or fix precision ≥ X and maximize recall.
- **Top-k accuracy**, **confusion matrix per class**, per-slice metrics.

## 2. Regression metrics
MSE, **RMSE** (same units), **MAE** (robust), **MAPE/sMAPE** (percent; breaks near zero), **R²** (variance explained; can be negative on test), **quantile/pinball loss** (prediction intervals), residual analysis (heteroscedasticity, patterns ⇒ missing features).

## 3. Ranking & retrieval metrics (essential for RAG, Phase 5)
- **Precision@k, Recall@k, Hit rate@k**
- **MRR** (mean reciprocal rank of first relevant)
- **MAP**, **nDCG@k** (graded relevance with position discount: `DCG = Σ rel_i / log₂(i+1)`)
- Pairwise accuracy; AUC.

## 4. Generation / NLP metrics (preview)
BLEU, ROUGE (n-gram overlap — weak), **perplexity** (LM fit), BERTScore (embedding similarity), exact match/F1 for QA, pass@k for code, human preference / Elo, **LLM-as-judge** (Phase 7). n-gram metrics correlate poorly with quality for open-ended generation.

## 5. Calibration
- A model is **calibrated** if among predictions with `p≈0.8`, ~80% are positive.
- **Reliability diagram**, **ECE (expected calibration error)**.
- Fix: **Platt scaling** (logistic on logits), **isotonic regression**, **temperature scaling** (single param on logits). Deep nets/LLMs are often overconfident.

## 6. Experimental design
- **Fixed, versioned test set** representative of production (include hard & edge cases). Don't tune on it.
- **Sample size & uncertainty:** report confidence intervals (bootstrap). With 100 test items, a 3-pt gain is noise. `SE ≈ √(p(1−p)/n)`.
- **Compare models with paired tests** on the *same* items (paired bootstrap, McNemar's test).
- **Random seeds:** run ≥3–5 seeds; report mean ± std.
- **Slices/subgroups:** metrics by segment (language, device, customer, class, length). Aggregate improvement can hide regressions and fairness issues.
- **Offline vs online metrics:** offline eval proxies; confirm with A/B tests (guardrail metrics!).
- **Metric gaming / Goodhart's law:** when a measure becomes a target, it ceases to be a good measure.
- **Label quality:** inter-annotator agreement; noisy labels cap achievable metrics (estimate the "human ceiling").
- **Benchmark contamination:** test data leaked into training (huge issue for LLM benchmarks).

## 7. Cross-validation recipes
- `StratifiedKFold` for classification; `GroupKFold` for grouped entities; `TimeSeriesSplit`/walk-forward for time; `RepeatedKFold` for small data; nested CV for model selection + unbiased estimate.

## 8. Hyperparameter tuning
`GridSearchCV` → `RandomizedSearchCV` → **Optuna** (TPE/Bayesian, pruning). Limit search space sensibly; log all trials; re-evaluate the *final* choice on untouched test once. Beware "winner's curse": best-of-many val scores are optimistic.

## 9. Error analysis (where real improvement comes from)
1. Collect misclassified examples (e.g., 100). 2. **Read them.** 3. Tag by failure category (label noise, ambiguous, missing feature, OOD, formatting…). 4. Count categories → prioritize. 5. Fix (data, features, labels, rules, model). 6. Re-measure. *This exact loop drives LLM system improvement in Phase 7.*

## 10. Learning curves & diagnostics
- **Learning curve** (error vs training size): converging high → bias problem (need better model/features); big gap → variance problem (more data/regularization).
- **Validation curve** (error vs hyperparameter). **Loss curves** over epochs.
- Sanity checks: overfit a tiny batch (should reach ~0 loss); shuffle labels (should perform at chance; if not, leakage); train on random features.

## 11. Fairness & robustness checks
Per-group metrics (demographic parity, equalized odds, calibration within groups) — tradeoffs are inevitable; document. Robustness: noise, typos, paraphrase, distribution shift tests. Adversarial examples.

## 12. Experiment tracking
Log params, data version, code commit, metrics, artifacts: **Weights & Biases, MLflow, TensorBoard, Comet, Neptune**. Name runs; save configs. (See [[08-Practical-ML-Engineering]].)

## Metric selection cheat sheet
| Problem | Primary metric |
|---|---|
| Balanced classification | accuracy / macro-F1 |
| Rare-event detection | PR-AUC, recall at fixed precision |
| Cost-sensitive | expected cost at chosen threshold |
| Probability outputs matter | log loss, Brier, calibration |
| Retrieval | recall@k, nDCG@k, MRR |
| Forecasting | MAE/RMSE + prediction interval coverage |
| Generative | task success rate + human/LLM-judge rubric |

Next → [[05-Neural-Networks-and-Backprop]]
