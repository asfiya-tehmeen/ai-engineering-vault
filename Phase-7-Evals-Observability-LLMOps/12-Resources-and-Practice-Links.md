---
tags: [phase-7, resources]
---
# 12 — Resources & Practice Links (Phase 7)

> ⭐ = top pick. I haven't opened every link; if one moves, search the title. arXiv IDs are from memory. Regulation and standards change — use **primary sources and counsel** for legal questions; my research surfaced secondary legal commentary for the EU AI Act timeline, and no primary text for ISO/IEC 42001 or NIST AI RMF.

## Evals (start here)
1. ⭐ Hamel Husain — *Your AI Product Needs Evals* https://hamel.dev/blog/posts/evals/ · *LLM Evals: Frequently Asked Questions* https://hamel.dev/blog/posts/evals-faq/ · more at https://hamel.dev/
2. ⭐ Eugene Yan — *Product Evals in Three Simple Steps* https://eugeneyan.com/writing/product-evals/ · *Evaluating the Effectiveness of LLM-Evaluators (aka LLM-as-Judge)* https://eugeneyan.com/writing/llm-evaluators/
3. ⭐ Shreya Shankar et al. — *Who Validates the Validators?* (aligning LLM-assisted evaluation with human preferences) https://arxiv.org/abs/2404.12272 · EvalGen
4. Anthropic engineering — *Demystifying evals for AI agents* (search title at https://www.anthropic.com/engineering) · Anthropic docs: "Define success criteria / Develop test cases" in the prompt-engineering section (docs.claude.com)
5. OpenAI — Evals guide in platform docs https://platform.openai.com/docs/guides/evals · OpenAI Cookbook eval examples https://cookbook.openai.com/
6. Judge methodology papers: *Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena* https://arxiv.org/abs/2306.05685 · *G-Eval* https://arxiv.org/abs/2303.16634 · *Prometheus* https://arxiv.org/abs/2310.08491 · position/verbosity bias studies (search "LLM judge position bias") · 2026 preprints on judge uncertainty/drift (search "Bias and Uncertainty in LLM-as-a-Judge Estimation", "Frozen Judges, Moving Agents")
7. Calibration how-tos (2026 blog/vendor guides — treat thresholds as rules of thumb): MLflow *LLM Judge Calibration: Measure, Correct and Report* https://mlflow.org/articles/llm-judge-calibration/ · Future AGI *LLM-as-Judge Best Practices in 2026* https://futureagi.com/blog/llm-as-judge-best-practices-2026/
8. Statistics: *Adding Error Bars to Evals* (Anthropic; Miller 2024) https://arxiv.org/abs/2411.00640 · Efron & Tibshirani *Intro to the Bootstrap* · Rogan–Gladen estimator (search "Rogan Gladen prevalence misclassification")

### Eval tools & platforms
- **Promptfoo** (YAML evals, CI, red-team) https://www.promptfoo.dev/docs/ · **Inspect AI** (UK AISI) https://inspect.aisi.org.uk/ · **DeepEval** https://docs.confident-ai.com/ · **RAGAS** https://docs.ragas.io/ · **lm-evaluation-harness** https://github.com/EleutherAI/lm-evaluation-harness · **HELM** https://crfm.stanford.edu/helm/ · **MLflow GenAI evaluation** https://mlflow.org/docs/latest/genai/
- Platforms: **Braintrust** https://www.braintrust.dev/docs · **Langfuse** https://langfuse.com/docs · **LangSmith** https://docs.smith.langchain.com/ · **Arize Phoenix** https://docs.arize.com/phoenix · W&B Weave https://docs.wandb.ai/weave · Comet Opik https://www.comet.com/docs/opik/ · Galileo · Patronus · Confident AI
- Practice sets & benchmarks to study methodology: MT-Bench, IFEval, HumanEval/MBPP (code), BIRD/Spider (SQL), τ-bench (agents), HarmBench (safety)

## Observability & LLMOps
- ⭐ **OpenTelemetry GenAI semantic conventions** — per 2026 sources these moved to https://github.com/open-telemetry/semantic-conventions-genai (status: **Development**) · agent spans spec: https://opentelemetry.io/docs/specs/semconv/gen-ai/gen-ai-agent-spans/ · general spans https://opentelemetry.io/docs/specs/semconv/gen-ai/gen-ai-spans/ (older docs pages may redirect)
- Explainers (verify against the spec; some posts conflict on stability): Dash0 *OpenTelemetry GenAI semantic conventions explained* https://www.dash0.com/knowledge/opentelemetry-genai-semantic-conventions-explained · Greptime post (May 2026) https://www.greptime.com/blogs/2026-05-09-opentelemetry-genai-semantic-conventions
- Instrumentation: OpenLLMetry https://github.com/traceloop/openllmetry · OpenInference https://github.com/Arize-ai/openinference · `opentelemetry-instrumentation-openai-v2` (OTel contrib) · Pydantic Logfire https://pydantic.dev/logfire
- Backends: Langfuse · Phoenix · Datadog LLM Observability · Grafana Tempo/Loki/Prometheus · Honeycomb · New Relic · Helicone https://docs.helicone.ai/ · Portkey https://portkey.ai/docs
- Books/essays: Chip Huyen *AI Engineering* (O'Reilly 2025) and *Designing Machine Learning Systems* · Google SRE Book (SLOs, error budgets) https://sre.google/books/ · Eugene Yan *Patterns for LLM Systems* https://eugeneyan.com/writing/llm-patterns/
- MLOps context: Made With ML https://madewithml.com/ · Full Stack Deep Learning https://fullstackdeeplearning.com/ · MLflow https://mlflow.org/ · DVC https://dvc.org/ · Evidently (drift) https://www.evidentlyai.com/

## Release management & experimentation
- Experimentation: *Trustworthy Online Controlled Experiments* (Kohavi, Tang, Xu) — the A/B testing bible · Statsig/Eppo/GrowthBook/LaunchDarkly docs (feature flags & experiments)
- Prompt management: Langfuse prompt mgmt · LangSmith Hub · Braintrust prompts · PromptLayer · Git-based workflows
- CI/CD: GitHub Actions https://docs.github.com/actions · Argo Rollouts/Flagger (progressive delivery) · Temporal for workflows
- Provider migration guides & deprecation pages (read the current ones for the models you use)

## Fine-tuning
- ⭐ Hugging Face **TRL** https://huggingface.co/docs/trl · **PEFT** https://huggingface.co/docs/peft · Transformers · bitsandbytes
- ⭐ **Unsloth** https://docs.unsloth.ai/ · **Axolotl** https://github.com/axolotl-ai-cloud/axolotl · **torchtune** https://github.com/pytorch/torchtune · **LLaMA-Factory** https://github.com/hiyouga/LLaMA-Factory · verl https://github.com/volcengine/verl · OpenRLHF
- Papers: **LoRA** https://arxiv.org/abs/2106.09685 · **QLoRA** https://arxiv.org/abs/2305.14314 · DoRA https://arxiv.org/abs/2402.09353 · *LoRA Learns Less and Forgets Less* https://arxiv.org/abs/2405.09673 · DPO https://arxiv.org/abs/2305.18290 · LIMA https://arxiv.org/abs/2305.11206 · Distilling the Knowledge in a Neural Network https://arxiv.org/abs/1503.02531 · "LoRA Without Regret" (Thinking Machines blog; search title)
- Guides: Maxime Labonne's LLM course https://github.com/mlabonne/llm-course · Sebastian Raschka (Ahead of AI) https://magazine.sebastianraschka.com/ · Lightning AI LoRA insights · Hugging Face *smol course* / learn hub https://huggingface.co/learn
- Hosted fine-tuning docs: OpenAI fine-tuning (SFT/DPO/RFT—check current options) https://platform.openai.com/docs/guides/fine-tuning · Google Vertex/Gemini tuning · AWS Bedrock customization · Anthropic/Bedrock model customization availability (check docs) · Together/Fireworks fine-tuning
- Compute: Colab/Kaggle, Modal, RunPod, Lambda, Vast.ai, SageMaker/Vertex

## Serving & infrastructure
- ⭐ vLLM https://docs.vllm.ai/ · SGLang https://docs.sglang.ai/ · TensorRT-LLM https://github.com/NVIDIA/TensorRT-LLM · NVIDIA Dynamo · llama.cpp https://github.com/ggml-org/llama.cpp · Ollama · LMDeploy · Ray Serve LLM · KServe https://kserve.github.io/website/ · BentoML
- Benchmarking: **GuideLLM** https://github.com/vllm-project/guidellm · NVIDIA **GenAI-Perf** · k6 / Locust · vLLM benchmark scripts
- Gateways: **LiteLLM** https://docs.litellm.ai/ · Portkey · Cloudflare AI Gateway https://developers.cloudflare.com/ai-gateway/ · Kong AI Gateway
- Papers/guides: PagedAttention (vLLM) https://arxiv.org/abs/2309.06180 · S-LoRA https://arxiv.org/abs/2310.18547 · *Efficient LLM Serving survey* https://arxiv.org/abs/2312.15234 · Kipply *Transformer Inference Arithmetic* https://kipply.github.io/blog/2022/transformer_inference_arithmetic/ · EleutherAI *Transformer Math 101* https://blog.eleuther.ai/transformer-math/
- Kubernetes for GPUs: NVIDIA GPU Operator docs · KEDA https://keda.sh/ · Karpenter

## Guardrails & security
- ⭐ **OWASP GenAI Security Project** — LLM Top 10 & Agentic Top 10 https://genai.owasp.org/ · **MITRE ATLAS** https://atlas.mitre.org/ · Google SAIF https://saif.google/
- Guardrail tools: **Llama Guard / Prompt Guard (PurpleLlama)** https://github.com/meta-llama/PurpleLlama · **NeMo Guardrails** https://github.com/NVIDIA/NeMo-Guardrails · **Guardrails AI** https://www.guardrailsai.com/docs · **Presidio** https://microsoft.github.io/presidio/ · cloud: AWS Bedrock Guardrails, Azure AI Content Safety / Prompt Shields, Google Model Armor, OpenAI moderation · Lakera Guard
- Red-teaming: **Garak** https://github.com/NVIDIA/garak · **PyRIT** https://github.com/Azure/PyRIT · promptfoo red-team https://www.promptfoo.dev/docs/red-team/ · **AgentDojo** https://arxiv.org/abs/2406.13352 · HarmBench https://www.harmbench.org/ · Gandalf https://gandalf.lakera.ai/ · Gray Swan arena https://www.grayswan.ai/ · HackAPrompt
- Supply chain: safetensors https://github.com/huggingface/safetensors · Sigstore/cosign · pip-audit · Trivy · OpenSSF Scorecard

## Governance, compliance & FinOps
- **EU AI Act:** Act Explorer & digital omnibus tracker https://artificialintelligenceact.eu/ · EU AI Office / Official Journal (primary; verify timelines) · secondary summaries surfaced in my research: Usercentrics, Cooley, DLA Piper, Formalize, regulation-ai.eu (*Regulation (EU) 2026/1744* page https://www.regulation-ai.eu/en/omnibus/)
- **NIST AI RMF** and **Generative AI Profile (NIST AI 600-1)** https://www.nist.gov/itl/ai-risk-management-framework · **ISO/IEC 42001** (AI management systems) and ISO/IEC 23894 (AI risk) — purchase/summary via iso.org
- Privacy/security: GDPR text & EDPB guidance https://edpb.europa.eu/ · HIPAA (HHS) · SOC 2 (AICPA) · ISO 27001 · Cloud Security Alliance AI controls https://cloudsecurityalliance.org/ai-safety-initiative
- Documentation: *Model Cards for Model Reporting* https://arxiv.org/abs/1810.03993 · *Datasheets for Datasets* https://arxiv.org/abs/1803.09010 · Google PAIR https://pair.withgoogle.com/ · Microsoft Responsible AI resources · Fairlearn https://fairlearn.org/
- FinOps: FinOps Foundation (AI/Gen-AI working groups) https://www.finops.org/ · provider usage/cost APIs & dashboards · Langfuse/Helicone cost tracking

## Courses & talks
- ⭐ **AI Evals For Engineers & PMs** (Hamel Husain & Shreya Shankar; paid course) — search title · DeepLearning.AI: *Evaluating AI Agents*, *Quality and Safety for LLM Applications*, *Finetuning Large Language Models*, *Red Teaming LLM Applications* https://www.deeplearning.ai/short-courses/
- Hugging Face learn hub (fine-tuning, smol course) · Full Stack LLM Bootcamp · AI Engineer conference talks on evals/observability https://www.ai.engineer/ · Latent Space https://www.latent.space/ evals episodes

## Practice
- **Build an eval for something you've already built** (Phases 4–6 projects) and publish the dataset, graders, and report.
- Replicate a judge-calibration study on a public dataset with human labels (e.g., MT-Bench human judgments, SummEval) and report TPR/TNR/κ + corrected estimates.
- Run **Promptfoo/Inspect** on a public repo's prompt/agent; contribute a fix.
- Load-test an open model with GuideLLM/GenAI-Perf and publish the sizing memo.
- Fine-tune a small model on a public task (e.g., text-to-SQL on Spider subset) and compare to prompting.
- Red-team a public demo (with permission) or Gandalf-style CTFs; write up attack categories and defenses.
- Draft a system card + DPIA-style assessment for an open-source RAG/agent app.

## If you only have 1 week
Day 1: error analysis on 100 traces → taxonomy. Day 2: code graders + dataset v1 + CIs. Day 3: calibrated binary judge (TPR/TNR + corrected rate). Day 4: CI gate + tracing. Day 5: canary + rollback drill. Day 6: LoRA vs prompt decision memo (small experiment). Day 7: guardrail + red-team mini-sprint and governance one-pager.

Next → [[13-Projects-and-Checkpoint]]
