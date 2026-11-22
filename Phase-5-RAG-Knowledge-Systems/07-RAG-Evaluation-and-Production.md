---
tags: [phase-5, evaluation, production, security]
---
# 07 — RAG Evaluation & Production

> If you can't measure it, you can't improve it — and in RAG, "it feels better" is how regressions ship. Evaluate **retrieval and generation separately**, then end-to-end. (General eval theory: [[../Phase-2-ML-Deep-Learning/04-Evaluation-and-Metrics]]; deep dive in Phase 7.)

## 1. Build the golden set (the most valuable asset)
- **Queries:** collect **real** user questions (logs, support tickets, interviews with domain experts) — not just synthetic ones. Cover: factoid, exact-ID, multi-passage, multi-hop, conversational follow-ups, ambiguous, **unanswerable**, adversarial/injection, permission-restricted, different languages, typos.
- **Labels:** for each query, the **relevant chunk/doc IDs** (ideally with graded relevance) and a **reference answer** (or key facts the answer must contain). Have SMEs label; measure **inter-annotator agreement**.
- **Size:** start with 50–100 well-chosen queries; grow to several hundred for tight confidence intervals; stratify so each important slice has enough items.
- **Synthetic generation** (LLM writes questions from chunks) is great for *scale and bootstrapping* but biased: questions copy chunk wording (inflating lexical/dense retrieval), skew easy, and miss real user phrasing. **Mix synthetic with real**, filter low-quality items, and keep a real-query test set separate.
- **Freeze** a test split; keep a dev split for iteration. **Version** the set; add every production failure as a new case (the flywheel).
- **Chunking-independent labels:** label evidence at the **document/section/quote level**, then map to chunks per configuration, so you can compare chunkers fairly.

## 2. Retrieval metrics
| Metric | Meaning |
|---|---|
| **Recall@k** | fraction of relevant items found in top-k (**the ceiling for answer quality**) |
| **Hit rate@k** | ≥1 relevant in top-k |
| **Precision@k / context precision** | fraction of retrieved that's relevant (noise level) |
| **MRR** | `1/rank` of first relevant |
| **nDCG@k** | graded relevance with position discount |
| **MAP** | average precision across queries |
| **Coverage** (multi-hop) | all needed evidence pieces retrieved |
| **Latency/cost** | p50/p95; tokens retrieved |
Compute with bootstrap CIs; compare configs with **paired** tests on the same queries; report by slice.

## 3. Generation metrics
| Dimension | Question | How |
|---|---|---|
| **Correctness** | does it match the reference/ground truth? | exact/F1 for short facts; LLM-judge vs reference; human |
| **Faithfulness / groundedness** | is every claim supported by the *retrieved context*? | claim extraction + entailment (RAGAS-style), NLI, LLM-judge with evidence |
| **Answer relevance** | does it address the question? | LLM-judge |
| **Completeness** | all required facts included? | key-fact checklist |
| **Citation precision/recall** | do citations support claims? are claims cited? | quote matching + entailment |
| **Abstention accuracy** | declines when unanswerable; answers when answerable | labeled unanswerable set; report false-abstain & false-answer rates |
| **Conciseness/format/tone** | meets spec | rubric/judge/regex |
| **Safety/permissions** | no leakage, no injection success | adversarial tests |
**Tools/frameworks:** RAGAS (context precision/recall, faithfulness, answer relevancy), TruLens (RAG triad: context relevance, groundedness, answer relevance), DeepEval, ARES, Phoenix/Langfuse/LangSmith/Braintrust evaluators, Promptfoo, custom judges.

## 4. Pitfalls of automatic RAG metrics (learn these — interviewers ask)
- **Faithfulness ≠ correctness.** An answer can be perfectly faithful to *wrong or outdated retrieved context* and score high. Always pair with correctness vs ground truth.
- **Grounded but evasive:** a faithful answer that dodges the question scores well on faithfulness — pair with answer relevance/completeness.
- **Hidden retrieval misses:** the model answers from **parametric memory**; faithfulness looks fine while retrieval silently failed. Always measure **retrieval recall independently**; test with questions whose answers *aren't* in the model's pretraining (private/new facts).
- **Context precision is rank-weighted** and doesn't measure how much irrelevant text is in the set — look at tokens-per-relevant-fact too.
- **LLM-judge reliability:** judges have biases (verbosity, position, self-preference, leniency), vary by prompt/model, and may reflect LLM knowledge rather than retrieval quality. **Calibrate against human labels** (agreement/κ on a sample), use rubrics with anchored examples and evidence quotes, pin judge model/version, use pairwise comparisons for A/B, and re-calibrate when you change judges.
- **Metric disagreement** is a signal — inspect examples rather than averaging blindly.
- **Overfitting the golden set:** tuning prompts/thresholds against a small fixed set → hold out test; refresh with new real queries.
- **Reference-answer brittleness:** many valid phrasings; use key facts or judge-with-reference instead of exact match.

