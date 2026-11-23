---
tags: [roadmap, index]
---
# 🗺️ AI Engineering + Forward Deployed Engineering — Master Roadmap

> [!abstract] Goal
> Go from zero to a hireable **AI Engineer / Forward Deployed Engineer (FDE)**: someone who builds *and ships* LLM-powered systems (RAG, agents, evals, production infra) **and** can sit with a customer, understand a messy real-world problem, and deploy a working solution inside their environment.

## What the two roles actually are

| | **AI Engineer** | **Forward Deployed Engineer (FDE)** |
|---|---|---|
| Core job | Build products/features on top of pre-trained models (not training models from scratch) | Embed with customers, scope ambiguous problems, build & integrate solutions on-site/remote |
| Day to day | RAG, agents, evals, prompts, tool integrations, latency/cost tuning | Everything an AI engineer does **+** discovery calls, demos, data/system integration, security reviews, stakeholder comms |
| Differentiator | Depth in LLM systems + software engineering | Same depth + communication, product judgment, autonomy in undefined problems |
| Origin | Software eng + applied ML | Palantir "Forward Deployed Software Engineer"; now at OpenAI, Anthropic, Databricks, Scale, Ramp, etc. |

The **shared trunk** (Phases 1–7) is identical. Phase 8 is the FDE-specific layer. Phase 9 is advanced/frontier depth + career.

> [!tip] How the market sees it (2026)
> Hiring guides consistently list: strong Python/software engineering → RAG → agents → **evals** (the most under-rated skill) → production/LLMOps → portfolio projects documenting *problem, architecture, evals, risks, tradeoffs*. Realistic timeline from solid programmer to job-ready: ~6–9 months part-time-intense. From zero programming: add 3–4 months.

## The phases

| # | Phase | You'll be able to… | Est. time |
|---|---|---|---|
| 1 | [[Phase-1-Foundations/00-Phase-1-Index\|Software & Python Foundations]] | Write, test, version, and containerize real Python programs; call and build HTTP APIs; query SQL | 6–10 wks |
| 2 | [[Phase-2-ML-Deep-Learning/00-Phase-2-Index\|ML & Deep Learning Foundations]] | Explain & implement gradient descent, neural nets, backprop in PyTorch; evaluate models properly | 4–6 wks |
| 3 | [[Phase-3-Transformers-LLM-Internals/00-Phase-3-Index\|Transformers & LLM Internals]] | Explain tokenization, embeddings, attention, pretraining → SFT → RLHF/DPO/RLVR, KV cache, sampling, context windows | 4–6 wks |
| 4 | [[Phase-4-Building-with-LLM-APIs/00-Phase-4-Index\|Building with LLM APIs]] | Prompting, structured outputs, tool use, streaming, caching, multimodal, context engineering, cost control | 3–4 wks |
| 5 | [[Phase-5-RAG-Knowledge-Systems/00-Phase-5-Index\|RAG & Knowledge Systems]] | Build & debug retrieval systems: chunking, embeddings, vector/hybrid search, reranking, GraphRAG, RAG evals | 4–5 wks |
| 6 | [[Phase-6-Agents-Tools-MCP/00-Phase-6-Index\|Agents, Tools & MCP]] | Design agent loops, tool schemas, memory, multi-agent systems, MCP servers, guardrails | 4–6 wks |
| 7 | [[Phase-7-Evals-Observability-LLMOps/00-Phase-7-Index\|Evals, Observability & LLMOps]] | Build eval suites, LLM-as-judge, tracing, CI for prompts, serving, fine-tuning (LoRA), security (prompt injection) | 5–6 wks |
| 8 | [[Phase-8-Forward-Deployed-Engineering/00-Phase-8-Index\|Forward Deployed Engineering]] | Run discovery, scope, prototype fast, integrate with enterprise systems, handle security/compliance, demo & communicate | 4–5 wks |
| 9 | [[Phase-9-Frontier-and-Career/00-Phase-9-Index\|Advanced Frontier + Career]] | Reasoning models, MoE, quantization, distillation, interpretability, paper reading, system-design interviews, portfolio, job hunt | ongoing |

## Per-phase deliverables (every phase has all of these)
Each phase folder contains:
1. **Index** – goals, topic map, weekly plan, checklist
2. **Concept notes** – beginner → advanced theory
3. **Exercises** – graded (🟢 easy / 🟡 medium / 🔴 hard)
4. **Q&A** – interview-style questions with hidden answers
5. **Resources & Practice Links** – courses, docs, books, problem sets
6. **Projects & Checkpoint** – build something + self-assessment

