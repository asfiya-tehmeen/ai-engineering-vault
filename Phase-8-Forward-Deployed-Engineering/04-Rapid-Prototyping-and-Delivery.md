---
tags: [phase-8, prototyping, delivery]
---
# 04 — Rapid Prototyping & Delivery

> The FDE superpower: **a working thing on the customer's real data, in days, that makes the problem — and the solution — concrete.** Speed creates trust and learning; discipline (evals, security, structure) keeps speed from creating a mess.

## 1. The delivery loop
```
discover → thin slice → demo on REAL data → feedback → measure → iterate → harden → pilot → production
   ▲__________________ weekly (or faster) cycles; every cycle ends with something users can react to ____________|
```
- **Time-to-first-demo target:** 3–7 days after data access. A demo with *their* documents/tickets beats any slide deck.
- **Demo-driven development:** define the next demo ("by Thursday, show 20 real invoices extracted with accuracy and failure reasons"), then build backward.
- **Walking skeleton first:** end-to-end pipe (input → processing → output → measurement) with trivial logic; then deepen each stage by evidence.

## 2. Choose the thin vertical slice
Pick the slice that **maximizes learning per day**: the riskiest assumption (data quality? accuracy? integration? user trust?) on the most common path. Examples: "classify + extract the top 2 doc types," "answer the top 20 policy questions with citations," "draft replies for 2 ticket categories." Defer: admin UIs, multi-tenancy, perfect error handling, edge integrations — **but not** evals, security basics, and logging.

## 3. Prototyping stack (pragmatic defaults)
| Need | Fast choices |
|---|---|
| Language | Python (+ TypeScript for UI) |
| LLM access | provider SDK + thin wrapper ([[../Phase-4-Building-with-LLM-APIs/01-API-Fundamentals]]); model via config |
| Structured output | Pydantic + schema-constrained outputs |
| Retrieval | pgvector/Chroma/LanceDB + hybrid; start NumPy-simple ([[../Phase-5-RAG-Knowledge-Systems/08-Build-RAG-From-Scratch]]) |
| Agents/tools | raw loop or a lightweight SDK ([[../Phase-6-Agents-Tools-MCP/09-Build-an-Agent-From-Scratch]]) |
| UI | Streamlit/Gradio/Next.js; **embed in the user's existing tool** when possible (Slack/Teams/browser extension/side panel) |
| Backend | FastAPI + Postgres + a queue for async jobs |
| Evals | pytest + JSONL golden set + calibrated judge ([[../Phase-7-Evals-Observability-LLMOps/09-Build-an-Eval-and-LLMOps-Pipeline]]) |
| Tracing | Langfuse/Phoenix/OpenTelemetry |
| Deploy | container → customer cloud/VPC (or your sandbox with redacted data first) |
| Notebooks | for exploration only; move logic into tested modules |
Prefer **boring, debuggable tech the customer's team can maintain** (their language/cloud/standards) when handoff is the goal.

## 4. Working on messy data (reality)
- **Get real samples on day 1–3** (even 30–50). Synthetic data is a fallback, not a substitute.
- **Look at the data yourself** — read 50 records; note formats, languages, scan quality, boilerplate, duplicates, edge cases.
- **Build an ingestion pipeline with quality reports** (fill rates, error counts, sample galleries).
- **Parsing is half the project** for documents ([[../Phase-5-RAG-Knowledge-Systems/04-Ingestion-Parsing-and-Chunking]]).
- **Respect sensitivity:** work in approved environments; minimize copies; redaction; no uploading customer data to unapproved services — **ask first**.
- **Create a golden set early** with SMEs (even 30 cases); evolve it.
- **Version everything** (data snapshots, prompts, configs) — customers will ask "what changed?"

## 5. Eval-first prototyping (stay honest while moving fast)
1. Day 1–3: collect ≥30 real cases + expected behavior (with SME). 
2. Build the simplest pipeline; run; **read failures**; tag categories. 
3. Each iteration: change one thing; **re-run the eval**; log metric deltas, cost, latency. 
4. Show customers **metrics + failure examples**, not just best-case demos — builds trust and calibrates expectations. 
5. Maintain a **"known limitations" list** and share it proactively.
6. Keep a **decision log** (what we tried, results, why we chose X).

## 6. Demo craft
- **Use their data and their words**; start from the user's problem statement, not your architecture.
- **Script the happy path + one realistic failure path** (show graceful handling and human escalation). Rehearse; have a recorded backup; test network/VPN/SSO beforehand.
- **Show the metric:** "On 120 held-out claims: 82% field accuracy, 61% straight-through; here are the top 3 failure types and the plan."
- **Make it interactive:** let users try their own queries (in a controlled environment) — adoption starts here.
- **End with a clear ask:** data access, SME hours, security reviewer intro, next demo date.
- **Capture reactions & requests** live; turn into backlog items with priority.
- **Avoid:** over-polished fake UIs masking weak backends, cherry-picked examples, promising features on stage.

