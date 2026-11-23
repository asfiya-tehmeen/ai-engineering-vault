---
tags: [phase-9, career, job-search, portfolio]
---
# 07 — Career Strategy & Job Search

> **Source caveat:** the sources I found are **career blogs, recruiter content, and university/bootcamp pages** (2026). Their consistent themes: **production skills and portfolio matter most; prompting alone is a weakening differentiator; employers want people who can build systems around models (testing, evaluation, monitoring, rollback); entry-level hiring managers reportedly weight portfolio about as much as education; adjacent roles are a common entry path.** Their numbers (e.g., average AI-engineer pay figures, "178% growth in AI integration hiring") are **unverified**. **I found no layoff or labor-market data**, so I make no claims about current market tightness. Combine these notes with your own market research (job boards, recruiters, people in roles).

## 1. Know the role map (and aim deliberately)
| Role | Core | Where this vault fits |
|---|---|---|
| **AI/LLM Application Engineer** | build LLM features/products (RAG, agents, evals) | Phases 1, 4–7 (core) |
| **Forward Deployed / Applied AI Engineer** | customer-embedded delivery | Phases 4–8 |
| **ML Engineer** | train/serve/maintain models and pipelines | Phases 1–3, 7 |
| **ML Platform / MLOps / LLMOps Engineer** | infra, serving, observability, CI/CD | Phases 1, 3, 7 |
| **Research Engineer** | implement/scale experiments for researchers | Phases 2–3, 9 |
| **Applied Scientist / Research Scientist** | new methods; often PhD-heavy | Phases 2–3, 9 (+ deeper theory) |
| **Data Scientist/Analyst (AI-augmented)** | analysis, experimentation, modeling | Phases 1–2, 7 |
| **Solutions Engineer / Architect (AI)** | pre-sales technical, architecture | Phases 4–8 |
| **AI Product Manager / Technical PM** | product strategy with technical depth | Phases 3–8 |
| **AI Safety / Evals / Red-team** | evals, adversarial testing, safety research | Phases 3, 6–7, 9 |
| **Security (AI/LLM)** | AI security engineering | Phases 6–7 |
| **Developer Relations / AI educator** | content, community, demos | all (+ communication) |
Pick **a primary and a secondary target role** for the next 6–12 months; tailor portfolio and outreach accordingly.

## 2. What employers want (synthesis + your edge)
- **Python + SQL + software fundamentals** (tests, APIs, Git, Docker, cloud) — still table stakes (Phase 1).
- **LLM systems skills:** RAG, agents/tool use, structured outputs, caching/cost control, **evals, observability, guardrails, release discipline** (Phases 4–7).
- **Judgment & ownership:** scoping, trade-offs, safety/privacy awareness, working across a whole problem — increasingly expected even of juniors.
- **Communication & customer sense** (Phase 8) — differentiates strongly.
- **Evidence over claims:** deployed projects with **numbers** (accuracy ± CI, latency, cost, failure analysis), public code, write-ups.
- **Learning velocity:** can you ramp on unfamiliar docs/tools? (Interview signals: debugging/learning rounds.)
- **Prompting-only skills** are a weak differentiator — show the *system around the prompt* (versioning, tests, evals, monitoring, rollback).

## 3. Paths by starting point
| You are… | Strengths to leverage | Gaps to close | Suggested route |
|---|---|---|---|
| **Software engineer (2+ yrs)** | engineering, systems | applied AI depth, evals, customer-facing | portfolio of 2–3 production-style AI projects; internal AI projects at current job; target AI app/FDE roles |
| **Data scientist/analyst** | stats, data, domain | production engineering, LLM systems | build RAG/agents with CI/evals; target applied AI/LLM eng; emphasize measurement rigor |
| **Recent grad / early career** | fundamentals, energy | production experience, proof | **portfolio + referrals**; consider adjacent first roles (data/backend/ML-platform/analytics eng) that touch production ML; internships, OSS, hackathons |
| **Career switcher (non-tech)** | domain expertise, communication | engineering fundamentals | go deep on Phase 1–4, build domain-specific projects (your old field is the edge → FDE/solutions/AI-for-X roles) |
| **Researcher/PhD** | theory, experiments | product/engineering hygiene, customer sense | emphasize reproducible code, evals, applied projects; research engineer/applied scientist roles |
| **Founder/freelancer** | shipping, customers | credibility signals | case studies with metrics; consider FDE/solutions roles or consulting |
**Adjacent-role strategy (reported by sources):** data engineering with ML exposure, backend on ML platforms, analytics engineering, QA/automation, DevOps/SRE → internal transfer or next-hop to AI roles. Juniors may start with tasks like **monitoring AI behavior in production, flagging drift, and reviewing AI-generated code** — show you can do these well.

