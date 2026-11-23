---
tags: [phase-8, discovery, decomposition]
---
# 02 — Discovery & Problem Framing

> The single biggest cause of failed AI projects is **solving the wrong problem well**. Discovery is how you avoid that. It is also the core of the Palantir-style **decomposition** interview: *take a vague, real-world challenge, break it into parts, and propose a solution.*

## 1. What discovery must produce
By the end of discovery (days to 2 weeks) you should be able to state, in one page:
1. **The problem** in the customer's words **and** in measurable terms (who suffers, how often, what it costs).
2. **The workflow** as it works *today* (steps, handoffs, systems, exceptions, volumes, time per step).
3. **The users & stakeholders** (who uses it, who decides, who blocks, who pays).
4. **The data & systems** involved (where it lives, quality, access, permissions, owners).
5. **The success metric(s)** with a **baseline** and a target.
6. **Constraints** (security, compliance, latency, budget, timeline, change tolerance, tech stack).
7. **Candidate solutions** and a **recommendation** (including "don't use AI here" if true).
8. **Risks & assumptions** to test first.

## 2. Stakeholder mapping
| Role | Cares about | Typical concerns | How to engage |
|---|---|---|---|
| **Executive sponsor** | outcomes, ROI, risk, strategic narrative | cost, reputation, timelines, compliance | brief, metric-driven updates; ask for decisions & air cover |
| **Economic buyer** | budget/value | payback, vendor risk | ROI model, references, staged commitment |
| **Champion** | wants the project to succeed | their own credibility | arm them with wins, narratives, early access |
| **End users** | daily workflow, workload | job security, extra work, trust | observe, co-design, train, respect expertise |
| **Domain SMEs** | correctness, edge cases | being replaced, bad outputs | make them reviewers/labelers; credit them |
| **IT / platform / data owners** | stability, access, standards | security, support burden, shadow IT | engage early; follow their processes |
| **Security / privacy / legal / compliance** | risk | data leaving boundary, regulation, liability | early security review; evidence pack ([[../Phase-7-Evals-Observability-LLMOps/08-Governance-Compliance-and-FinOps]]) |
| **Skeptics / blockers** | status quo, prior failed projects | "AI doesn't work here" | listen; find the legitimate concern; design a test |
| **Procurement / finance** | contracts, vendor onboarding | timelines | start paperwork early |
Draw a **power/interest map** and a **who-can-say-no** list. Many deployments fail because an unconsulted gatekeeper vetoes late.

## 3. Discovery interviews (how to ask)
**Principles (from customer-discovery practice, e.g., *The Mom Test*):** ask about **specific past behavior**, not hypotheticals or opinions about your idea; talk **less**; **dig into emotion and cost**; **look for evidence of effort already spent** (workarounds, spreadsheets, hires) as proof of real pain; avoid pitching.
**Question bank**
- *Context:* "Walk me through the last time this happened. What triggered it? Then what?" 
- *Frequency/volume:* "How many per week? Peak? Seasonality?" 
- *Time & cost:* "How long does each take? Who touches it? What does a mistake cost?" 
- *Exceptions:* "What goes wrong? What's the weirdest case? What do you do when the system says X?" 
- *Tools & data:* "Where do you look? Which systems? Who owns them? Can I see an example (redacted)?" 
- *Workarounds:* "What have you tried? What's in that spreadsheet?" 
- *Decision criteria:* "How do you know it was done right? Who checks?" 
- *Impact:* "If this were solved, what changes for you? For the business?" 
- *Constraints:* "What can't change? Who must approve? What happened last time someone tried to automate this?" 
- *Success:* "How will we know in 6 weeks it's working?" 
- *Closing:* "Who else should I talk to? What did I forget to ask?"
**Avoid:** leading questions ("Would an AI assistant help?"), solutions-first talk, jargon, interrupting silence, over-promising, treating the sponsor's view as the user's reality.
**Techniques:** **shadowing/contextual inquiry** (watch real work for 2–4 hours), **artifact collection** (real documents, tickets, emails, spreadsheets), **process walkthroughs**, **5 Whys**, **jobs-to-be-done** framing ("When ___, I want to ___, so I can ___"), **sampling** (read 50 real cases yourself — this is *error analysis for the human process*).

## 4. Understanding the current workflow
Produce a **process map** (swimlane): actors, steps, systems, decision points, handoffs, wait times, volumes, error/rework loops. Annotate each step with **time, cost, failure rate, data used, and whether it's judgment vs routine**. Identify:
- **Bottlenecks** (long waits, queues), **rework** (errors found late), **handoffs** (information loss), **judgment-heavy** steps (where AI assists vs decides), **routine** steps (automation candidates), **compliance checkpoints** (must stay human/auditable).
- **Value location:** where would saving time/risk matter most? (Not always where the loudest stakeholder points.)