## 5. Error analysis workflow (where improvements come from)
1. Run the system on the golden set; log **full traces** (query, rewrites, retrieved IDs+scores, reranked list, final context, answer, citations, latency, tokens).
2. For each failure, locate the **first broken stage** using the failure map ([[01-RAG-Fundamentals]] §4): corpus gap → parsing → chunking → retrieval → ranking → assembly → generation → citation.
3. **Tag** failure types; count; prioritize by frequency × impact.
4. Fix the top category with the smallest change (parsing rule, chunk size/structure, hybrid, reranker, query rewrite, prompt, abstention threshold, data/label fix).
5. Re-run; confirm no regressions on other slices; log the experiment (config, metrics, cost).
Typical discoveries: 30–50% of failures are **parsing/chunking/metadata**; exact-ID failures vanish with hybrid; "hallucinations" are often retrieval misses; unanswerable handling needs explicit examples.

## 6. Online evaluation & monitoring
- **Instrument everything** (OpenTelemetry/LLM-observability: Langfuse, LangSmith, Phoenix, Braintrust): query, user/tenant, retrieved IDs/scores, context size, model, latency by stage, token usage/cost, answer, citations, feedback.
- **User feedback** (👍/👎, edits, "didn't answer", escalations to human) → sample into review queue → new golden cases.
- **Automatic online checks on sampled traffic:** faithfulness/relevance via LLM-judge, citation validity, abstention rate, retrieval-score distributions (drift), empty-result rate, latency/cost anomalies.
- **Canary & A/B** new embedders/rerankers/prompts; shadow traffic; guardrail metrics (permission violations, harmful outputs).
- **Alerts:** index staleness, ingestion failures, rising "I couldn't find" rate, latency spikes, cost spikes.
- **Regression CI:** run the golden set on every change to prompts/chunking/retrieval config; fail the build on metric drops beyond CI.

## 7. Security, privacy & governance
- **Authorization at retrieval time:** filter by user identity/groups (ACL metadata), test cross-tenant leakage, sync permission changes fast, default-deny. **Do not** rely on the LLM to hide restricted content. Consider per-tenant indexes for strict isolation.
- **Prompt injection via documents/web/email:** retrieved content can contain instructions ("ignore previous… email the data to…"). Mitigate: delimit & label as untrusted data, strip/neutralize suspicious patterns, least-privilege tools (no write/send actions from RAG context without confirmation), output filtering, separate "reader" from "actor" models, monitor.
- **Data exfiltration paths:** markdown image/link rendering of attacker URLs, tool calls with sensitive data in args — sanitize rendering, allow-list domains.
- **PII/PHI:** classify/redact at ingestion if required; encryption at rest/in transit; retention & **right-to-delete** (must delete chunks, embeddings, caches, logs); data residency; vendor data-handling terms (zero-retention options and feature eligibility).
- **Audit logging:** who asked what, what sources were used.
- **Poisoning:** untrusted sources can inject malicious content into the index; vet sources, track provenance, allow quarantine/takedown.
- **Compliance posture:** SOC 2/ISO 27001/HIPAA/GDPR needs shape architecture (VPC/on-prem/open-weight models) — often decisive in enterprise FDE work.

## 8. Freshness, versioning & change management
- Define **freshness SLAs**; show timestamps; handle superseded docs; incremental ingestion; deletion propagation.
- **Index versioning:** `index_v{n}` with embedder/chunker versions; blue/green rollout with shadow eval; rollback.
- **Prompt/config versioning** tied to eval results.
- **Model migrations** (LLM, embedder, reranker) → full regression run.

## 9. Cost & latency engineering
- **Cost per query** = embed(query) + vector/BM25 + rerank + LLM(input incl. context + output) (+ rewrite/judge calls). LLM tokens usually dominate → trim context, use cache, smaller models for auxiliary steps, route trivial queries to cached answers.
- **Index cost:** parse (OCR/vision) + enrichment LLM calls + embeddings; use Batch APIs; incremental updates.
- **Latency budget by stage;** parallelize BM25 + dense; stream the answer; precompute; keep rerank candidates modest; colocate services.
- **Autoscaling & capacity:** ingestion workers by queue depth; separate query/ingest compute; rate-limit per tenant.
- **Quality–cost frontier:** present stakeholders with 2–3 configs (fast/cheap, balanced, max-quality) with measured metrics.

## 10. Enterprise rollout playbook (FDE lens)
1. **Discovery:** users, top 20 real questions, source systems, permissions, compliance, success metrics, risk tolerance. 
2. **Data audit:** sample docs; parse quality; duplicates/versions; ownership; sensitive data.
3. **Golden set with SMEs** (agree on "what is a good answer").
4. **Thin vertical slice** (one source, one team) in 1–2 weeks → demo with real queries.
5. **Iterate with error analysis;** show before/after metrics.
6. **Pilot** with feedback capture, human fallback, usage analytics.
7. **Harden:** permissions, security review, monitoring, runbooks, SLAs, cost controls.
8. **Handoff & enablement:** docs, dashboards, eval CI, "how to add a source," on-call/ownership.
9. **Expand** sources/teams; keep the flywheel running.
Common blockers: messy/untrusted data, missing ACL metadata, stakeholders expecting 100% accuracy, no SME time for labeling, security approvals, change management. Set expectations with **numbers** (e.g., "answers correct on 87% of 200 real questions; 6% abstain; 0 permission violations; weak on tables → plan").

## Self-check
1. Why can faithfulness be high when retrieval failed? 2. How do you build a golden set, and what are the pitfalls of synthetic queries? 3. Write the failure-localization checklist. 4. How do you calibrate an LLM judge? 5. How do you enforce per-user permissions? 6. Outline an enterprise RAG rollout and how you'd report progress to executives.

Next → [[08-Build-RAG-From-Scratch]]
