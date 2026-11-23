---
tags: [phase-8, exercises]
---
# 10 — Exercises (Phase 8)

🟢 easy · 🟡 medium · 🔴 hard. **FDE skills are learned by doing.** Wherever possible, practice with **real people**: a local small business, nonprofit, school, or a friend's workplace (with permission, no sensitive data). When that's impossible, **role-play with a peer or an LLM playing the customer** (give it a persona, hidden constraints, and instructions to be realistic and a bit difficult). Record yourself; review. Keep all artifacts in `phase8-exercises/` — they become your portfolio (sanitized).

## A. Role & landscape (Note 01)
- [ ] 🟢 A1. Read 10 real FDE-style job postings (any company). Tabulate: responsibilities, required skills, travel %, seniority, engagement model, red flags. Write a 1-page "what this role actually is" synthesis.
- [ ] 🟢 A2. Write a one-sentence contrast of FDE vs SWE vs SA vs SE vs consultant vs TAM, then test it against 3 postings with confusing titles.
- [ ] 🟡 A3. Fill the FDE skill matrix with honest self-ratings (0–3) + evidence + a 90-day plan for your two weakest areas.
- [ ] 🟡 A4. Interview (informational chat) 2 people in FDE/solutions/customer-engineering roles; write up their day-to-day, biggest surprises, and advice.
- [ ] 🔴 A5. Write a 2-page memo: "Why the FDE model exists for enterprise AI, its risks (services trap), and how a company should productize learnings." Cite sources and mark opinion vs evidence.

