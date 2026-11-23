---
tags: [phase-8, fde, role]
---
# 01 — The FDE Role & Landscape

## 1. What is a Forward Deployed Engineer?
An engineer who **works directly with customers — often embedded in their environment — to build and ship working solutions on top of a company's product/platform**, then owns the **outcome in production**. Common across sources: *post-sale* (no sales quota), *writes production code*, *customer-facing*, *owns delivery*, and *feeds learnings back to the product*.
One-line contrast (used across sources): a **software engineer** builds a product for many customers; a **solutions architect** designs how one customer should use it; an **FDE** builds and ships the working system for one customer.

### Origin
The term and practice come from **Palantir** ("Forward Deployed Software Engineer"), where engineers sat with customers (government, defense, industrial, healthcare) to deploy the platform against real operational problems — commonly described as the "Delta" (deployment engineer) alongside "Echo" (analyst) archetypes, vs the "Dev" (product engineer). Over the 2020s the model spread to data/AI infrastructure companies and, in the LLM era, to **AI labs and AI-native startups**, because **frontier models are powerful but the last mile (data, workflows, integration, trust, adoption) is bespoke and hard**. Titles you'll see: Forward Deployed Engineer/Software Engineer (FDSE), Forward Deployed AI Engineer, Applied AI Engineer, Solutions Engineer (technical, post-sale), Customer Engineer, Deployment Strategist, Agent/AI Implementation Engineer. **Titles are inconsistent — read the job description, not the title.**

## 2. Why the role exists now (the business logic)
- **Enterprise AI adoption bottleneck:** pilots stall in "pilot purgatory" because of integration, data quality, security reviews, workflow fit, and change management — not model capability alone.
- **Product-market fit discovery:** FDEs reveal what customers *actually* need; patterns across deployments become product features (the **FDE → product feedback loop**).
- **Trading margin for moat (a common framing):** vendors accept lower software-like margins (more services-like effort) early to win strategic accounts, create switching costs, and learn faster. A tension to understand: **FDE work can drift into custom consulting that doesn't scale** — good FDE orgs deliberately **productize repeated patterns**.
- **Trust & accountability:** high-stakes customers want an accountable engineer on the hook.
*(Sources are mostly career/analyst blogs; treat narratives as informed opinion.)*

## 3. FDE vs adjacent roles
| Role | Primary mission | Pre/post-sale | Code? | Customer contact | Success measured by |
|---|---|---|---|---|---|
| **Forward Deployed Engineer** | build & own customer-specific production solution | post-sale (sometimes pilot/pre-sale for strategic) | **heavy, production** | very high (embedded) | outcome in production, adoption, value realized, reusable learnings |
| **Software Engineer (product)** | build the core product for all customers | n/a | heavy | low/indirect | product quality, scale, roadmap |
| **Solutions Engineer / Sales Engineer** | demonstrate fit, scope, support the sale | pre-sale | light–medium (demos, POCs) | high | deals won/pipeline |
| **Solutions Architect** | design scalable implementation plans/reference architectures | pre/post | light–medium | high | sound designs, adoption of best practices |
| **Customer Success / TAM** | relationship, adoption, renewals | post | little | high | retention/expansion/NPS |
| **Professional Services / Consultant** | deliver implementations/transformation | post | varies | high | project delivery, utilization |
| **Applied AI/ML Engineer** | build AI features/models (often internal product) | n/a | heavy | medium | model/system performance |
| **Research Engineer/Scientist** | advance models/methods | n/a | heavy | none | research output |
| **Product Manager** | decide what to build and why | n/a | none | high | product outcomes |
| **Support Engineer** | resolve incidents/tickets | post | medium | high (reactive) | resolution time/CSAT |
FDEs often combine **SWE depth + SE/SA customer skills + consultant problem-framing + PM instinct** — and the ownership is what distinguishes them.

## 4. What FDEs actually do (activities & rhythm)
- **Discover:** interview stakeholders/users, observe workflows, map systems and data, find the real bottleneck.
- **Scope:** convert ambiguity into a bounded pilot with success metrics and risks.
- **Prototype fast:** thin vertical slice on real data in days; demo early and often.
- **Integrate:** SSO, APIs, databases, legacy systems, files, networks, security reviews.
- **Evaluate & harden:** golden sets, guardrails, monitoring, red-teaming ([[../Phase-7-Evals-Observability-LLMOps/00-Phase-7-Index|Phase 7]]).
- **Deploy & operate:** production readiness, runbooks, on-call-ish responsibility during rollout.
- **Enable & hand off:** train users, champions, documentation, knowledge transfer.
- **Feed back:** bug reports, feature requests, patterns → product/engineering; contribute reusable components.
- **Expand:** identify next use cases; support renewals/expansion technically.
**Typical week shape (from "day in the life" accounts):** customer-facing time concentrated in meetings/workshops (often afternoons), deep build time in mornings/evenings, a weekly written update + demo, and a sync with the internal product/engineering team. **Travel/on-site** expectations in postings range from minimal to ~30–60%; frontier-lab roles vary by team. Verify per posting.

