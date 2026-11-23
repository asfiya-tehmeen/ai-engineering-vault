---
tags: [phase-8, communication, change-management, writing]
---
# 06 — Stakeholders, Communication & Change Management

> Many technically excellent engineers fail as FDEs because the customer never **trusted, understood, or adopted** what they built. Communication is not "soft"; it's the delivery mechanism for engineering value. This note gives concrete techniques.

## 1. Three registers (switch fluently)
| Audience | They want | Format | Don't |
|---|---|---|---|
| **Executives** | outcome, risk, decision needed, timeline, cost | 1 page / 3 slides; headline first; numbers; clear ask | jargon, architecture tours, hedging everything |
| **Technical peers** (IT, data, security, engineers) | design, constraints, trade-offs, interfaces, failure modes | diagrams, ADRs, runbooks, code | hand-waving security, "magic" AI |
| **End users** | "what changes for me, is it reliable, will it make my day easier/harder" | demos on their tasks, short guides, office hours | training on features instead of workflows |
Before any communication, answer: **Who is this for? What decision or action do I want? What do they already know/fear?**

## 2. The Pyramid Principle (answer first)
Lead with the **conclusion/recommendation**, then 2–4 supporting points, then detail on demand. 
> *"Recommendation: proceed to a 6-week production pilot for tier-1 billing tickets. The prototype met our 85% accuracy target (n=150), saved a projected 28% handle time, and cleared security review for read-only access. Risks: KB staleness and agent adoption; both have mitigations. Decision needed by Friday: approve 4 SME hours/week and the IT access request."* 
Then supporting slides/pages. Same structure for emails, Slack updates, and interviews ("Here's my approach; here's why; here's what I'd test first").

## 3. Writing: the FDE's most leveraged skill
- **Weekly update (the backbone):**
  ```
  Status: 🟢/🟡/🔴 (one line why)
  Progress vs plan: shipped / in progress / slipped
  Metrics: quality, adoption, cost (with trend)
  Risks & blockers (with owner/ETA): ...
  Decisions/asks needed: ...
  Next week: ...
  ```
  Consistent format = predictability = trust. Never surprise a sponsor.
- **One-pagers:** problem, options, recommendation, risks, next steps.
- **Design docs/ADRs:** context, decision, alternatives considered, consequences.
- **Meeting notes with decisions & action items** (owner + date) sent within 24h.
- **Runbooks & handoff docs** ([[07-Production-Adoption-and-Handoff]]).
- **Post-mortems:** blameless; timeline, root cause, detection, fixes, prevention.
- **Product feedback:** (see §9).
- **Style:** short sentences, concrete numbers, active voice, no hedging stacks, bullets for scan-ability, define acronyms. Read *On Writing Well*-style guidance; edit ruthlessly.

## 4. Running discovery calls, workshops & demos
- **Agenda with outcomes** ("by the end we will decide X"); invite the right people; send pre-reads; timebox.
- **Facilitation:** invite quiet voices, park tangents, summarize decisions aloud, capture actions live, close with next steps. Use whiteboards/Miro for process mapping and use-case prioritization.
- **Workshop formats:** process-mapping session, **use-case prioritization** (value/feasibility grid), **failure-mode brainstorm** ("how could this go wrong?"), **golden-set labeling session** with SMEs, **acceptance criteria workshop**.
- **Demos:** see [[04-Rapid-Prototyping-and-Delivery]] §6. Rehearse; manage the room; handle "gotcha" questions calmly (acknowledge, note it, follow up).

## 5. Explaining AI accuracy & limitations (a core FDE skill)
- **Replace "accuracy" with the business-relevant statement:** "On 150 recent invoices it correctly extracted every field on 82%; 15% needed one fix; 3% needed manual handling. Each fix takes ~20 seconds vs 4 minutes from scratch."
- **Show the error profile** (what kinds of mistakes, how severe, how detected) and the **safeguards** (citations, validation, human review thresholds).
- **Set expectations on non-determinism:** "It can word things differently each time; we test and monitor consistency; critical facts come from your systems/sources with citations."
- **Don't oversell or undersell:** neither "AI will replace the team" nor "it's just a toy." Use "assist → automate with oversight → autonomous within bounds" ladders.
- **Hallucination framing:** "It can state wrong things confidently when it lacks information; we design so answers must be grounded in your documents, show sources, and say 'not found' when appropriate; here's our measured rate."
- **Anchor on baselines:** human error rates are not zero — compare fairly (and report variance).
- **Make uncertainty actionable:** confidence thresholds route to humans.

## 6. Building trust
- **Reliability beats brilliance:** meet small commitments consistently; communicate early when slipping; own mistakes quickly ("I broke X; here's the fix and prevention").
- **Competence signals:** precise answers, know the details of their domain/process, show you read their documents.
- **Respect expertise:** SMEs and operators know things you don't; ask, credit them publicly, involve them in evaluation.
- **Transparency:** share failure analyses, limitations lists, open risks.
- **Multi-thread relationships:** don't depend on one champion; build ties with users, IT, security, finance.
- **Give before asking:** useful insights from data, quick wins, shortcuts.
- **Be a good guest** (on-site): follow security rules, be punctual, observe etiquette, dress/behave appropriately, learn names.

