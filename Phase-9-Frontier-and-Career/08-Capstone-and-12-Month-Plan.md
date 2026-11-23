---
tags: [phase-9, capstone, plan]
---
# 08 — Capstone & 12-Month Plan

> The vault gives you the **map**. This note gives you the **route**: a capstone that integrates everything, a 12-month schedule with checkpoints, and a learning operating system so progress doesn't depend on motivation.

## 1. The integrated capstone: `enterprise-ai-teammate`
One flagship system that demonstrates the **whole stack** — your headline portfolio piece. Choose a domain you can access realistic (or synthetic) data for: support, claims, contracts, IT helpdesk, finance ops, healthcare admin (synthetic), logistics exceptions.
**Components**
1. **Software foundations (P1):** typed Python, tests, CI, Docker, FastAPI, Postgres; clean repo.
2. **Model understanding (P2–3):** a short design note on model/tokenizer/context/cost choices and why (token math, KV/cost estimates; maybe a small custom component like a reranker or tiny fine-tune).
3. **API craft (P4):** structured outputs, streaming, caching, batch, context engineering, cost meter.
4. **Knowledge (P5):** ingestion with permissions, hybrid retrieval + rerank, citations, abstention, golden set.
5. **Agents (P6):** tools via MCP behind a gateway, propose-then-approve actions, sandboxing, budgets, memory with poisoning defenses.
6. **Quality & ops (P7):** error-analysis-derived eval suite, calibrated judges (TPR/TNR), CI gate, tracing/dashboards, versioned prompts/config, canary/rollback, guardrails + red-team report, governance pack.
7. **Customer delivery (P8):** discovery report, scope/gates, ROI model, enterprise-readiness pack, runbook, handoff, pilot readout, case study.
8. **Frontier depth (P9):** one deep-dive extension — pick one: *test-time compute study* (best-of-N/verifier curves for a task in your system), *quantization/serving study* (self-host an open model path with measured quality/cost), *RLVR mini-experiment* for a verifiable subtask, or *interpretability probe/SAE experiment* on a small model related to your domain.
**Deliverables:** public repo + demo video (5–8 min) + 10-page technical write-up + 2-page executive summary + eval/security/governance artifacts + a blog post of lessons. **Acceptance criteria:** reproducible setup (`make demo`), eval suite with CIs passing in CI, security threat model with tested mitigations, cost/latency table, honest limitations.

## 2. 12-month plan (adapt to your starting point)
Assumes ~10–15 hrs/week alongside work/study; scale pace up/down. Each quarter ends with a **review gate**.
### Q1 (months 1–3): Foundations → ML → LLM internals
- **Month 1:** Phase 1 (Python, DSA basics, Git/CLI, HTTP/SQL, Docker/tests). *Output:* 3 small repos, capstone-API skeleton.
- **Month 2:** Phase 2 (math, classical ML, backprop, PyTorch). *Output:* micrograd + MLP/CNN; ML eval project with CIs.
- **Month 3:** Phase 3 (tokenization, transformer, nanoGPT, pretraining/inference math). *Output:* tinyformer + llm-calc.
**Gate 1:** can write attention from memory, solve LC-easy/medium, explain backprop/KV cache, ship a tested FastAPI app.
### Q2 (months 4–6): Build with LLMs
- **Month 4:** Phase 4 (APIs, prompts, structured outputs, tools, streaming, caching, cost). *Output:* chatkit + extraction pipeline.
- **Month 5:** Phase 5 (RAG). *Output:* hybrid-rag with golden set and ablation table.
- **Month 6:** Phase 6 (agents/MCP). *Output:* agent runtime + MCP server + injection red-team.
**Gate 2:** portfolio has 2 flagship projects with evals; can design RAG/agent systems under constraints; start networking + public writing.
### Q3 (months 7–9): Quality, operations, customer craft
- **Month 7:** Phase 7 (evals, judges, observability, release). *Output:* evalkit + trace-lab.
- **Month 8:** Phase 7 continued (fine-tuning, serving, guardrails, governance). *Output:* tune-or-not memo + serve-bench.
- **Month 9:** Phase 8 (FDE craft). *Output:* playbook + pilot-in-two-weeks + case studies.
**Gate 3:** capstone core built; mock interviews begin; resume/LinkedIn/GitHub polished; 10 informational chats done.
### Q4 (months 10–12): Frontier depth + job search
- **Month 10:** Phase 9 frontier modules + one paper reproduction/extension; capstone finish + write-up.
- **Month 11:** Job search sprint (applications, referrals, mocks); publish capstone + blog.
- **Month 12:** Interviews/offers; decision framework; plan first 90 days; retrospective and next-year plan.
**Gate 4:** offers or clear funnel diagnosis and adjustments; capstone public; learning cadence sustained.
**If you already work as an engineer:** compress Q1 (skip known parts), spend more on Q2–Q3, and start the job search earlier. **If you're starting from zero programming:** expect Phase 1 to take 3–4 months; extend the schedule rather than skipping foundations.