## 5. Company types & what the FDE role looks like there
| Type | Examples (illustrative; verify current practice) | FDE flavor |
|---|---|---|
| **Data/ops platform vendors** | Palantir-style platforms, data/analytics platforms | deep on-site deployments; build pipelines/apps/ontologies on the platform; long engagements, regulated customers |
| **Frontier AI labs** | OpenAI, Anthropic, Google DeepMind-adjacent, etc. | strategic enterprise customers; agents/RAG/evals/safety; shape deployment best practices; feedback to research/product; high bar, high variance by team |
| **AI-native application startups** | vertical copilots/agents (legal, finance, health, support, coding) | FDE = early customers' success + product discovery; heavy prototyping; tight loop with founders |
| **Enterprise AI/search platforms** | enterprise search/assistants/agent platforms | connectors, permissions, relevance tuning, rollout |
| **Cloud/data infra** | warehouse, MLOps, observability vendors | reference implementations, migrations, performance, customer architecture |
| **Fintech/vertical SaaS with FDE motions** | spend mgmt, payments, health IT | integrate with customer ERP/EHR; custom workflows |
| **Consultancies/SIs** | big consultancies, boutique AI shops | project-based delivery; more utilization/billing pressure; broader client variety |
| **Government/defense tech** | — | clearances, on-prem/air-gapped, mission contexts |
Choose based on: **customer type, autonomy, product maturity, on-site expectations, comp, mission, learning**.

## 6. Engagement models (how work is structured)
1. **Embedded long-term** (Palantir-style): months/years on-site/remote with a customer team.
2. **Pilot-to-production sprints:** 4–12 week pilots with exit criteria → production rollout.
3. **Hub-and-spoke pods:** a small team (FDE + PM/strategist + data/ML) serving several accounts.
4. **Hybrid pre-/post-sale:** FDE joins late-stage technical validation for strategic deals, then stays for delivery.
5. **Enablement mode:** FDE builds reference implementations and trains the customer's/partner's engineers to own it.
6. **Platform extension:** builds reusable connectors/templates that become product.
Know the model before you accept a role; it shapes workload, travel, and career trajectory.

## 7. The FDE skill matrix
| Domain | What "good" looks like |
|---|---|
| **Engineering** | writes clean, tested, deployable Python/TS (and SQL); debugs unfamiliar codebases quickly; understands APIs, auth, data modeling, cloud, containers |
| **Applied AI** | RAG, agents, evals, prompting, fine-tuning trade-offs, cost/latency, guardrails (Phases 3–7) |
| **Problem framing** | decomposes ambiguity, finds the real bottleneck, quantifies value |
| **Systems & integration** | navigates enterprise stacks, identity, networking, data quality, legacy |
| **Communication** | writes crisp updates; explains to execs and users; runs workshops; handles conflict |
| **Judgment & ownership** | prioritizes ruthlessly, manages risk, says no with data, takes responsibility for outcomes |
| **Product sense** | recognizes reusable patterns; writes actionable feedback to product |
| **Security & compliance literacy** | speaks to CISOs/DPOs credibly ([[../Phase-7-Evals-Observability-LLMOps/08-Governance-Compliance-and-FinOps]]) |
| **Resilience & curiosity** | thrives with unclear requirements, travel, shifting priorities, new domains weekly |
**Who thrives:** people who like **builder + people** work, tolerate ambiguity, get energy from customer impact, and can switch contexts. **Who struggles:** those who need long uninterrupted deep work only, dislike customer-facing conflict, or want pure research/product-scale engineering.

## 8. Pros, cons, and trade-offs (be honest with yourself)
**Pros:** high impact & visibility, rapid learning across industries, business skills, strong career optionality (founder, product, eng leadership, solutions leadership), often strong compensation (varies; higher at frontier labs per career-site claims — verify).
**Cons:** context-switching, customer-driven urgency, travel, ambiguous scope, on-call-like responsibility, "services creep," politics, risk of being the "human API" between customer and product, sometimes less deep-technical-craft time than product engineering.
**Mitigations:** protect build time, set working agreements, productize repeats, document, negotiate scope, build a support network.

## 9. Career paths
FDE → Senior/Staff FDE (own strategic accounts/patterns) → **FDE lead/manager** (build a team, engagement model) → Solutions/Field CTO → Product/Engineering leadership → **Founder** (many founders come from FDE-like roles: you've seen problems up close) → AI consulting/independent. Lateral moves: product management, applied AI, solutions architecture, customer-success engineering.

## 10. Compensation (caveat)
Career-site sources conflict (for example, base figures of roughly the mid-$100Ks to low-$200Ks for typical roles, and much higher total compensation at frontier labs once equity is included, with huge variance). **Don't anchor on blog numbers** — use current offers, levels.fyi-style data, your location, and the company's level bands; consider equity risk, travel, and growth.

## 11. How this vault maps to FDE readiness
| FDE need | Where you learned it |
|---|---|
| Strong software engineering | Phase 1 |
| ML/LLM fundamentals to reason about failures | Phases 2–3 |
| Build LLM features | Phase 4 |
| Knowledge systems | Phase 5 |
| Agents & integrations | Phase 6 |
| Quality, safety, ops, governance | Phase 7 |
| **Customer-facing delivery** | **Phase 8 (this phase)** |

## Self-check
1. Contrast FDE, SE, SA, and consultant in one sentence each. 2. Why does the FDE model exist for AI products? 3. What is the "trading margin for moat" tension and how do good orgs resolve it? 4. List five engagement models and when each fits. 5. Which skills from Phases 1–7 matter most, and what's new in Phase 8? 6. Questions to ask a recruiter to learn the actual job.

Next → [[02-Discovery-and-Problem-Framing]]
