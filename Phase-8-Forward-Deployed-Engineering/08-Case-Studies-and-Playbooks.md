---
tags: [phase-8, case-studies, playbooks]
---
# 08 — Case Studies & Playbooks

> **These are illustrative, composite scenarios** written to teach patterns — they are **not** accounts of real company engagements, and the numbers are plausible examples, not benchmarks. Use them to practice: read the setup, **pause and write your own decomposition/plan**, then compare with the walkthrough. Each case follows: *Situation → Discovery → Design → Pitfalls → Metrics → Outcome/lessons.* Also use them as raw material for interview stories (swap in your own project details honestly).

## Case 1 — Support copilot for a telecom (RAG + CRM)
**Situation.** 400 tier-1 agents; avg handle time 9 min; 40k tickets/month; KB of 12k articles (many stale); CRM in Salesforce-like system; goal: cut handle time without policy violations.
**Discovery.** Shadowed 6 agents for 3 hours each: ~45% of time searching KB/CRM, ~20% retyping, ~15% waiting on tools. Read 200 tickets: top two categories (billing disputes, plan changes) = 38% volume with clear policies. Found 3 conflicting KB versions for refund rules. Security: read-only CRM, PII present, no customer data to third parties without DPA.
**Design.** Agent side-panel (embedded in CRM): account summary (CRM read-only API) + RAG over *current* KB with citations + drafted reply; hybrid retrieval + reranker; metadata on `effective_date`/`status`; abstain when no policy found; no auto-send. Golden set: 150 tickets labeled by 3 SMEs; calibrated judge for policy-correctness; A/B on handle time.
**Pitfalls.** Stale/duplicate KB (fix: deprecate old versions, add recency/status filters, KB owner role); agents distrusted drafts (fix: citations + "view source" + 10-min training + champions); CRM API latency (cache account summaries); prompt injection via customer emails quoted in tickets (delimit as data; no tool-write capability).
**Metrics.** Primary: median handle time (−28% pilot); guardrails: policy violations = 0 in audit sample (n=300), draft acceptance 62%, edit distance trend ↓, CSAT neutral/up. Cost: ~$0.06/ticket with caching and a mid-tier model.
**Lessons.** The KB hygiene work mattered more than model choice; embedding in the agent's existing tool drove adoption; the abstain behavior built trust.

## Case 2 — Claims intake for an insurer (document extraction)
**Situation.** 120k claims/year; PDFs, scans, emails; manual data entry; cycle time 11 days; leakage concerns.
**Discovery.** Mapped the process: intake → classification → data entry → coverage check → adjuster review → payment. 60% of time in intake/entry. Document types: FNOL forms, repair estimates, medical bills, police reports. Samples: 30% scans with skew/low quality; handwriting present.
**Design.** Classify → type-specific schemas → extraction with evidence quotes → validation rules (totals/dates/policy numbers vs master data) → confidence via dual extraction disagreement → human review queue with highlighted source → idempotent write to claims system. Vision-LLM for scans vs OCR+layout parser A/B per doc type.
**Pitfalls.** Handwriting accuracy low (route to human; collect labels); policy-number format varies (regex + lookup validation); concurrency with legacy claims system (queue + idempotency keys); PHI/PII handling (VPC deployment, redaction in logs).
**Metrics.** Field-level F1 by doc type (e.g., 0.96 digital, 0.83 scans), straight-through rate 58%, review time −55%, error rate vs human baseline comparable; cost/doc $0.11.
**Lessons.** Document-type slicing exposed where to invest; human-review UX produced training labels; validation rules caught most hallucinated amounts.

## Case 3 — Contract review for a legal team (long-document RAG + comparison)
**Situation.** Review vendor contracts against playbook (30 clause types); lawyers spend 2–3 hours/contract.
**Discovery.** Interviews: lawyers fear missed risks and "confident wrong" answers; playbook has fallback positions. Contracts 20–150 pages.
**Design.** Clause-structure-aware parsing with section paths; per-clause retrieval + comparison to playbook; output: risk flags with quotes and page cites; **no legal advice framing**, lawyer-in-the-loop; native citations/quote verification; unanswerable → "not found."
**Pitfalls.** Cross-references ("subject to Section 12.3") require multi-hop retrieval; defined terms; redlines/track changes in DOCX; false negatives are costly → optimize recall for high-risk clauses, accept more false positives.
**Metrics.** Recall on high-risk clause detection 94% (target 90%), precision 71%; time per contract −45%; zero fabricated quotes after verification step.
**Lessons.** Align metric to risk asymmetry (recall first); lawyers became labelers and champions; contract-wide cross-reference handling needed agentic navigation tools.