## 3. Weekly operating rhythm
| Block | Hours | Activity |
|---|---|---|
| Learn | 3–4 | read/watch one concept note + one primary source |
| Build | 5–7 | exercises + project work (code first) |
| Evaluate | 1–2 | write/extend evals, error analysis, transcript/trace reading |
| Write | 1 | lab notebook, write-up, or blog |
| Review | 1 | flashcards (Q&A notes), weekly review, plan next week |
| Network | 1 | one outreach/informational chat/community interaction/PR |
**Rules:** *build before you read more*; *ship something every week*; *measure everything you claim*; *take notes in the vault*.

## 4. Review checkpoints (self-assessment loop)
Every quarter: (1) re-take the self-assessment tables across phases; (2) run a **mock interview** (AI engineer + FDE); (3) review portfolio against "hireable project" criteria; (4) list the 3 biggest gaps and plan fixes; (5) update goals; (6) get external feedback from a mentor/peer.
**Leading indicators of progress:** projects with evals and CIs, public artifacts, mock-interview scores, number of production-style concepts you can explain *and* implement, comfort reading papers/docs, quality of outreach conversations.

## 5. Risks & mitigations
| Risk | Mitigation |
|---|---|
| Tutorial hell | project-first; enforce "build from scratch" exercises; weekly shipped artifact |
| Scope explosion | one capstone; time-box; cut features, not evals |
| Imposter syndrome | track wins (brag doc); teach/write; peer community |
| Burnout | sustainable cadence; rest weeks each quarter |
| Tool/model churn | invest in portable assets (evals, patterns); verify docs before building |
| Isolation | study group, mentor, meetups, open-source |
| Market uncertainty | diversify target roles/companies; keep earning/learning options; adjacent-role routes |
| Over-reliance on AI assistants | use for explanation/review after attempting; periodically do "no-AI" practice |

## 6. Obsidian setup tips for this vault
- **Daily note + weekly review template** (Core plugin: Templates; Community: Periodic Notes).
- **Spaced repetition:** `Q::A` flashcards extracted from Q&A callouts (Spaced Repetition plugin); schedule short daily reviews.
- **Dataview** to query read-log/experiment-log (e.g., table of papers by status/tag).
- **Graph view** to see connections; keep MOCs (maps of content): `00-Roadmap`, per-phase indexes, a "Portfolio MOC," a "Interview MOC."
- **Link liberally** between concepts, projects, and papers; tag notes (`#evals`, `#rag`, `#agents`).
- **Version the vault** with Git (private repo) for history and backups; exclude secrets.
- **Maintain "state-of" notes** with a date, sources, and confidence level; revisit quarterly.

## 7. After the roadmap: staying excellent
Pick a **specialty** (evals/LLMOps, agents & security, RAG/knowledge systems, inference/efficiency, FDE/solutions, safety/interpretability, AI product) and go 10× deeper over 2–3 years while keeping breadth. Teach what you know. Mentor newcomers. Revisit this vault yearly — update tool/provider specifics, refresh projects, and add your own field notes.

Next → [[09-Exercises]]