## 5. Decomposition (the core skill)
Break a vague challenge into **independent, testable sub-problems**. A reusable structure:
1. **Clarify the goal:** who/what/why/success metric/constraints/timeline. Ask clarifying questions *first*; restate the problem.
2. **Identify actors & workflows:** who does what today.
3. **Enumerate data/system inputs & outputs:** what comes in, what must come out, where it goes.
4. **Split into components:** e.g., ingest → classify → extract → decide → act → monitor → feedback.
5. **For each component:** options (rules/ML/LLM/human), risks, data needs, evaluation approach, effort.
6. **Prioritize:** value × feasibility × risk → pick a **thin slice** that proves the riskiest assumption.
7. **Propose architecture & plan:** MVP, milestones, success criteria, what could go wrong, how you'd measure.
8. **State assumptions** and **open questions**; show tradeoffs; invite feedback.
**Example decomposition — "Reduce claims-processing time at an insurer":** goal metrics (cycle time, accuracy, leakage) → claim types & volumes → intake channels (email, portal, PDFs) → document types → extraction fields → policy coverage rules → fraud signals → adjuster decision points → payments system integration → audit/compliance → exception handling → change management. Slice 1: **intake classification + extraction for the top 2 doc types of the most common claim type**, with human review, measured on straight-through rate and field accuracy.
**Communication tips for interviews & real life:** think aloud, draw a diagram, state assumptions, check in ("does this match your situation?"), quantify, propose a first step you can start Monday.

## 6. Is AI the right tool? (feasibility triage)
| Question | If yes → | If no → |
|---|---|---|
| Is the task **language/unstructured-data heavy** (docs, emails, tickets, calls)? | LLMs/RAG strong fit | consider rules/ML/SQL |
| Is the logic **deterministic & well-specified** (calculations, validations)? | **code**, not LLM | — |
| Is **variability** high (many formats/phrasings)? | LLM adds value | rules suffice |
| Is a **wrong answer** costly/irreversible? | human-in-loop, verification, narrow scope | can automate more |
| Is **ground truth** obtainable (labels, outcomes, SME review)? | measurable → proceed | can't evaluate → don't automate |
| Is **data accessible & permitted**? | proceed | fix access first (often the real blocker) |
| Is the **volume** big enough to matter? | ROI plausible | maybe not worth building |
| Is **latency/cost** compatible with LLM calls? | ok | smaller models/caching/precompute |
**Value/Feasibility/Risk grid:** score each candidate use case (1–5) on **business value**, **technical feasibility** (data, model capability, integration), **risk** (compliance, error cost), **adoption likelihood** (champion, workflow fit). Start with **high value, high feasibility, low–medium risk, strong champion**. Resist the shiny, risky, politically loaded first project.
**Common "AI isn't the answer" findings:** the real bottleneck is **data access or process design**, a **form/field change** eliminates the need, a **SQL report** solves it, or the **approval policy** (not drafting speed) is the constraint.

## 7. Metrics & baselines
- Pick **1 primary metric** + **guardrail metrics** (quality, risk, cost, latency). E.g., primary: *median time to resolve*; guardrails: *policy violations = 0*, *escalation correctness ≥ x%*, *CSAT not worse*.
- **Measure the baseline now** (before building): time per task, error rate, volume, cost — otherwise you can't prove value. Sample real cases; time them; get SME judgments.
- Convert to **dollars/risk** where possible; define **minimum viable improvement** that justifies production.
- Agree **how success will be evaluated and by whom** (golden set with SMEs; [[../Phase-7-Evals-Observability-LLMOps/01-Eval-Foundations]]).

## 8. Data & systems discovery (technical)
Inventory: **sources** (systems, tables, file shares, APIs), **formats**, **volumes**, **freshness**, **quality** (nulls, duplicates, versions), **sensitivity** (PII/PHI/confidential), **ownership**, **access method** (API, DB read replica, export, SFTP), **permissions model**, **change process**, **sandbox availability**, **latency of access approvals** (weeks!). Get **samples early** (redacted/synthetic if needed). Ask: "Which system is the source of truth for X?" "What breaks if we read from there?" "Who can grant access, and how long does it take?"
**Data audit quick checks:** sample 100 records → fill rates, duplicates, inconsistent formats, language mix, scan quality, join keys; document findings and risks.

## 9. Risks, assumptions & the riskiest-assumption-first test
List assumptions (data accessible? accuracy attainable? users will adopt? security will approve? latency acceptable?). Rank by **impact × uncertainty**. Design the **cheapest test** that could falsify the riskiest one (e.g., run an LLM over 50 real documents and measure accuracy *this week*; request read-only data access *today*; show a mock UI to 3 users). **Fail fast, cheaply.**

## 10. Discovery artifacts (templates to build)
- **Stakeholder map & interview schedule**
- **Interview notes** (with verbatim quotes, observations vs interpretations)
- **Current-state process map** + pain-point annotations
- **Problem statement:** "For [user], who [struggles with X], we will [outcome] so that [metric]. Success = [baseline → target]."
- **Use-case scorecard** (value/feasibility/risk/adoption)
- **Data & systems inventory** with access status
- **Constraints & non-goals**
- **Risk/assumption register** with first tests
- **Recommendation memo** (1–2 pages): what to build first, why, what you'd do next, what you need from the customer.
- **Discovery readout** (30 min): findings, the problem, options, recommendation, ask.

## 11. Common discovery mistakes
Taking the sponsor's framing as truth · interviewing only managers · pitching instead of listening · skipping observation of real work · no baseline · picking the use case by hype · ignoring access/security timelines · not reading real examples · assuming data quality · failing to identify the gatekeeper · over-scoping the first project · hiding "AI isn't needed" findings.

## Self-check
1. Write the discovery output checklist from memory. 2. Rewrite three leading questions into non-leading ones. 3. Decompose "improve customer-support quality for a telecom" into components and pick a thin slice. 4. Fill in a value/feasibility/risk grid for four use cases. 5. Name three signs that AI isn't the right tool. 6. What's your riskiest-assumption test for a contract-review pilot?

Next → [[03-Scoping-Planning-and-Commercials]]