## Case 4 — Supply-chain exception agent (agent + ERP integration)
**Situation.** Planners manually handle late shipments, stock-outs, and PO mismatches across ERP/WMS/email; 300 exceptions/day.
**Discovery.** 70% of exceptions follow 12 patterns with documented resolutions; 30% need judgment. ERP write access is highly restricted; approvals exist for expedites > $10k.
**Design.** Agent with read tools (ERP, WMS, carrier tracking), **proposal tools** (draft expedite/PO change/email to supplier) requiring planner approval; MCP servers behind gateway; per-user authorization; budget/step caps; trace-based dashboards; eval with 100 historical exceptions replayed in a sandbox.
**Pitfalls.** ERP rate limits; stale cache leading to wrong actions (fresh reads before proposal); agent "creative" actions outside policy (tool allow-list; policy checks in code); trust building (explain with evidence links).
**Metrics.** pass^3 on replay suite 0.81; planner acceptance 68%; exception resolution time −35%; zero unauthorized writes; cost/exception $0.40.
**Lessons.** "Propose-then-approve" delivered value quickly and safely; read-only-first and replay evals de-risked autonomy.

## Case 5 — KYC document processing for a bank (compliance-heavy)
**Situation.** Onboarding delays from manual review of IDs, proof of address, corporate docs across jurisdictions.
**Discovery.** Strict regulatory audit trails; data cannot leave bank VPC; model risk management (MRM) framework requires validation, documentation, monitoring. Existing vendor OCR; escalation to compliance officers.
**Design.** Deployed inside bank VPC using cloud-private model endpoints (or open-weight on-prem if required); extraction + consistency checks (name/address/DOB across docs) + sanctions screening via existing service; **decision support only** (officer decides); full audit logging; fairness slice analysis across nationalities/doc types; governance pack for MRM.
**Pitfalls.** Document fraud/forgery signals need specialized tools (don't claim LLM detects forgeries reliably); multilingual docs; MRM timelines (weeks); change control for any prompt/model update.
**Metrics.** Extraction accuracy by jurisdiction/doc type; officer time per case −40%; zero PII incidents; MRM approval achieved with documented limitations.
**Lessons.** Governance deliverables were as important as code; framing as decision support smoothed approvals.

## Case 6 — Clinical documentation assistant (healthcare, HIPAA)
**Situation.** Clinicians spend hours on notes; hospital wants ambient note drafting from visit transcripts.
**Discovery.** PHI; BAA required; EHR integration via FHIR; clinicians skeptical of hallucinations; liability concerns; time-poor users.
**Design.** Transcription (approved vendor) → structured note draft (SOAP) with **sentence-level source links to transcript**; clinician must review/sign; no autonomous chart writes; evaluation by clinicians on 200 encounters (omissions, fabrications, severity); strict logging and access control; PHI redaction in telemetry.
**Pitfalls.** Omission errors (silent) are as dangerous as fabrications → measure both; specialty vocabulary; EHR write-back approvals; clinician workflow friction.
**Metrics.** Fabrication rate 0.4% (all minor, caught in review), omission rate 3% flagged; documentation time −35%; satisfaction 4.3/5; zero PHI incidents.
**Lessons.** Provenance links and review-by-design were adoption keys; define "harm severity" taxonomy with clinicians.

## Case 7 — Internal IT helpdesk agent (agents + ITSM + SSO)
**Situation.** 8k employees; 25k tickets/quarter; 40% password/access/how-to.
**Discovery.** ServiceNow-like ITSM; Okta-like IdP; strict approval for access changes; many how-to answers in Confluence.
**Design.** Slack/Teams bot: RAG for how-to; tools for ticket lookup/creation, **access request submission** (approvals unchanged), password-reset flow via IdP self-service links (not direct reset); per-user identity enforced; injection tests with malicious pasted logs; eval on 150 historical tickets.
**Pitfalls.** Over-privileged service account in prototype (replaced with delegated tokens); users pasting secrets (redaction); bot answering outside scope (guardrails + scope classifier).
**Metrics.** Deflection 31% (verified resolutions), CSAT parity, 0 unauthorized actions, ticket creation quality ↑ (structured summaries).
**Lessons.** Don't re-implement approvals — integrate with them; measure verified resolution, not "chats."

## Case 8 — Sales research & outreach assistant (web tools + CRM + safety)
**Situation.** Reps spend 40% time on account research and emails.
**Discovery.** CRM, news/web sources, internal playbooks; brand/compliance constraints; fear of hallucinated claims in emails.
**Design.** Research agent (web search/fetch with allow-lists + CRM read) → structured account brief with cited sources → email draft; reps approve/send; **no auto-send**; claims must be cited; guardrails for tone/compliance; eval using rep ratings and fact-check sample.
**Pitfalls.** Prompt injection from web pages; stale data; privacy (scraping personal data); cost of browsing (budgets).
**Metrics.** Research time −60%; factual error rate in briefs 2% (sampled); reply rate neutral/up; cost/brief $0.35.
**Lessons.** Citation-or-silence policy and limited tools made the agent safe enough to ship; manage cost via effort routing.

## Case 9 — Manufacturing QA knowledge assistant (air-gapped, on-prem)
**Situation.** Plant engineers search manuals, NCRs, SOPs; network is air-gapped; legacy Windows environment.
**Discovery.** No internet; Windows laptops; GPU server available (2×H100-class); documents in file shares with scans; safety-critical context.
**Design.** Open-weight LLM + embedder + reranker served on-prem (vLLM); offline bundle (containers, weights, wheels); hybrid retrieval with BM25 + vectors in Postgres; ACL sync from AD; citations with page images; update/rollback procedure documented; limited UI in browser.
**Pitfalls.** Quality gap vs frontier models (mitigated with better retrieval and prompts, smaller scope); weight loading times; GPU capacity planning; no telemetry egress (local observability).
**Metrics.** Answer correctness 84% on SME-labeled set (n=200), search time −50%, 0 data egress; maintenance playbook accepted.
**Lessons.** Constraints drive architecture; **sizing + offline operations** were the real engineering; set expectations on model capability honestly.

## Case 10 — Data-analyst agent for a retailer (text-to-SQL + governance)
**Situation.** Business users wait days for analyst queries; warehouse with 400 tables, inconsistent metrics.
**Discovery.** Metric definitions disputed ("revenue" varies); PII in some tables; analysts cautious about accuracy.
**Design.** Semantic layer for certified metrics/views; **read-only role limited to curated views**; schema+glossary in context; SQL validation (sqlglot), row/time limits; show SQL + explanation + sample; ask clarifying questions; execution-accuracy eval on 100 gold questions; red-team for destructive/PII queries.
**Pitfalls.** Ambiguous questions (clarify), join explosions (validators + EXPLAIN), metric drift, trust (always show SQL and lineage).
**Metrics.** Execution accuracy 78% overall (92% on certified metrics), request backlog −40%, zero policy violations; analysts shifted to higher-value work.
**Lessons.** Fix the semantic layer first; constrain scope to certified data.

## Failure case studies (post-mortem style)
### F1 — The impressive demo that never shipped
**What happened:** a polished prototype on cherry-picked contracts; no golden set; security review started in week 10; sponsor changed roles.
**Root causes:** no production path/criteria; late security; single-threaded relationships; no measured baseline.
**Do instead:** gates + early security + multi-thread + baseline + golden set.
### F2 — Accuracy "improved" but users revolted
**What happened:** model upgrade raised judge scores but changed tone/format; agents' edits increased; complaints spiked.
**Root causes:** judge not calibrated to user preferences; no canary; no stakeholder communication.
**Do instead:** user-signal metrics, canary, release notes, rollback ([[../Phase-7-Evals-Observability-LLMOps/04-Release-Management-and-Experimentation]]).
### F3 — Permission leak in RAG
**What happened:** assistant surfaced HR docs to unauthorized users via a cached answer.
**Root causes:** ACLs not enforced at retrieval; semantic cache keyed without user/ACL context; no leakage tests.
**Do instead:** permission-aware retrieval, cache keys with ACL scope, adversarial tests, audit logs.
### F4 — The cost spike
**What happened:** agent loops on edge cases drove a 5× bill in a week.
**Root causes:** no per-run budgets; no cost alerts; retries on non-idempotent tools.
**Do instead:** budgets/step caps, anomaly alerts, idempotency, kill switch.
### F5 — Over-customization trap
**What happened:** FDE built bespoke connectors/UI per customer; no reuse; team saturated.
**Root causes:** no productization discipline; no feedback to product; scope creep.
**Do instead:** reuse metrics, productize repeated needs, enforce change-request process.

## The meta-playbook (extracted patterns)
1. **Embed in the workflow**, not a separate app. 2. **Start read-only / propose-then-approve.** 3. **Measure a baseline and a golden set before building.** 4. **Slice results** by document type/segment; fix the weakest slice. 5. **Show sources and abstain** when unsure. 6. **Start security and access requests on day 1.** 7. **Pick champions and train in context.** 8. **Plan production, ownership, and run costs before the pilot ends.** 9. **Document limitations honestly.** 10. **Productize what repeats.**

## How to use these cases
- **Interview practice:** pick a case, give yourself 5 minutes to ask clarifying questions, 10 to decompose, 10 to propose a plan and metrics ([[09-FDE-Interview-Prep]]).
- **Portfolio:** build a miniature of one case with synthetic data and publish the artifacts (discovery notes, scope, eval, system card).
- **Retros:** compare your project against the meta-playbook.

## Self-check
1. For Case 3, why optimize recall over precision, and how do you communicate it? 2. Which cases required "propose-then-approve," and why? 3. List the first week of work for Case 9 given no internet access. 4. Identify the root causes and fixes for F3. 5. Create your own case from a domain you know.

Next → [[09-FDE-Interview-Prep]]