## Progress tracker
- [ ] Phase 1 — Software & Python Foundations
- [ ] Phase 2 — ML & Deep Learning Foundations
- [ ] Phase 3 — Transformers & LLM Internals
- [ ] Phase 4 — Building with LLM APIs
- [ ] Phase 5 — RAG & Knowledge Systems
- [ ] Phase 6 — Agents, Tools & MCP
- [ ] Phase 7 — Evals, Observability & LLMOps
- [ ] Phase 8 — Forward Deployed Engineering
- [ ] Phase 9 — Advanced Frontier + Career

## Detailed topic map (what each later phase will cover)

### Phase 2 — ML & Deep Learning
Linear/logistic regression · loss functions · gradient descent · overfitting/regularization · train/val/test · metrics (precision/recall/F1/AUC) · trees & ensembles (sklearn, XGBoost) · neural nets · backprop · optimizers (SGD/Adam/AdamW) · PyTorch tensors/autograd/training loops · CNN/RNN overview · embeddings intuition · math primer (linear algebra, calculus, probability, information theory: entropy, cross-entropy, KL)

### Phase 3 — Transformers & LLM Internals
Tokenization (BPE) · embeddings · self-attention/multi-head · positional encodings (RoPE) · transformer block (LayerNorm, residuals, MLP) · decoder-only architecture · pretraining & scaling laws · post-training (SFT, RLHF, DPO, GRPO/RLVR) · inference (KV cache, batching, speculative decoding) · sampling (temperature/top-p) · context windows & long-context · multimodal models · open vs closed models · hallucination · build nanoGPT

### Phase 4 — Building with LLM APIs
Anthropic/OpenAI/Gemini APIs · messages & roles · system prompts · prompt engineering patterns · structured outputs/JSON schema · function/tool calling · streaming · prompt caching · batch APIs · multimodal inputs · context engineering · token/cost/latency budgeting · rate limits/retries · model routing

### Phase 5 — RAG
Embeddings models · vector DBs (pgvector, Qdrant, Pinecone, Chroma…) · ANN (HNSW/IVF) · chunking strategies · metadata filtering · BM25 + hybrid search · reranking (cross-encoders) · query rewriting/HyDE · contextual retrieval · GraphRAG · agentic RAG · document parsing (PDF/tables/OCR) · RAG evals (faithfulness, context precision/recall)

### Phase 6 — Agents
ReAct/plan-and-execute · tool design · agent loops · memory (short/long-term) · multi-agent patterns · MCP (Model Context Protocol) servers/clients · Claude Agent SDK / OpenAI Agents SDK / LangGraph · computer use/browser agents · sandboxing/code execution · human-in-the-loop · guardrails · agent evals · failure modes

### Phase 7 — Evals, Observability, LLMOps
Eval datasets · code graders vs LLM-as-judge (bias, calibration) · regression testing · Braintrust/Langfuse/LangSmith/Promptfoo/Phoenix · tracing (OpenTelemetry) · CI/CD for prompts · serving (vLLM, TGI, Ollama) · fine-tuning (LoRA/QLoRA, TRL) · quantization · caching/latency · security (prompt injection, data exfiltration, OWASP LLM Top 10) · cost management · cloud (AWS Bedrock, Azure OpenAI, Vertex)

### Phase 8 — Forward Deployed Engineering
What FDEs do · discovery & requirements · scoping & SOWs · rapid prototyping · data integration (ERP/CRM/legacy, SSO, APIs, ETL) · enterprise security & compliance (SOC2, HIPAA, GDPR, VPC/on-prem) · change management · demos & exec comms · writing · measuring ROI · case studies · FDE interview loops

### Phase 9 — Advanced + Career
Reasoning models & test-time compute · MoE · quantization/distillation · RL for LLMs · mechanistic interpretability · synthetic data · long-context/memory research · reading papers · AI system design interviews · portfolio & GitHub · resume · networking · negotiation

## Rules of engagement for yourself
1. **Build, don't just read.** ≥50% of time in code.
2. **One project per phase** — commit to GitHub with a README (problem, architecture, evals, tradeoffs).
3. **Spaced repetition** — turn Q&A into flashcards (Obsidian Spaced Repetition plugin).
4. **Keep a `Learning Log`** — 5 lines/day: what I did, what confused me.
5. Check [[Phase-1-Foundations/00-Phase-1-Index|Phase 1]] to start.
