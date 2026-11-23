---
tags: [phase-8, interview, career]
---
# 09 — FDE Interview Prep

> **Source caveat:** public information on FDE loops comes mostly from prep sites and candidate reports. **Palantir's process is the best documented**; **OpenAI/Anthropic/other loops are thinly sourced and vary by team and time**. Formats below are *reported* patterns, not guarantees. Always read the company's own recruiting guidance (Palantir publishes advice titled "Navigating Open-Ended Questions" per sources — read it first), ask your recruiter for the exact loop, and confirm policies like **AI-tool use during interviews** (reported as prohibited at some firms).

## 1. What FDE interviews are really testing
Reported consensus: **judgment and customer-facing reasoning weigh about as much as coding.** Dimensions:
1. **Problem decomposition** under ambiguity (can you structure a messy real-world problem?).
2. **Technical depth & fluency** (code, debugging, system design, data, APIs, applied AI).
3. **Learning speed** (can you ramp on unfamiliar docs/systems quickly?).
4. **Customer empathy & communication** (discovery questions, explaining trade-offs, handling pushback).
5. **Ownership & bias to action** (shipping, prioritization, risk management).
6. **Judgment & ethics** (when to say no; safety/privacy).
7. **Motivation & values fit** (why this role/company; mission alignment).

## 2. Typical loop structures (reported)
| Stage | What | Notes |
|---|---|---|
| **Recruiter screen** | background, motivation, logistics (travel, location, work authorization) | clarify role type, team, on-site expectations, comp band |
| **Technical screen** | coding (Python/SQL typical), sometimes data wrangling or API tasks | practical > puzzle; clean, working code and clear communication |
| **Onsite/virtual loop** (e.g., 3–5 rounds of ~45–60 min) | mix of: **decomposition/case**, **coding**, **re-engineering/debugging**, **system/solution design**, **customer scenario/role-play**, **project deep dive**, **behavioral/values** | order/names vary |
| **Take-home (some companies)** | build a small solution (e.g., RAG/agent/data pipeline) with write-up | evaluate engineering + communication + evals |
| **Hiring manager / values / team match** | motivation, culture, expectations; at some AI labs a **values/safety panel** can be decisive (per reports) | prepare thoughtful answers |
**Palantir-style specifics (as reported):** *Decomposition* round (vague real-world problem; mostly discussion; breaking into components and proposing a solution; often weighted heavily), *Re-engineering/debugging* round (read an unfamiliar codebase of a few hundred lines and fix subtle bugs; learn from documentation under time pressure), *Learning* round (unfamiliar docs/API), technical coding, hiring-manager conversation. **Timelines** reported range from ~3 weeks to a few months depending on company and team.
**OpenAI/Anthropic (reported, thin):** take-home or project deep dive; solution design involving retrieval/agents/evals/safety; behavioral and values rounds; decentralized by team.

