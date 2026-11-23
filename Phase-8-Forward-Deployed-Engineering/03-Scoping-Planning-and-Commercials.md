---
tags: [phase-8, scoping, planning, roi]
---
# 03 — Scoping, Planning & Commercials

> Scope is where engagements are won or lost. Under-scope and you deliver something trivial; over-scope and you miss dates, burn trust, and deliver nothing production-ready. Your job: **a small, valuable, measurable first step with explicit exit criteria — and a credible path from there to production.**

## 1. The scoping principles
1. **Thin vertical slice over broad horizontal layer.** One user, one workflow, end-to-end (data in → value out → measured), rather than "build the platform."
2. **Outcome-defined, not feature-defined.** "Reduce median ticket handling time 30% for tier-1 password/billing issues with zero policy violations" beats "build an AI assistant."
3. **Time-boxed with decision gates.** Pilot of 4–8 weeks with **go / iterate / stop** criteria agreed up front.
4. **Explicit non-goals.** What you will *not* do (other departments, languages, channels, custom UI, model training).
5. **Evidence-driven expansion.** Earn scope with measured results.
6. **Plan for production from day one.** A pilot that can't become production is a demo.
7. **Shared risk, shared responsibility.** The customer owns data access, SME time, decisions, and change management.

## 2. The one-page scope (template)
```
Title: ______________________      Owner(s): FDE ___ | Customer sponsor ___ | Champion ___
Problem & users: who, what pain, volume, cost today (baseline)
Goal & success metrics: primary + guardrails; baseline → target; how/by whom measured
MVP / pilot scope: user group, workflow(s), channels, data sources, integrations, languages
Non-goals: ...
Approach (high level): architecture sketch; build vs configure; human-in-the-loop design
Evaluation plan: golden set (size, who labels), metrics, thresholds for go/iterate/stop
Timeline & milestones: weekly deliverables; demo dates; decision gate dates
Dependencies & customer responsibilities: access, SMEs (hours/week), approvals, environments, data
Risks & mitigations: top 5 (security approval, data quality, adoption, accuracy, vendor/model changes)
Security & compliance: data classes, residency, retention, approvals needed, reviewers
Production path: what must be true to move beyond pilot; estimated effort; operating model
Communication: cadence (weekly update, biweekly demo), channels, escalation path
```

## 3. Success criteria & exit gates (make "done" unambiguous)
- **Quantitative:** "≥85% field-level accuracy on the golden set (n=150), ≥70% straight-through, p95 latency ≤ 10 s, 0 permission violations, cost ≤ $0.20/doc."
- **Qualitative:** user satisfaction (≥4/5 from 10 pilot users), SME sign-off on failure analysis, security approval received.
- **Gates:** *Gate 1 (week 2)*: data access + baseline + golden set ready; *Gate 2 (week 4)*: prototype meets ≥X on golden set; *Gate 3 (week 8)*: pilot results vs targets → **go (production plan) / iterate / stop**.
- Include **what happens if targets are missed** (extend with changes? narrow scope? stop?). Pre-agree to avoid goalpost wars.
- Specify **evaluation ownership** — who labels, who signs off, which dataset is the arbiter (frozen test set; [[../Phase-7-Evals-Observability-LLMOps/01-Eval-Foundations]]).

## 4. Estimation under uncertainty
- **Decompose** work into tasks (discovery, data access, ingestion, prototype, evals, integration, UI, security review, hardening, training); estimate each with **ranges** (best/likely/worst) and **confidence**.
- **Known multipliers in enterprise work:** *access & approvals* (weeks, calendar-bound, not effort-bound), *data cleaning*, *integration surprises*, *SME availability*, *review cycles*, *environment setup*, *change-control windows*. Pad calendar time, not effort.
- **Critical path & parallelization:** start slow, calendar-bound items (access requests, security review, procurement, VPN/network allow-lists) **on day 1**.
- **Spikes:** time-boxed technical investigations to retire risk before committing.
- **Buffers & communication:** commit to *milestones with confidence levels* ("80% confident demo-ready by Oct 15; risk: SSO approval") rather than false precision; update early when slipping.
- **Reference classes:** use past engagements' actuals (keep a log!).
- **AI-specific uncertainty:** *accuracy ceiling is unknown until you test on real data* → plan an early **feasibility checkpoint** and be willing to change approach/scope.

## 5. Roles, RACI & ways of working
| Activity | FDE | Customer PM/sponsor | SMEs | IT/Sec | Product/Eng (vendor) |
|---|---|---|---|---|---|
| Define success metrics | R | A | C | I | C |
| Provide data access | C | A | I | R | I |
| Golden-set labeling | C | A | R | I | I |
| Build prototype | R/A | I | C | I | C |
| Security review | R | A | I | R | C |
| User training/rollout | R | A | R | C | I |
| Production ops | R→handoff | A | I | R | C |
(R=Responsible, A=Accountable, C=Consulted, I=Informed.) Agree **working agreements**: meeting cadence, response times, decision rights, where artifacts live, how changes to scope are requested and approved.

## 6. ROI & business-case modeling
**Structure:** `Value = (time saved × volume × loaded cost) + (error/rework reduction × cost per error) + (revenue uplift/retention) + (risk reduction × expected loss avoided)`; `Cost = build/engineering + model/inference + infra + licenses + change management + ongoing ops + human review time`.
- **Be explicit about assumptions** and show a **sensitivity table** (accuracy, adoption rate, volume, review rate). Example variables: tickets/month, % automatable, handle time before/after, % requiring human review, cost per token/inference, support cost per hour.
- **Net value must include human review cost** — "AI drafts, human verifies" often yields 30–60% time savings, not 90%.
- **Adoption curve:** value arrives only as users adopt; model a ramp.
- **Payback & confidence intervals:** show conservative/base/optimistic.
- **Intangibles:** speed, quality consistency, employee experience, compliance — mention but don't inflate.
- **Validate with the pilot** (measured numbers replace assumptions). Keep the model in a spreadsheet the customer's finance can inspect.
**Common ROI errors:** counting 100% automation, ignoring review/exception handling, using list-price tokens without caching/batch, ignoring integration/ops costs, assuming immediate adoption, double counting benefits, using vendor marketing stats.

