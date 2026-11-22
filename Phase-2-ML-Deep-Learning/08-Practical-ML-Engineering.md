---
tags: [phase-2, mlops, practical]
---
# 08 — Practical ML Engineering

## 1. Project hygiene
```
project/
  data/ (git-ignored; DVC / object store)   notebooks/   src/proj/ (data.py model.py train.py eval.py)
  configs/ (yaml: hydra / pydantic)   tests/   pyproject.toml   README.md   Makefile
```
- Separate **config from code**; one command to train (`python -m proj.train --config configs/base.yaml`).
- **Notebooks for exploration only**; export logic to modules; test it.
- Seeds, pinned deps (`uv.lock`), recorded git commit + data version with every run.

## 2. Data work (the real job)
- **Data versioning & lineage:** DVC, lakeFS, Delta/Iceberg, dataset hashes, snapshot IDs.
- **Validation:** schema + distribution checks (Pandera, Great Expectations, Pydantic). Fail fast on bad data.
- **Labeling:** guidelines, pilot rounds, inter-annotator agreement, adjudication, active learning, tools (Label Studio, Argilla, Prodigy), weak supervision (Snorkel), synthetic/LLM-assisted labeling (with human audit).
- **Data quality > quantity:** dedupe (exact/near via MinHash), filter, balance, remove PII, check label noise (cleanlab / confident learning).
- **Splits** defined once & frozen; hold out "golden" evaluation sets.
- **Privacy/compliance:** PII scrubbing, consent, retention, GDPR/HIPAA, differential privacy (concept), data residency — core FDE topics.

## 3. Experiment tracking & management
- Track: hyperparams, metrics (train/val curves), system metrics, artifacts (checkpoints, plots, confusion matrices), code/data versions.
- Tools: **Weights & Biases**, **MLflow**, TensorBoard, Comet, Neptune, Aim. Hydra/Optuna for sweeps.
- Habit: **one change per experiment**; keep an experiment log (hypothesis → result → decision) in your Obsidian vault.

## 4. Compute
- **Where:** Colab/Kaggle (free tiers), local GPU, cloud (AWS/GCP/Azure), GPU clouds (Lambda, RunPod, Modal, CoreWeave, Vast.ai), Apple Silicon (MPS) for small stuff.
- **GPU basics:** VRAM size is the main constraint; A100/H100/H200/B200 (data-center), RTX 4090/5090 (consumer). Cost/hour awareness; spot/preemptible instances with checkpointing; **always shut down instances**.
- **Back-of-envelope:** training FLOPs ≈ `6 · params · tokens`; inference memory ≈ `params × bytes/param` (fp16=2 B, int8=1 B, 4-bit=0.5 B) + KV cache. (Phase 3/7 expand.)

## 5. Hugging Face ecosystem (first look)
```python
from transformers import AutoTokenizer, AutoModelForSequenceClassification, pipeline
clf = pipeline("sentiment-analysis")             # downloads a default model
tok = AutoTokenizer.from_pretrained("distilbert-base-uncased")
model = AutoModelForSequenceClassification.from_pretrained("distilbert-base-uncased", num_labels=2)
from datasets import load_dataset
ds = load_dataset("imdb")
```
- **Hub** (models/datasets/Spaces), `transformers`, `datasets`, `tokenizers`, `evaluate`, `peft`, `trl`, `accelerate`, `sentence-transformers`, `diffusers`, `safetensors`. Read **model cards** (license! intended use, limitations, training data). Fine-tuning in Phase 7.

## 6. From model to product (MLOps overview)
| Stage | Concerns |
|---|---|
| Packaging | model + preprocessing in one artifact; ONNX/TorchScript/safetensors; Docker |
| Serving | batch vs online; REST/gRPC; FastAPI, BentoML, Ray Serve, TorchServe, Triton, vLLM (LLMs) |
| Performance | latency (p50/p99), throughput, batching, quantization, caching, GPU utilization |
| Deployment | canary, shadow, blue/green, A/B, rollback, feature flags |
| Monitoring | input drift (PSI, KS test), prediction drift, data quality, latency/errors, business KPIs, delayed labels |
| Retraining | triggers (schedule/drift), pipelines (Airflow/Dagster/Kubeflow/Metaflow/Prefect), model registry, approvals |
| Feature stores | Feast, Tecton — consistent train/serve features; avoid **training–serving skew** |
- **Training–serving skew:** different preprocessing at inference than training — a top silent failure. Share the exact code path.
- **Concept/data drift → silent performance decay.** Monitor and set retraining criteria.
- **Cost:** compute, labeling, human review, opportunity cost.

## 7. ML system design framework (interview-ready)
1. Clarify objective & success metric (business → ML metric). 2. Constraints (latency, scale, privacy, budget). 3. Data (sources, labels, volume, freshness). 4. Baseline (heuristic / simple model). 5. Features/model choice & why. 6. Training & eval plan (splits, metrics, slices). 7. Serving architecture. 8. Monitoring & feedback loop. 9. Risks (bias, privacy, failure modes, abuse). 10. Iteration roadmap.
Practice prompts: fraud detection, recommendation feed, search ranking, spam filter, demand forecasting, support-ticket router, document classification for a bank.

## 8. Responsible ML / AI ethics (a must for FDE)
- **Bias sources:** sampling, historical, measurement, label, aggregation, deployment. **Fairness definitions** conflict (parity vs equalized odds vs calibration); choose with stakeholders; document.
- **Explainability & documentation:** model cards, datasheets for datasets, decision logs.
- **Privacy:** membership inference, training-data extraction, federated learning, DP-SGD, data minimization.
- **Safety & misuse:** dual-use, abuse monitoring, human oversight, red-teaming.
- **Regulation:** EU AI Act (risk tiers), GDPR (automated decisions, right to explanation), US sector rules (HIPAA, ECOA), NIST AI RMF, ISO/IEC 42001. High-stakes domains require human-in-the-loop.
- **Environmental cost** of training/inference.

## 9. Communication of ML results (FDE skill)
Explain with the **baseline**, a **plain-language metric** ("catches 8 of 10 frauds, 1 false alarm per 20 flags"), **error examples**, **uncertainty**, **limits**, and **cost/benefit**. Show a confusion matrix annotated with dollar impact. Avoid jargon; avoid "99% accurate" without context.

Next → [[09-Exercises]]