## B. Discovery & problem framing (Note 02)
- [ ] 🟢 B1. Rewrite 10 leading/hypothetical interview questions into past-behavior, non-leading ones.
- [ ] 🟡 B2. **Real discovery:** run two 30–45-minute discovery interviews with someone who has a repetitive document/communication-heavy workflow (small business, nonprofit, teacher, clinic admin). Produce: notes with verbatim quotes, process map, pain points with time/volume estimates.
- [ ] 🟡 B3. **Shadowing/observation exercise:** observe someone do a real task for 60+ minutes (or watch a screen recording); list steps, tools, interruptions, errors, workarounds.
- [ ] 🟡 B4. Build a **stakeholder map** for a fictional bank KYC project (who's sponsor, champion, user, gatekeeper, skeptic, security, finance) and a plan to engage each.
- [ ] 🟡 B5. **Value/feasibility/risk/adoption scorecard** for 6 candidate use cases at a fictional insurer; pick #1 and justify; identify the riskiest assumption and the cheapest test.
- [ ] 🟡 B6. **Baseline measurement:** time 30 real task instances (or use a public dataset such as support tickets) to compute a baseline distribution (median, p90, error rate); document method and caveats.
- [ ] 🔴 B7. Write a **discovery report** (3–5 pages): problem statement, current-state process map, data/systems inventory, constraints, options, recommendation, risks/assumptions with first tests.
- [ ] 🔴 B8. Do 10 **decomposition drills** (see Note 09 prompts) with a timer; record; self-grade on clarify/assume/structure/slice/metrics/human factors.
- [ ] 🔴 B9. "Is AI the right tool?" exercise: for 10 proposed AI projects, classify as {LLM, ML, rules/SQL, process change, don't build} with justification and one fact you'd need to confirm.

## C. Scoping & commercials (Note 03)
- [ ] 🟢 C1. Fill the one-page scope template for a "contract-review pilot" and a "support copilot pilot."
- [ ] 🟡 C2. Define go/iterate/stop gates with numeric thresholds, evaluation owners, and what happens if missed.
- [ ] 🟡 C3. Estimate a pilot using task decomposition with best/likely/worst ranges; identify calendar-bound items and schedule them first.
- [ ] 🟡 C4. Build an **ROI model** (spreadsheet) with inputs (volume, handle time, % automatable, review rate, adoption ramp, inference cost, build/run cost), conservative/base/optimistic cases, and a sensitivity table; list assumptions to validate.
- [ ] 🟡 C5. Create a **RACI** and **working agreement** for a pilot with a customer PM, SMEs, IT/security, and your product team.
- [ ] 🟡 C6. Write a **risk register** (≥10 risks) for a RAG pilot with early signals, mitigations, owners.
- [ ] 🔴 C7. **Scope-creep role-play:** the sponsor asks for 3 extra features mid-pilot. Respond with a change-request analysis and two options with timeline impact (record the conversation; get feedback).
- [ ] 🔴 C8. Compare pricing models (usage vs seat vs outcome) for an AI support product; for each, discuss incentive effects on adoption, cost control, and technical architecture choices.
- [ ] 🔴 C9. Draft a mock **SOW excerpt**: scope, deliverables, acceptance criteria, customer responsibilities, assumptions, change control, IP/data handling (flag items for legal review).

## D. Rapid prototyping & delivery (Note 04)
- [ ] 🟢 D1. **48-hour prototype:** pick a public dataset (tickets, invoices, contracts) and build a walking skeleton (ingest → process → output → metric) + a 20-case golden set.
- [ ] 🟡 D2. **5-day thin slice** on a real or realistic workflow: working demo on real-ish data, eval harness, cost/latency table, known-limitations list, decision log.
- [ ] 🟡 D3. Write a **demo script** (10 min: problem, live demo, one failure path, metrics, ask) and deliver it to a peer; collect feedback.
- [ ] 🟡 D4. Run a **data audit** on a messy dataset: fill rates, duplicates, format inconsistencies, language mix, scan quality; produce a quality report and mitigation plan.
- [ ] 🟡 D5. Classify your prototype's code into spike/prototype/production and harden the expensive-to-retrofit parts (secrets, auth, logging, evals, config).
- [ ] 🔴 D6. **Pilot readout:** produce a 8-section readout (exec summary, results vs targets with CIs, user voice, failure analysis, risks, production plan, asks) using real pilot numbers.
- [ ] 🔴 D7. Extract reusable assets from two projects you built (connectors, eval harness, prompt templates); create a shared library/template repo with docs; measure time saved on a third project.
- [ ] 🔴 D8. Write a **product-feedback memo** (problem, evidence, impact, workaround, proposal) from a real friction you hit with a tool/API you used.

## E. Enterprise integration & data (Note 05)
- [ ] 🟢 E1. Build an **access tracker** template and fill it for a fictional project needing CRM, ERP, SharePoint, and data-warehouse access, including escalation paths.
- [ ] 🟡 E2. Stand up the **mock enterprise sandbox** (Keycloak, Postgres, fake CRM API with rate limits, fake ITSM with webhooks, SFTP, SOAP service, MinIO, egress proxy) via docker-compose.
- [ ] 🟡 E3. Implement **OIDC login** (Keycloak) in a FastAPI app; map groups → roles; enforce RBAC on endpoints.
- [ ] 🟡 E4. Implement **SCIM** provisioning/deprovisioning (basic user create/deactivate) and verify access revocation within seconds.
- [ ] 🟡 E5. Build a **CRM connector** handling pagination, rate limits (429 + Retry-After), retries with backoff, idempotent upserts, and webhook ingestion with signature verification.
- [ ] 🟡 E6. Ingest **batch files from SFTP** (partial files, encodings, schema drift) idempotently; add archive/replay and a DLQ.
- [ ] 🟡 E7. Call a model API **through an egress proxy with an allow-list and TLS interception**; handle custom CA certs; document the failure modes you hit.
- [ ] 🔴 E8. **Permission-aware retrieval:** sync ACLs from a mock SharePoint into your RAG index; write adversarial tests for cross-user leakage (including via cache and follow-up queries).
- [ ] 🔴 E9. Wrap an enterprise system as an **MCP server** with per-user authorization behind a gateway; add audit logs and rate limits.
- [ ] 🔴 E10. Answer a **50-question security questionnaire** for your RAG/agent app (data flows, retention, subprocessors, access control, logging, incident response, AI-specific risks); create an evidence pack (diagrams, policies, test results).
- [ ] 🔴 E11. **Air-gapped dry run:** prepare an offline bundle (containers, model weights, Python wheels, docs), deploy on a machine with no internet, and write the upgrade/rollback procedure.
- [ ] 🔴 E12. Integrate with a **legacy interface** (SOAP/XML or fixed-width files); add schema validation, character-encoding tests, and contract tests.

## F. Communication & change management (Note 06)
- [ ] 🟢 F1. Rewrite three rambling status updates into the weekly-update template (status, progress, metrics, risks/blockers, asks, next steps).
- [ ] 🟡 F2. **Pyramid-principle drill:** take a technical result (e.g., "recall@5 improved from 0.71 to 0.84 using hybrid + rerank") and write executive, technical, and end-user versions (3–5 lines each).
- [ ] 🟡 F3. Explain **82% field accuracy** and **hallucination risk** to a non-technical COO in 60 seconds; record; test with a non-technical friend and iterate until they can restate it correctly.
- [ ] 🟡 F4. Facilitate a **60-minute workshop** (process mapping + use-case prioritization) with 3–5 people; prepare agenda, pre-read, board; collect feedback.
- [ ] 🟡 F5. Create an **adoption plan** (ADKAR) for rolling a copilot to 400 agents: champions, training, feedback loops, metrics, resistance handling.
- [ ] 🟡 F6. Write a **user training one-pager** (task-based) and a 10-minute recorded walkthrough; test with a real user.
- [ ] 🔴 F7. **Bad-news role-play:** deliver "accuracy is 64% vs 85% target" to a skeptical VP; record; critique against facts→impact→options→recommendation→ask.
- [ ] 🔴 F8. **Difficult stakeholder role-plays** (demanding exec, security gatekeeper, resistant user, absentee sponsor, AI-optimist); rotate roles with a peer; collect rubric feedback.
- [ ] 🔴 F9. **Ethical pushback:** a customer wants fully automated decisions about loan denials with no human review. Write the memo/talk track to decline, explain risks, and propose a compliant alternative.
- [ ] 🔴 F10. Write a **blameless post-mortem** for a (real or simulated) incident with timeline, root causes, detection gaps, and prevention actions.

## G. Production, adoption & handoff (Note 07)
- [ ] 🟡 G1. Produce a **production readiness review (PRR)** for one of your Phase 5–7 projects using the checklist; score each item and create a gap-closing plan.
- [ ] 🟡 G2. Write a **runbook** (deploy, rollback, key rotation, reindex, model change, common incidents) and have a peer follow it cold; fix gaps.
- [ ] 🟡 G3. Define **support tiers and SLAs** for a copilot; draft the escalation matrix and incident severity definitions.
- [ ] 🟡 G4. Build a **value-realization dashboard** (baseline vs current, adoption, quality, cost per task) with synthetic data; write a 1-page QBR summary.
- [ ] 🔴 G5. **Handoff drill:** package a project (architecture, runbook, eval guide, config reference, security dossier, user guide, limitations); pair with a peer who must operate it; measure time-to-first-successful-operation; iterate.
- [ ] 🔴 G6. **Pilot-purgatory autopsy:** analyze a real or hypothetical failed pilot; identify root causes from the Note 07 table and rewrite the plan so it would have succeeded.
- [ ] 🔴 G7. Plan an **expansion roadmap** from use case #1 to #3: reusable components, governance, gates, and champion strategy.

## H. Case practice (Note 08)
- [ ] 🟢 H1. For each of the 10 cases, write a 5-line "what I'd ask first" list before reading the solution; compare afterwards.
- [ ] 🟡 H2. Pick two cases; produce a full **discovery → scope → eval plan** (3 pages each).
- [ ] 🟡 H3. Create your own case study from a project you built (sanitized): situation, discovery, design, pitfalls, metrics, lessons.
- [ ] 🔴 H4. **Miniature engagement:** implement a scaled-down version of one case with synthetic data and publish artifacts (discovery notes, scope, eval report, system card, runbook).
- [ ] 🔴 H5. Write two more **failure case studies** (post-mortem style) based on your own mistakes or public incidents.

## I. Interview prep (Note 09)
- [ ] 🟡 I1. **Story bank:** write 10 STAR+Learning stories across the themes; practice each to ≤2 minutes; get feedback.
- [ ] 🟡 I2. **Decomposition mock loops:** 6 sessions × 30 minutes with a peer/mentor; use different prompts; record and review.
- [ ] 🟡 I3. **Debugging mock rounds:** 4 sessions; a peer plants 3–5 subtle bugs in 250–400 lines of unfamiliar code; fix in 45 minutes while narrating.
- [ ] 🟡 I4. **Learn-the-docs drill:** pick an unfamiliar API (e.g., a payments or calendar API); in 45 minutes read the docs and implement a small working task.
- [ ] 🟡 I5. **System design mocks** (AI/RAG/agents in enterprise settings): 4 sessions; practice requirements → architecture → evals → security → rollout → cost.
- [ ] 🔴 I6. **Take-home simulation (4–6 hours):** "build a RAG/agent solution for X with evals and a write-up"; timebox, deliver a README with assumptions, results, limitations; get critique.
- [ ] 🔴 I7. **Full mock loop (4 hours):** recruiter chat, coding, decomposition, debugging, design, customer role-play, behavioral; get structured feedback; repeat after improvements.
- [ ] 🔴 I8. **Project deep-dive rehearsal:** present your best project in 10 minutes; endure 20 minutes of hostile questions; refine.
- [ ] 🔴 I9. Prepare a **target list** (10 companies/roles), tailor resumes/portfolio links, and write 5 outreach messages to people in the role.
- [ ] 🔴 I10. **Offer-evaluation worksheet:** compare two hypothetical offers across comp, equity risk, role type, travel, growth, mission; practice a negotiation conversation.

## J. Mini-capstones (pick 2)
- [ ] 🔴 J1. **Full simulated engagement** (see [[13-Projects-and-Checkpoint]]).
- [ ] 🔴 J2. **Customer-ready governance + security pack** for an LLM app (data flows, threat model, red-team report, system card, SOC2-style control mapping).
- [ ] 🔴 J3. **Enterprise integration reference implementation** (SSO + SCIM + CRM + ITSM + permission-aware RAG) with docs and tests.
- [ ] 🔴 J4. **FDE playbook repo:** templates (discovery guide, scope, ROI, risk register, weekly update, PRR, runbook, handoff) + two worked examples.

## Cadence
Weekly: 1 real-people interaction (discovery/demo/role-play), 1 written artifact, 1 mock interview round. Keep a **field journal**: what surprised me, what I'd do differently, which templates I improved. Review [[11-QA-Interview-Questions]] on Sundays.