## 7. Commercial awareness (technical person's view)
You're usually not negotiating price, but you must understand how deals shape technical choices:
- **Pricing models:** usage/consumption (tokens/API calls/documents), per-seat, platform/subscription tiers, outcome-based (per resolved ticket/successful extraction), professional-services fees (T&M, fixed-scope), committed-spend discounts. Each shapes incentives (e.g., usage pricing encourages efficiency; seat pricing encourages broad adoption).
- **Pilot structures:** paid pilots (better signal), free POCs (risk of low commitment), design-partner agreements (discounts for feedback/case study rights).
- **Contract artifacts you may touch:** SOW (scope, deliverables, acceptance criteria), DPA/security addendum, SLA/support terms, IP ownership/licensing of custom work, publicity rights.
- **IP & reuse:** clarify who owns custom code/prompts/evals/fine-tuned weights; make sure reusable components can flow back to product without leaking customer data.
- **Don't over-commit:** never promise accuracy/dates/features in conversations that aren't in the agreed scope; route commercial changes to the account owner; document change requests.
- **Know your company's FDE motion:** is it a paid service, a loss-leader to win strategic accounts, or tied to consumption commitments? It affects how much custom work is appropriate.

## 8. Managing scope creep (and saying no)
- **Change-request process:** every addition is logged with impact (effort, timeline, risk) and a decision (accept/defer/decline).
- **Parking lot:** capture good ideas without derailing; revisit at gates.
- **Say no with data and alternatives:** "That adds a second language and 3 new integrations; it pushes go-live ~5 weeks. Option A: ship English/email first and add Spanish in phase 2 once we hit 85%; Option B: move the date."
- **Anchor on outcomes:** "Does this help us hit the success metric?" 
- **Protect the production path:** don't trade away evals, security, or monitoring to hit a demo date.
- **Escalate early** when scope/dates are at risk; escalate with options, not complaints.

## 9. Risk register (living document)
| Risk | Likelihood | Impact | Early signal | Mitigation | Owner |
|---|---|---|---|---|---|
| Data access delayed | H | H | no ticket progress by week 1 | escalate via sponsor; use synthetic/sample data meanwhile | customer IT |
| Accuracy below threshold | M | H | <70% on first 50 cases | error analysis; retrieval/prompt/model changes; narrow scope; human review | FDE |
| Security approval slow/blocked | M | H | questionnaire unanswered | start day 1; evidence pack; offer VPC/on-prem | FDE + sec |
| Low user adoption | M | H | users skip tool | co-design; champions; training; embed in existing tool | sponsor/FDE |
| Model/API change or deprecation | L–M | M | provider notice | version pinning, eval gate, fallback | FDE |
| Prompt injection/data leak | M | H | red-team finding | guardrails, least privilege, monitoring ([[../Phase-7-Evals-Observability-LLMOps/07-Guardrails-and-Security-Hardening]]) | FDE/sec |
| Champion leaves | M | H | reorg news | multi-thread relationships | FDE |
| Cost exceeds budget | M | M | cost/task > target | caching, routing, batching | FDE |
Review weekly; promote risks to the sponsor when mitigation needs their authority.

## 10. Planning the pilot→production path (before you start the pilot)
Write the **production readiness criteria** early (see [[07-Production-Adoption-and-Handoff]]): SLOs, security sign-off, monitoring, runbooks, training, support model, ownership. Estimate **post-pilot effort** and **ongoing cost** so the sponsor can budget; otherwise a successful pilot dies at the budget stage ("pilot purgatory").

## 11. Example scope: "Support copilot for a mid-size telecom"
- **Problem:** tier-1 agents spend ~9 min/ticket searching KB & CRM; 40k tickets/month; CSAT flat.
- **Goal:** −30% median handle time on billing & plan-change tickets; **0** policy violations; agent satisfaction ≥4/5.
- **Pilot scope:** 20 agents, 2 ticket categories, English, in-agent side panel (draft reply + cited policy + account summary), read-only CRM access, no auto-send.
- **Non-goals:** voice, other languages, auto-resolution, CRM write-back, new UI app.
- **Eval:** 150 real tickets labeled by 3 SMEs; metrics: policy-correctness (binary judge calibrated, κ reported), citation validity, edit distance accepted by agents; online: AHT delta in A/B.
- **Timeline:** wk1 access + baseline + golden set; wk2–3 RAG+prompt prototype; wk4 shadow with agents; wk5–6 live pilot; wk7 analysis; wk8 gate review.
- **Dependencies:** CRM read-only API (IT), KB export (support ops), 4 hrs/week SME time, security review (week 1 start).
- **Risks:** outdated KB articles, CRM latency, agent trust; mitigations in the register.
- **Production path:** SSO integration, monitoring dashboards, runbook, training for 400 agents, owner = support-ops engineering.

## Self-check
1. Write a one-page scope for a contract-review pilot (use the template). 2. Define go/iterate/stop gates with numbers. 3. Estimate a pilot with ranges and name three calendar-bound risks. 4. Build an ROI sensitivity table for 3 variables. 5. Respond to a scope-creep request with options. 6. What must be true to avoid pilot purgatory?

Next → [[04-Rapid-Prototyping-and-Delivery]]
