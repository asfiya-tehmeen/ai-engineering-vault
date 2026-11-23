---
tags: [phase-8, projects, checkpoint]
---
# 13 — Projects & Phase Checkpoint

> For FDE roles, **artifacts that show customer-facing judgment** matter as much as code. Each project below produces both: working software and the *documents a real engagement produces*. Publish sanitized versions (synthetic data; no confidential info; permission for anything real) on GitHub or a personal site.

## Project 1 — `fde-playbook`: your reusable engagement toolkit *(Notes 02–07)*
A repo of templates and worked examples: **discovery interview guide** (+ question bank), stakeholder map, process-map template, use-case scorecard, **one-page scope**, go/iterate/stop gate sheet, **ROI model spreadsheet** (with sensitivity tab), risk register, RACI/working agreement, weekly update, steering-readout deck outline, demo script, **production readiness review (PRR)**, runbook, handoff checklist, retro template, product-feedback memo. Include **two filled-in examples** (e.g., support copilot; invoice extraction).
**Quality bar:** a stranger could run an engagement from it.

## Project 2 — `mock-enterprise`: enterprise integration sandbox *(Note 05)*
`docker compose` environment simulating a customer's stack: **Keycloak** (OIDC/SAML + SCIM), **Postgres** ERP-like DB, **fake CRM API** (pagination, rate limits, errors, webhooks), **fake ITSM** (tickets + webhooks), **SFTP** batch drops, **legacy SOAP** service, **egress proxy** with allow-list/TLS interception, **MinIO**, **Vault** for secrets. Provide seed data and a README of "integration challenges to solve."
Then build a reference app: SSO login + role mapping, SCIM sync, CRM connector with retries/idempotency, webhook receiver, SFTP ingestion with DLQ/replay, permission-aware RAG with ACL sync, MCP server behind a gateway, and contract tests.
**Deliverable:** sandbox repo + integration reference implementation + a 2-page "lessons from integrating" write-up.

## Project 3 — `pilot-in-two-weeks`: end-to-end thin-slice pilot *(Notes 02–04, 06)*
Pick a realistic workflow (real person with permission, or a public dataset + persona). In **two weeks**: discovery interviews → baseline → scope with gates → prototype with golden set and calibrated judge → weekly updates → mid-point demo → pilot readout with numbers.
**Deliverables:** discovery report, scope doc, ROI model, weekly updates (×2), demo recording, eval report with CIs, pilot readout deck, retrospective. **Show a failure and how you handled it.**

## Project 4 — `enterprise-readiness-pack`: security, governance & operations dossier *(Notes 05, 07; Phase 7 note 08)*
For one of your AI apps (Phase 5/6/7 capstone): **security questionnaire answers (≥50 questions)**, architecture & data-flow diagrams with trust boundaries, threat model + red-team summary, system card, DPIA-style assessment, SLOs and monitoring plan, runbook, incident playbook, vendor/provider assessment, and a **customer-facing FAQ** (data retention, training use, permissions, auditability, outages, model changes).
**Stretch:** an "air-gapped deployment guide" with offline bundle and upgrade/rollback procedure.

## Project 5 — `case-study-portfolio`: three written case studies *(Notes 08, 09)*
Three sanitized case studies from your projects (or Project 3): *Situation → Discovery → Design → Pitfalls → Metrics → Outcome → What I'd do differently*, each 2–3 pages with a diagram and a metrics table. Include **one failure/post-mortem**. Add a 10-minute talk version of the best one (recorded).
**Use:** interview deep-dives and your personal site.