## 4. The portfolio (your strongest asset)
### 4.1 What to build (choose 3 flagship projects)
Pick from this vault's capstones and make them **production-flavored**:
1. **RAG system with measured retrieval + generation evals and permissions** ([[../Phase-5-RAG-Knowledge-Systems/12-Projects-and-Checkpoint]]).
2. **Agent with tools, sandboxing, pass^k evals, and a security threat model** ([[../Phase-6-Agents-Tools-MCP/13-Projects-and-Checkpoint]]).
3. **LLMOps/eval pipeline** (calibrated judge, CI gate, tracing, canary) ([[../Phase-7-Evals-Observability-LLMOps/13-Projects-and-Checkpoint]]).
4. **FDE-style case study** (discovery → pilot → handoff artifacts) ([[../Phase-8-Forward-Deployed-Engineering/13-Projects-and-Checkpoint]]).
5. **Frontier lab** (test-time compute / quantization / interpretability replication) — signals depth ([[12-Projects-and-Final-Checkpoint]]).
Tailor: *AI app roles* → 1–3; *FDE* → 1, 2, 4; *ML/platform* → 3 + serving/fine-tuning; *research eng* → 5 + a paper replication.
### 4.2 What makes a project "hireable"
- **Real problem & users** (even if small), **clear metric**, **baseline**, **results with uncertainty**, **failure analysis**, **cost/latency numbers**, **security/privacy notes**, **README with architecture diagram, how to run, limits, next steps**, **tests/CI**, **deployed demo or screencast**.
- **Honest scope.** One deep project beats five shallow clones. Avoid "yet another chatbot wrapper" without evals.
- **Documentation habit:** ADRs (decision log), eval report, system card, runbook — shows production thinking.
### 4.3 Presentation
- **GitHub:** pinned repos (3–5), profile README (who you are, target role, flagship links), clean commit history, issues/PRs showing process.
- **Personal site/blog:** project pages (problem → approach → results → lessons), write-ups, resume link, contact.
- **Short videos:** 3–5 min demos with metrics.
- **Public writing:** 1 post per month on something you learned (including failures) — compounding credibility.
- **Open-source contributions:** merged PRs to relevant projects.

## 5. Resume & LinkedIn
- **Structure:** headline (target role + 2–3 signals), summary (2–3 lines), **projects section prominent** (esp. early career), experience with **impact bullets** (action + method + metric), skills (grouped), education.
- **Impact bullets formula:** *"Built X using Y that improved Z by N% (measured on M), reducing cost by $K / latency by P ms."* Include eval numbers and scale (docs, users, QPS).
- **Tailor per role:** mirror keywords from the posting *honestly*; reorder to put relevant projects first; one page for <10 years of experience.
- **ATS hygiene:** simple formatting, standard headings, PDF unless told otherwise.
- **LinkedIn:** clear headline, "Open to" signals, featured projects/posts, recommendations; engage thoughtfully.
- **Avoid:** buzzword soup, unverifiable claims, listing every tool, AI-generated fluff; make sure you can defend every line in an interview.

## 6. Networking & outreach (referrals > cold applications)
- **Map targets:** 20–30 companies (AI labs, AI-native startups, enterprise AI platforms, data/cloud vendors, vertical AI companies, consultancies) + 3–5 people at each (engineers, hiring managers, recruiters, alumni).
- **Informational chats (20 min):** ask about the work, team, what they look for, advice; **be specific and respectful of time**; follow up with thanks and what you did with advice.
- **Outreach template (short):**
  > *"Hi [Name] — I'm [role/background] building [relevant project: one line + link]. I admire [specific thing about their work]. I'm exploring [role type] and would value 15 minutes to learn how your team approaches [specific topic]. Happy to share my write-up on [X] if useful. Thanks either way!"*
- **Give value:** share a useful analysis, bug report, PR, or eval relevant to their product; engage with their content thoughtfully.
- **Communities:** meetups, conferences (volunteer/attend), Discords/Slacks, open-source communities, local AI groups, alumni networks, hackathons.
- **Referrals:** ask once you have rapport and a tailored application; make it easy (share resume + 3 bullets on fit).
- **Recruiters:** be clear on target role, location/visa, timeline, comp expectations (after they share the band).

## 7. Application strategy
- **Quality over quantity:** aim for ~5–10 tailored applications/week with referrals where possible; track in a spreadsheet (company, role, stage, contacts, dates, notes).
- **Funnel metrics:** applications → recruiter screens → onsites → offers; diagnose the weakest stage and fix it (resume? interview skills? targeting?).
- **Timing:** apply early after postings open; start interviewing at 2–3 companies in parallel to create comparison/leverage; plan interview prep blocks.
- **Different company types:** AI labs (very selective, project/values depth, thin public loop info), AI-native startups (speed, ownership, broad scope), big tech (structured loops), enterprise AI vendors (customer skills), consultancies/SIs (breadth, utilization), government/defense (clearances).
- **Location/visa/remote:** clarify early; some customer-facing roles need on-site/travel.
- **Alternative routes:** contract/freelance gigs to build experience, internships/apprenticeships, fellowships/residencies, open-source sponsorship, startup founding-engineer roles.