## 3. Decomposition / case rounds (master this)
### The framework (use it aloud)
1. **Listen & restate** the problem; **ask clarifying questions** (goal, users, scale, constraints, timeline, success metric, existing systems, what's been tried).
2. **State assumptions** explicitly (and confirm).
3. **Map the system/workflow:** actors, inputs/outputs, data, decisions, handoffs; draw it.
4. **Decompose** into components/sub-problems; note dependencies.
5. **Prioritize** by value, feasibility, risk; choose a **first slice** that tests the riskiest assumption.
6. **Propose solution(s):** approach per component (rules/ML/LLM/human), data needs, architecture, integration, evaluation plan, security/compliance, rollout.
7. **Metrics & guardrails:** how you'll know it works; baseline; failure modes.
8. **Risks & mitigations;** what you'd do in week 1 / month 1.
9. **Summarize** and invite feedback; adapt when the interviewer adds constraints.
**Tips:** think aloud; structure verbally ("three parts: data, model, rollout"); quantify (back-of-envelope: volume, cost, time); acknowledge uncertainty and propose experiments; don't jump to "use an LLM" — consider whether SQL/rules suffice; show customer empathy ("who is affected and how will they react?").
### Practice prompts (do 20+ with a timer; speak out loud)
1. "A hospital system wants to reduce no-shows." 2. "A retailer's returns fraud is rising." 3. "A city wants to speed up building permits." 4. "An airline's maintenance logs are messy; reduce delays." 5. "A bank wants to detect suspicious transactions faster." 6. "A utility wants to predict outages from field reports." 7. "A newsroom wants help with investigative research on 1M documents." 8. "A telecom wants to reduce customer churn." 9. "A logistics firm wants to optimize last-mile delivery exceptions." 10. "A university wants to personalize student support." 11. "A pharma company wants faster literature review." 12. "A government agency wants to process benefit applications." 13. "A SaaS company wants an AI support agent." 14. "A factory wants to cut scrap rate." 15. "An insurer wants to automate claims triage." 16. "A legal team wants contract review." 17. "A call center wants QA on 100% of calls." 18. "A grocery chain wants better demand forecasting for perishables." 19. "A media company wants to tag and search its video archive." 20. "A nonprofit wants to match donors to programs."
**Self-grade:** Did I clarify first? State assumptions? Show structure? Identify the riskiest assumption? Propose measurable success? Consider the human side? Offer a first slice?

## 4. Re-engineering / debugging rounds
**What it is (reported):** you're given an unfamiliar codebase (hundreds of lines) with failing behavior or subtle logic bugs; find and fix quickly, explaining your reasoning.
**Approach:** read the entry points & tests; **reproduce** first; form hypotheses; bisect with prints/debugger; check boundary conditions (off-by-one, None handling, time zones, floating point, mutation/aliasing, state leakage, integer division, sort stability, async races); **explain as you go**; write a minimal fix and a test; consider related bugs.
**Practice:** debug open-source bug issues; use bug-injection exercises (write buggy code or have a friend inject 5 subtle bugs); do code-reading drills on 300-line programs; read unfamiliar API docs and implement a small task in 30 minutes.

## 5. Coding rounds
- Practical Python/SQL: parse messy data, call an API with pagination/retries, implement a rate limiter, group/aggregate, write window-function queries, simple algorithms ([[../Phase-1-Foundations/04-Data-Structures-and-Algorithms]], [[../Phase-1-Foundations/10-Exercises]]).
- Applied AI coding: implement cosine search / BM25 / RRF, a tool loop, structured extraction with validation, an eval harness with CIs ([[../Phase-5-RAG-Knowledge-Systems/08-Build-RAG-From-Scratch]], [[../Phase-6-Agents-Tools-MCP/09-Build-an-Agent-From-Scratch]], [[../Phase-7-Evals-Observability-LLMOps/09-Build-an-Eval-and-LLMOps-Pipeline]]).
- **Communicate:** restate, plan, code cleanly, test with examples, discuss complexity/edge cases.

## 6. Solution / system design rounds (AI-flavored)
Common prompts: "Design an enterprise RAG assistant," "Design an agent that processes invoices end-to-end," "Design a support copilot with permissions and audit," "Deploy an LLM app in an air-gapped environment," "Design evals and monitoring for X," "Reduce LLM costs by 50%."
**Structure:** requirements & scale → data & integration → architecture (ingest, retrieval/tools, model/serving, guardrails) → eval & quality → security/permissions/compliance → rollout/ops/monitoring → cost & latency → risks & trade-offs → phased plan.
Lean on the vault: [[../Phase-5-RAG-Knowledge-Systems/07-RAG-Evaluation-and-Production]], [[../Phase-6-Agents-Tools-MCP/07-Agent-Reliability-Safety-and-Security]], [[../Phase-7-Evals-Observability-LLMOps/06-Serving-and-Infrastructure]], [[05-Enterprise-Integration-and-Data]].

## 7. Customer scenario role-plays
You play the FDE; the interviewer plays a customer (exec, skeptic, IT/security, end user).
**Common scenarios & key moves**
- **Skeptical VP:** "Our last AI project failed." → listen, ask what failed, propose a small measurable pilot with exit criteria.
- **Security architect:** "Data can't leave our environment." → options: VPC/private endpoints/open-weight on-prem; data-flow diagram; controls; evidence pack.
- **Frustrated user:** "The AI gave a wrong answer." → empathize, reproduce, show how errors are measured, escalation path, improvement loop.
- **Exec wanting 100% accuracy:** reframe to measurable targets, risk-tiered human review, baseline comparison.
- **Scope creep from sponsor:** acknowledge, quantify impact, offer phased options.
- **Missed deadline:** facts → impact → options → recommendation → ask.
- **Pricing/commercial push:** stay in lane; route to account owner; don't promise.
**Scoring rubric (self/peer):** discovery questions quality · empathy · clarity · structure · honesty about limits · concrete next steps · tone under pressure.
**Drill:** record yourself; review for filler words, rambling, jargon; practice 60-second versions.

## 8. Project deep-dive & take-home tips
- **Pick 2–3 projects** (from Phases 4–7 capstones) and prepare a 10-minute walkthrough: **problem → users → constraints → design choices & alternatives → evals (numbers!) → failures & fixes → results → what you'd do differently.**
- Be ready for drilling: "Why this retriever?" "How did you validate the judge?" "What broke in production?" "What did you cut?" "How would it scale/secure?"
- **Take-home best practices:** read the prompt twice; clarify scope assumptions in writing; **time-box**; deliver a **README** (how to run, design, assumptions, evals, limitations, next steps); include tests and a small eval; show cost/latency; keep the repo clean; be honest about shortcuts. Quality of **communication** often decides.

## 9. Behavioral & values
**Story bank (STAR: Situation, Task, Action, Result — plus *Learning*):** prepare 8–10 stories mapped to themes:
1. **Ownership under ambiguity** 2. **Customer/user empathy** 3. **Conflict/disagreement** 4. **Failure & recovery** 5. **Delivering bad news** 6. **Learning fast** 7. **Technical depth/debugging** 8. **Leadership without authority** 9. **Ethical dilemma/saying no** 10. **Impact with metrics.**
Use real details; quantify results; keep each to ~2 minutes; practice follow-ups ("what would you change?").
**Values/safety conversations (AI labs):** be ready to discuss **responsible deployment** (prompt injection, privacy, human oversight, over-reliance), how you'd handle a customer wanting to deploy something risky, your views on AI risks/benefits (thoughtful, not performative), and why you want *this* mission.

## 10. Questions to ask (and why)
- *Role reality:* "What does a typical engagement look like — length, team structure, on-site vs remote, travel?" "How are FDEs assigned to customers?" 
- *Impact & ownership:* "What does success look like at 6/12 months?" "How much of the work is reusable vs bespoke?" "How do FDE learnings reach product?" 
- *Technical environment:* "What stack do FDEs use? How are evals/guardrails handled? Self-hosted/air-gapped deployments?" 
- *Growth:* "Career paths from FDE? Mentorship? Promotion criteria?" 
- *Culture/sustainability:* "On-call expectations? How is travel managed? Team size/turnover?" 
- *Commercial:* "Is FDE work paid services or tied to consumption? How are customer expectations/scope managed?"
Good questions signal judgment and help you decide whether *you* want the job.

## 11. Preparation plan (6 weeks)
| Week | Focus |
|---|---|
| 1 | Role research; polish 2–3 projects & story bank; clarify target companies/role types |
| 2 | Decomposition drills (10 prompts, spoken + written) |
| 3 | Coding fluency (Python/SQL practical, applied-AI snippets) + debugging drills |
| 4 | System/solution design (AI/RAG/agents, enterprise constraints) |
| 5 | Customer role-plays (record, review), take-home practice, mock loops |
| 6 | Full mock loops with feedback; refine stories; logistics (references, work authorization, comp research) |
**Resources for peers/mocks:** friends/colleagues, communities, paid mock platforms, recording yourself. Aim for **15+ mock rounds** before the real loop.

## 12. Offer, negotiation & choosing
- **Know the level & band;** compare total comp (base, bonus, equity with vesting/valuation realism), location, travel, benefits. Public salary figures for FDE roles are inconsistent — rely on current offers and reputable comp data for your market.
- **Evaluate:** customer mix, engagement model, autonomy, product maturity/revenue, manager, mission, learning, work-life sustainability, visa/location constraints.
- **Negotiate politely with evidence;** ask about equity terms, refreshers, signing bonus, start date, travel reimbursement.
- **Reference checks:** prepare references who can speak to customer-facing impact.

## 13. Red flags to watch for (on either side)
Candidate red flags: jumping to solutions without questions; ignoring users/risks; vague metrics; can't explain their own project; blames others; overclaims AI capability. Company red flags: unclear engagement model, no product-feedback path, FDEs as pure custom-dev shop, extreme/unspecified travel, no security/eval discipline, high churn.

## 14. Day-of checklist
Sleep; test tech; have water/notes (if allowed); clarify each prompt before diving in; think aloud; manage time (check in); admit unknowns and propose how to find out; keep calm when stuck (go back to examples, simplify); send a concise thank-you note.

## Self-check
1. Perform a 15-minute decomposition on prompt #5 and critique yourself. 2. Debug a 250-line program with 3 planted bugs in 30 minutes. 3. Role-play the security-architect scenario. 4. Present your best project in 10 minutes with metrics. 5. Draft 8 STAR stories. 6. List 8 questions you'd ask the hiring manager.

Next → [[10-Exercises]]