## 7. Change management & adoption
Software that isn't used creates no value. Adoption is an **engineering + people** problem.
- **ADKAR lens:** **A**wareness (why change?), **D**esire (what's in it for me?), **K**nowledge (how to use it), **A**bility (can I actually do it?), **R**einforcement (sustain). Diagnose *which* step is failing.
- **Design for the workflow:** embed in existing tools (Teams/Slack/CRM sidebar), minimize clicks, keep the human's mental model, provide undo/override.
- **Champions & early adopters:** recruit respected peers; give them early access and a voice; they become trainers.
- **Address fears directly:** job security, deskilling, surveillance, blame for AI errors → clarify role ("assistant, not replacement"), accountability ("you remain the decision-maker"), metrics privacy.
- **Training:** short, task-based, in-context (10-minute sessions, cheat sheets, recorded demos, office hours); teach **how to verify** AI outputs and **when not to use** it.
- **Feedback loops:** in-product thumbs/comments; weekly triage; "you said, we did" updates — users feel heard.
- **Measure adoption:** active users, tasks completed with the tool, repeat usage, acceptance/edit rates, satisfaction; investigate drop-offs.
- **Incentives & policy:** align with managers' goals; update SOPs; remove duplicate old steps.
- **Pace rollout:** pilot group → department → org; gather evidence; communicate wins.
- **Handle resistance:** listen for legitimate concerns (accuracy, workload, trust); turn skeptics into reviewers/testers; show data, not slogans.

## 8. Delivering bad news & handling conflict
**Framework:** *Facts → impact → options → recommendation → ask*.
> *"Accuracy on the new document type is 64%, below our 85% target. Impact: we can't auto-process these yet; the pilot date is at risk by ~2 weeks. Options: (A) human-review all of this type (works now, less savings), (B) 2 more weeks to improve parsing with 3 SME hours, (C) exclude this type from phase 1. I recommend A now + B in parallel; I need SME time Tuesday."*
- **Tell early, in person/live for serious issues, then follow up in writing.** No surprises in steering meetings.
- **Separate problem from person;** stay calm; ask questions to understand the other side's constraints.
- **Difficult stakeholders:** the demanding exec (give crisp status + decisions), the skeptic (invite testing), the gatekeeper (find what would make them comfortable), the scope-creeper (change-request process), the absentee sponsor (escalate via the champion, show impact of delays), the "AI will fix everything" optimist (reality-check with data).
- **Escalation:** document; present options; involve the right level; avoid blindsiding people.
- **Ethical pushback:** if asked to deploy something risky/unethical/illegal (e.g., no human oversight for consequential decisions, privacy violations), **say no clearly, explain risks, propose safer alternatives, escalate internally** (legal/security/leadership). Your credibility depends on this.

## 9. Writing actionable product feedback (FDE → product flywheel)
```
Title: [Specific problem]            Customer(s): ___  Frequency: ___  Severity: ___
Context: workflow/job-to-be-done (1–2 lines)
Problem: what happens today, evidence (trace links, quotes, metrics)
Impact: time/$/risk/adoption; blocked deals/renewals?
Workaround: what we did; cost of workaround; reuse potential
Proposal: suggested solution(s), alternatives, constraints (not just "add feature X")
Priority rationale & urgency: ___
```
Aggregate across engagements: "6 of 9 enterprise deployments needed SharePoint permission-aware retrieval; here's our reference implementation and effort." Product teams respond to **patterns + evidence + proposed solutions**.

## 10. Managing your own workload & well-being
- Protect deep-work blocks; batch meetings; automate repetitive tasks; templates for updates/notes.
- Maintain a **running log** (decisions, learnings, contacts) — invaluable for handoffs and performance reviews.
- Set boundaries on after-hours support; define escalation tiers and who covers.
- Seek **mentors and peer FDEs**; share playbooks; learn from others' post-mortems.
- Watch for **burnout** signals (constant context-switching, travel fatigue); negotiate sustainable patterns.

## 11. Communication anti-patterns
Burying the lead · status updates with no decisions/asks · jargon with execs · over-promising dates · hiding problems until the steering meeting · demo-only metrics · vague "it works pretty well" · long unstructured Slack dumps · blaming the customer or product team · skipping meeting notes · speaking for the customer's SMEs.

## Self-check
1. Rewrite a rambling status as pyramid-principle update. 2. Draft a weekly update for a yellow-status pilot. 3. Explain 82% field accuracy to a COO in three sentences. 4. Plan adoption for 400 support agents using ADKAR. 5. Deliver bad news using facts→impact→options→recommendation→ask. 6. Write a product-feedback memo from a recurring integration pain.

Next → [[07-Production-Adoption-and-Handoff]]