## 8. Interview readiness (integrate prior phases)
- **AI engineer loops:** Python/SQL coding, system design (RAG/agents/evals/cost/security), project deep dive, ML fundamentals, behavioral — use Q&A notes from Phases 1–7.
- **FDE loops:** add decomposition, debugging, customer role-plays ([[../Phase-8-Forward-Deployed-Engineering/09-FDE-Interview-Prep]]).
- **Research-eng loops:** coding + ML implementation (attention, training loops), paper discussion, experiment design.
- **Practice cadence:** 2–3 mock rounds/week for 4–6 weeks; record; targeted drills on weaknesses.
- **Take-homes:** timebox; README; evals; honest limitations.
- **AI tools in interviews/applications:** follow each company's policy (some prohibit AI in interviews); use AI to *practice and critique*, not to fake skill; never misrepresent.

## 9. Offers, negotiation & decision-making
- **Know the components:** base, bonus, equity (strike/valuation/vesting/refreshers), signing bonus, benefits, relocation, remote/hybrid, travel expectations, level/title.
- **Research comp** from current sources (levels-type sites, peers, recruiters); **ignore blog averages** (they conflict).
- **Negotiate calmly with evidence:** competing offers/timelines, scope and level, specific asks (e.g., level, equity, signing, start date). Ask for time to decide; get terms in writing.
- **Decision framework (score 1–5):** learning rate, mentorship/manager quality, impact & ownership, mission/ethics fit, company stability/growth, comp, work-life/travel, location/visa, resume signal, optionality.
- **Evaluate the team, not just the brand:** talk to engineers; ask about on-call, evals discipline, product-feedback loops, churn.
- **Red flags:** vague role, no eval/security culture, hero-mode ops, high turnover, constant re-orgs, FDEs as pure custom-dev with no product feedback.

## 10. First 90 days (once you land)
- **0–30:** learn product/customers/codebase; ship a small fix; meet stakeholders; set up your learning log; ask for explicit expectations.
- **31–60:** own a small project end-to-end; add an eval/monitoring improvement; document; seek feedback.
- **61–90:** deliver a measurable win; propose one reusable improvement; present learnings; plan growth goals with your manager.
- **Habits:** over-communicate, write things down, keep a "brag doc" (impact log), build relationships across product/security/support.

## 11. Long-term career strategy
- **T-shaped growth:** depth in one area (e.g., evals/LLMOps, agents, RAG, inference, security) + breadth across the stack and business.
- **Durable skills outlast tools:** software engineering, measurement/evals, systems thinking, communication, security mindset, learning agility.
- **Tool churn:** expect frameworks/models/APIs to change quarterly; keep **portable assets** (eval suites, prompts-as-files, architecture patterns).
- **Compounding assets:** public portfolio, relationships, writing, reusable libraries, domain expertise.
- **Opportunities:** staff/principal IC, engineering management, solutions/field CTO, product, founder/consultant, research. Revisit goals every 6 months.
- **Ethics & responsibility:** you'll make deployment decisions with real consequences — build a habit of saying "not like this, but like that" with alternatives.
- **Well-being:** sustainable pace, boundaries, community; avoid burnout from constant FOMO.

## 12. Job-search sprint plan (12 weeks)
| Weeks | Actions |
|---|---|
| 1–2 | Choose target roles; audit portfolio gaps; polish 3 flagship READMEs with metrics; draft resume/LinkedIn/GitHub profile |
| 3–4 | Build target list (30 companies); 10 informational chats; publish one write-up |
| 5–6 | Start applications (referral-first); weekly mock interviews; fix weak funnel stages |
| 7–10 | Interview loops; keep applying; refine stories; negotiate prep |
| 11–12 | Offers/decisions; thank-yous; plan first 90 days; document learnings for the next cycle |
**Weekly:** 10 outreach touches, 5 tailored applications, 3 mock rounds, 5 hours building/writing, 1 review of funnel metrics.

## Self-check
1. Choose a primary and secondary target role and justify with your background. 2. Define "hireable project" criteria and score your best project. 3. Write a resume bullet with method + metric + scale. 4. Draft an outreach message and a follow-up. 5. List the stages of your job-search funnel and how you'll diagnose drop-off. 6. Create a decision matrix for two hypothetical offers.

Next → [[08-Capstone-and-12-Month-Plan]]
