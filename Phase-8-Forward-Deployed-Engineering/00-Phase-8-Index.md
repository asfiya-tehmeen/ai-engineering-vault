---
tags: [phase-8, index, fde]
---
# Phase 8 — Forward Deployed Engineering

> [!abstract] Why this phase exists
> Phases 1–7 gave you the **technical trunk** shared by every AI engineer. The **Forward Deployed Engineer (FDE)** adds a second trunk: **turning ambiguous customer problems into working, adopted, production systems** inside messy real-world organizations. The technical bar is "strong software + applied AI engineer"; the differentiators are **discovery, scoping, communication, integration, delivery under uncertainty, and ownership of the outcome**. This phase teaches that trunk — and how to get hired for it.

**Duration:** 4–6 weeks of study + ongoing practice (this is a *craft*; you only get good by doing engagements, even simulated ones) · **Prereq:** Phases 1–7 (especially 4–7).

> [!warning] Source quality — read this
> The FDE role is new enough that most public material is **career-blog and recruiter content**. My research for this phase found: (a) the **Palantir loop is the best documented** (decomposition + re-engineering rounds, reported by candidates and prep sites); (b) **OpenAI/Anthropic loops are thinly sourced** and vary by team; (c) **salary numbers conflict** between sources (I give no figures beyond "wide range, higher at frontier labs"); (d) I found **no named-company job postings** in my searches, only smaller-company and staffing postings; (e) claims like "FDE postings grew 729% YoY" come from a career-content site — treat as directional. The **frameworks in this phase** (discovery, scoping, integration, comms, handoff) are synthesized from consulting/product/engineering practice and the common role shape across sources, not from any one company's playbook. **Always check each company's own careers page and interview guidance.**

## Topic map
| # | Note | Core ideas |
|---|---|---|
| 01 | [[01-The-FDE-Role-and-Landscape]] | what FDEs are, origin, FDE vs adjacent roles, company types, engagement models, skills, career paths |
| 02 | [[02-Discovery-and-Problem-Framing]] | stakeholder mapping, discovery interviews, decomposition, feasibility triage, success metrics, discovery artifacts |
| 03 | [[03-Scoping-Planning-and-Commercials]] | MVP/pilot design, success criteria, estimation under uncertainty, ROI models, scope control, risk registers |
| 04 | [[04-Rapid-Prototyping-and-Delivery]] | thin vertical slice, demo-driven development, eval-first prototyping, messy data, productizing patterns |
| 05 | [[05-Enterprise-Integration-and-Data]] | systems landscape, integration patterns, SSO/SCIM, networking, security reviews, getting access, on-prem |
| 06 | [[06-Stakeholders-Communication-and-Change]] | exec/technical comms, writing, demos, workshops, change management, bad news, trust |
| 07 | [[07-Production-Adoption-and-Handoff]] | pilot→production, runbooks, support models, adoption, handoff, feedback to product, expansion |
| 08 | [[08-Case-Studies-and-Playbooks]] | 10 worked engagement cases + failure post-mortems |
| 09 | [[09-FDE-Interview-Prep]] | loop structure, decomposition & re-engineering rounds, customer role-plays, behavioral stories, negotiation |
| 10 | [[10-Exercises]] | 90+ exercises (role-plays, writing, integration, cases) |
| 11 | [[11-QA-Interview-Questions]] | 60 Q&A |
| 12 | [[12-Resources-and-Practice-Links]] | guides, books, tools, practice |
| 13 | [[13-Projects-and-Checkpoint]] | 6 projects + capstone simulated engagement + self-assessment |

## Weekly plan (6 weeks)
| Week | Focus | Output |
|---|---|---|
| 1 | Notes 01, 02 | discovery interview guide + a real discovery call with a local business/nonprofit (or a peer role-play) → discovery report |
| 2 | Note 03 | scoping doc, ROI model, risk register for that problem |
| 3 | Notes 04, 05 | thin-slice prototype + integration against a mock enterprise sandbox (SSO, CRM, ITSM) |
| 4 | Note 06 | weekly update, exec readout deck, demo script; record and review yourself |
| 5 | Notes 07, 08 | production-readiness pack, runbook, handoff; write 2 case studies |
| 6 | Note 09 + capstone | mock-interview loop (decomposition, re-engineering, role-play) + [[13-Projects-and-Checkpoint\|capstone engagement]] |

## Completion checklist
- [ ] Explain the FDE role, how it differs from SWE/SE/SA/consultant/TAM, and where your strengths fit
- [ ] Run a structured discovery interview (non-leading questions) and produce a discovery report with a decomposed problem, metrics baseline, and feasibility triage
- [ ] Write a one-page scope (problem, users, success metrics, MVP, non-goals, risks, timeline) and defend it
- [ ] Build an ROI model with explicit assumptions and sensitivity analysis
- [ ] Ship a thin vertical slice in ≤2 weeks with an eval harness and a demo that uses the customer's own data
- [ ] Integrate with a mock enterprise stack (OIDC/SAML SSO, SCIM, REST/SOAP, file drop, CRM/ITSM), and answer a security questionnaire
- [ ] Communicate in three registers: executive (outcome/risk/ask), technical (design/trade-offs), end-user (how it helps me)
- [ ] Deliver bad news and manage expectations on AI accuracy with numbers
- [ ] Produce a production-readiness pack: SLOs, monitoring, runbook, rollback, training, handoff
- [ ] Complete a decomposition case, a re-engineering/debugging drill, a system design, and a customer role-play at interview quality
- [ ] Capstone simulated engagement delivered with all artifacts

Back to [[../00-Roadmap]].