## Project 6 (Capstone) — Full simulated engagement (6–8 weeks) *(everything)*
**Setup:** choose a domain (insurance claims, bank KYC, hospital documentation with synthetic data, logistics exceptions, legal contract review, IT helpdesk). Recruit a **partner** (or run LLM personas with detailed hidden constraints) to play: sponsor, champion, end users, IT, security, SMEs. Define a fictional company with realistic systems (use Project 2's sandbox).
**Phases & deliverables**
1. **Discovery (wk 1):** interviews, process map, baseline, data audit, stakeholder map → discovery report + recommendation.
2. **Scoping (wk 2):** one-page scope, success metrics/gates, ROI model, risk register, security intake.
3. **Prototype (wk 2–4):** thin slice on realistic data; golden set (≥100 cases) + calibrated judge; weekly updates; 2 demos with stakeholder feedback.
4. **Integration & security (wk 3–5):** SSO, permissions-aware data access, guardrails, red-team, questionnaire, evidence pack.
5. **Pilot (wk 5–7):** simulated pilot users; adoption/feedback loop; metrics vs targets; scope-creep and bad-news scenarios injected by your partner (e.g., an access delay, an accuracy miss, a security finding).
6. **Production & handoff (wk 7–8):** PRR, runbook, training, support model, handoff package, 30/60/90 plan, value-realization plan, retro, product-feedback memo.
**Final presentations:** a **15-minute executive readout** (recorded) and a **30-minute technical walkthrough** (recorded).
**Evaluation rubric (self + partner):** discovery quality · scoping clarity · technical quality & evals · integration/security rigor · communication (written/verbal) · handling of surprises · production/handoff readiness · honesty about limitations · artifacts quality.
This capstone is your strongest demonstration of FDE readiness.

## Phase 8 Self-assessment (0–3; ≥2 everywhere before interviewing)
| Skill | 0–3 |
|---|---|
| Explain FDE role, engagement models, and differences from adjacent roles | |
| Run a structured, non-leading discovery interview and observe real work | |
| Decompose an ambiguous problem aloud with structure and a first slice | |
| Assess AI suitability; score use cases (value/feasibility/risk/adoption) | |
| Produce a baseline and success metrics with guardrails | |
| Write a one-page scope with gates and non-goals | |
| Estimate under uncertainty; identify calendar-bound risks | |
| Build an ROI model with sensitivity | |
| Control scope with change requests and options | |
| Ship a thin-slice prototype with golden set and demo | |
| Prepare and deliver an effective demo (incl. failure path) | |
| Integrate with enterprise stacks (SSO, SCIM, APIs, files, webhooks, legacy) | |
| Navigate network constraints, proxies, VPC/on-prem/air-gapped options | |
| Complete a security questionnaire and evidence pack | |
| Explain AI accuracy/limits to executives and end users | |
| Write crisp weekly updates, one-pagers, and post-mortems | |
| Deliver bad news and handle difficult stakeholders | |
| Plan and measure adoption/change management | |
| Run a production readiness review and design handoff | |
| Debug unfamiliar code; learn new docs quickly | |
| Tell strong STAR stories and present project deep-dives | |

## Final mock loop (≈4 hours; do with a mentor/peer)
1. **Recruiter/behavioral (30 min):** motivation, role understanding, 2 STAR stories.
2. **Decomposition/case (45 min):** vague customer problem → clarifying questions → decomposition → plan/metrics/risks.
3. **Coding/applied-AI (45 min):** practical Python/SQL + a RAG/eval/agent snippet.
4. **Debugging/re-engineering (45 min):** fix subtle bugs in unfamiliar code while narrating.
5. **Solution design (45 min):** enterprise AI system with permissions, integration, evals, rollout.
6. **Customer role-play (30 min):** skeptical exec + security gatekeeper + scope creep + bad news.
7. **Project deep dive (30 min):** best project with metrics; hostile questions.
**Debrief:** structured feedback on each dimension; update your plan; repeat in 1–2 weeks.

## What you've unlocked
You now have the full stack: **software + ML + LLM internals + APIs + RAG + agents + evals/LLMOps + customer-facing delivery.** What remains is *frontier depth* — reasoning models, MoE and quantization, distillation, RL for LLMs, interpretability, paper reading — plus the **job hunt** itself (portfolio, resume, networking, negotiation).

⏸️ **Pause point.** Tell me when you're ready for **Phase 9 — Advanced Frontier + Career** (I'll research current frontier trends and open problems, notable recent papers and reading lists, and the 2026 AI hiring landscape first), which completes the roadmap.

Back to [[00-Phase-8-Index]] · [[../00-Roadmap]]