## 7. Code quality: throwaway vs durable
Classify code explicitly:
| Type | Standard |
|---|---|
| **Spike/throwaway** (investigations) | fast, in notebooks/scripts; delete or archive; never ship |
| **Prototype** (customer-visible, pilot) | typed, linted, basic tests, config-driven, logged, secrets handled; modular enough to evolve |
| **Production** | full tests, CI/CD, observability, security review, runbook, ownership |
**Rule:** prototypes that survive become production — so **start with the production-grade parts that are expensive to retrofit**: secrets management, auth, data boundaries, logging/tracing, evals, idempotency, configuration, dependency pinning.
**Repo hygiene:** `README` (how to run), `Makefile`/scripts, `.env.example`, pinned deps, pre-commit, CI running lint/tests/evals, ADRs for key decisions, `docs/` with architecture + runbook.

## 8. Working agreements & rhythm
- **Daily:** short build blocks; quick async updates in the shared channel when blocked.
- **Weekly:** written update (progress vs plan, metrics, risks, asks, next week) + demo/office hours.
- **Biweekly/at gates:** steering readout with sponsor.
- **Feedback loop:** 3–5 pilot users with a direct channel; collect issues in a visible tracker; triage twice weekly.
- **Time protection:** block deep-work time; batch customer meetings; avoid being the "always-on helpdesk."
- **Pairing with customer engineers:** transfers knowledge, builds ownership, exposes constraints early.

## 9. Productizing patterns (the FDE→product flywheel)
After the 2nd–3rd similar engagement, extract reusable assets:
- **Connectors/ingestion modules** (SharePoint, Salesforce, ServiceNow), **eval harnesses**, **prompt/tool templates**, **deployment scripts**, **guardrail packs**, **dashboards**, **runbook templates**.
- Decide **where each asset lives:** internal FDE toolkit → shared library → product feature (with product/eng).
- **Feedback to product:** write actionable requests (problem, frequency, workaround, impact, proposed solution, evidence from traces) — see [[06-Stakeholders-Communication-and-Change]].
- **Beware the services trap:** bespoke work that never converges. Track % reuse, time-to-deploy per engagement, and push repeat needs into product.

## 10. Common prototype → pilot pitfalls
- Prototype works on cherry-picked data; **no golden set** → surprise at scale.
- Hidden manual steps (you're the pipeline) → not repeatable.
- No auth/logging → blocked at security review.
- Hard-coded config/paths/keys → breaks in customer environment.
- Latency/cost never measured → production sticker shock.
- UI separate from the user's workflow → low adoption.
- Single-threaded knowledge (only you can run it) → bus factor.
- Skipping error handling for LLM failures (timeouts, refusals, malformed JSON) → brittle pilot.
- Overfitting prompts to demo cases.

## 11. Pilot execution checklist
- [ ] Success metrics + baseline + golden set agreed
- [ ] Access to data/systems working in the pilot environment
- [ ] Auth/SSO and permissions model implemented (or approved interim)
- [ ] Logging/tracing, cost tracking, error handling in place
- [ ] Human-in-the-loop and escalation paths designed
- [ ] User training/onboarding materials; champions identified
- [ ] Feedback capture (in-tool thumbs/comments) + triage process
- [ ] Weekly metrics report automated (quality, adoption, cost, latency)
- [ ] Risk register reviewed weekly; security/compliance sign-offs tracked
- [ ] Rollback/disable switch and support contacts documented
- [ ] Pilot readout template ready (results vs targets, learnings, recommendation)

## 12. The pilot readout (structure)
1. **Executive summary:** outcome vs targets; recommendation (go/iterate/stop). 
2. **What we built** (1 diagram) and who used it. 
3. **Results:** primary + guardrail metrics vs baseline, with CIs; adoption; cost per task. 
4. **User voice:** quotes, satisfaction, time saved stories. 
5. **Failure analysis:** top failure modes, severity, fixes done/remaining. 
6. **Risks & compliance status.** 
7. **Production plan:** scope, timeline, team, cost, operating model. 
8. **Asks & decisions needed.** 
Honesty about weaknesses is credibility.

## Self-check
1. Define "walking skeleton" and "thin vertical slice." 2. What would you build in 5 days after getting 50 real documents? 3. Classify code into spike/prototype/production and list what to harden early. 4. Script a 10-minute demo including a failure path. 5. Which assets do you productize after two engagements? 6. Structure a pilot readout.

Next → [[05-Enterprise-Integration-and-Data]]
