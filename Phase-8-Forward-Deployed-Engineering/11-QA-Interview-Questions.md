---
tags: [phase-8, qa, interview]
---
# 11 — Q&A (Phase 8)

> Answer aloud first (timebox 60–120 seconds), then expand. For scenario questions, practice with **structure** (clarify → assumptions → plan → metrics → risks → next step).

## Role & landscape
> [!question]- Q1. What is a Forward Deployed Engineer?
> An engineer who works directly with customers — often embedded in their environment — to build and ship working solutions on a company's product/platform, owning the outcome in production, and feeding learnings back to the product. Post-sale, hands-on code, customer-facing, accountable for results.

> [!question]- Q2. How does an FDE differ from a software engineer, solutions engineer, and solutions architect?
> A product SWE builds for many customers; a solutions engineer demonstrates fit and supports the sale (pre-sale, lighter code); a solutions architect designs how a customer should adopt the product; an FDE builds and ships the working system for one customer and owns the outcome.

> [!question]- Q3. Why do AI companies need FDEs?
> Models are powerful but the last mile — data access, integration, workflows, security, trust, adoption — is bespoke and hard. FDEs close the gap, reveal real needs, and turn repeated patterns into product features.

> [!question]- Q4. What's the "services trap" and how do good FDE orgs avoid it?
> Bespoke custom work that never converges into reusable product. Avoid via productizing repeated needs, measuring reuse, change-request discipline, feedback to product, and shared libraries/connectors.

> [!question]- Q5. What does an FDE's week look like?
> Split between customer-facing time (discovery, workshops, demos, steering) and build time (prototyping, integration, evals), with a weekly written update, product feedback to internal teams, and ongoing risk/access management.

> [!question]- Q6. Who thrives as an FDE?
> People who enjoy building and people work, tolerate ambiguity and context-switching, take ownership, communicate clearly, and learn new domains fast.

## Discovery & framing
> [!question]- Q7. A customer says "we want an AI assistant." What do you do first?
> Don't start building. Clarify the problem: who the users are, what workflow/pain, volume/cost, success metric and baseline, systems/data, constraints, previous attempts. Shadow real work, read real cases, then propose options and a thin first slice.

> [!question]- Q8. How do you run a good discovery interview?
> Ask about specific past behavior, avoid leading/hypothetical questions and pitching, dig into time/cost/exceptions/workarounds, observe real work, collect artifacts, and ask who else to talk to. Separate observations from interpretations.

> [!question]- Q9. Walk me through decomposing a vague problem.
> Clarify goal/users/constraints/metric; state assumptions; map actors, inputs/outputs, systems; split into components (ingest → classify → extract → decide → act → monitor); evaluate options per component; prioritize by value/feasibility/risk; choose a thin slice that tests the riskiest assumption; define metrics, risks, and the first-week plan.

> [!question]- Q10. How do you decide whether AI is the right tool?
> Check if the task is language/unstructured-data heavy, variable, tolerant of errors with oversight, has obtainable ground truth, accessible data, enough volume, and compatible latency/cost. If logic is deterministic or the real bottleneck is process/data access, use code/SQL/process changes instead.

> [!question]- Q11. Which use case do you start with?
> High business value, high feasibility, low–medium risk, strong champion, clear metric and data access — often an assistive (human-in-the-loop) workflow rather than the most glamorous or risky one.

> [!question]- Q12. How do you handle a skeptical stakeholder?
> Listen for the legitimate concern (past failure, accuracy, job fears), turn it into a test or evaluation criterion, involve them as a reviewer, show data and failure analysis, and keep promises small.

> [!question]- Q13. What's your riskiest-assumption-first approach?
> List assumptions (data access, accuracy, adoption, security approval, latency), rank by impact × uncertainty, and design the cheapest test that could falsify the top one — e.g., run on 50 real documents this week or request data access today.

## Scoping & planning
> [!question]- Q14. What goes in a pilot scope?
> Problem/users/baseline, success metrics + guardrails, MVP scope and non-goals, approach, evaluation plan, timeline/gates, customer dependencies, risks, security/compliance needs, and the production path.

> [!question]- Q15. How do you define pilot success?
> Quantitative targets on a frozen golden set (accuracy, straight-through rate, latency, cost, zero violations), qualitative user feedback, security sign-off, with go/iterate/stop gates agreed beforehand and a plan for missed targets.

> [!question]- Q16. How do you estimate when everything is uncertain?
> Decompose, use ranges and confidence levels, start calendar-bound items (access, security review) on day 1, run spikes to retire risk, add buffers for approvals/data cleaning, and communicate milestones with risks.

> [!question]- Q17. How do you build an ROI case?
> Value = time saved × volume × loaded cost + error reduction + risk/revenue effects, minus build/run/review costs, with explicit assumptions, an adoption ramp, conservative/base/optimistic scenarios, and sensitivity analysis — then replace assumptions with pilot measurements.

> [!question]- Q18. The sponsor keeps adding scope. What do you do?
> Acknowledge, log a change request with effort/timeline/risk impact, tie it to the success metric, offer options (phase 2, trade-off, move date), and escalate with options if needed — protecting evals/security/production readiness.

> [!question]- Q19. How do you avoid pilot purgatory?
> Define production criteria and run-cost ownership up front, start security early, measure baseline and value, plan adoption, name a customer-side owner, build for hardening from day one, and get budget alignment before the pilot ends.

## Delivery & prototyping
> [!question]- Q20. What do you build in the first week after getting real data?
> A walking skeleton: ingest → process → output → measure, on 30–50 real cases with an initial golden set; show a demo with metrics and failure examples, then iterate on the top failure category.

> [!question]- Q21. How do you stay fast without creating a mess?
> Classify code (spike/prototype/production); harden early what's expensive to retrofit — secrets, auth, logging/tracing, evals, config, idempotency, dependency pinning; keep a decision log and clean repo.

> [!question]- Q22. How do you make demos effective?
> Use the customer's data and language, script a happy path plus a realistic failure path, show metrics, let users try it, capture reactions, end with a clear ask, and avoid promising features on stage.

> [!question]- Q23. How do you handle messy data?
> Get real samples early, read records yourself, build quality reports, fix parsing first, create a golden set with SMEs, version snapshots, and handle sensitivity in approved environments.

> [!question]- Q24. When do you productize?
> When a need repeats (≥3 engagements), generalizes, fits the product strategy, and is maintainable; otherwise keep it customer-specific or partner-delivered. Write evidence-based feedback to product.

## Integration & enterprise
> [!question]- Q25. What are the usual blockers in enterprise deployments?
> Data/system access and approvals, security questionnaires and architecture reviews, identity/SSO and permissions, network restrictions (firewalls/proxies/VPC), data quality, change control, and procurement.

> [!question]- Q26. SAML vs OIDC vs SCIM?
> SAML and OIDC authenticate users via an IdP (SAML older XML-based enterprise SSO; OIDC modern OAuth-based). SCIM provisions/deprovisions users and groups so access follows HR/IdP changes.

> [!question]- Q27. How do you enforce the customer's document permissions in a RAG system?
> Capture source ACLs at ingestion, sync permission changes, filter retrieval by the requesting user's identity/groups, default-deny, include ACL scope in cache keys, test for leakage, and audit access.

> [!question]- Q28. The customer says data can't leave their environment. Options?
> Deploy in their VPC/tenant using private endpoints to cloud-hosted models, or self-host open-weight models on-prem (even air-gapped), with a data-flow diagram, controls, offline artifact/update procedures, and honest discussion of capability trade-offs.

> [!question]- Q29. How do you integrate with a legacy system with no API?
> Prefer batch file drops or database views/CDC (read-only), message queues or middleware if available; use RPA/UI automation only as a last resort with verification; add schema validation, idempotency, and monitoring.

> [!question]- Q30. How do you speed up a security review?
> Start day 1, provide an evidence pack (architecture/data-flow, data classification, retention/deletion, access control, logging, subprocessors/provider terms, red-team results, attestations), offer deployment options, engage a security champion, and answer precisely and consistently.

> [!question]- Q31. Common LLM-specific security questions and answers?
> Where does data go/is it trained on/how long retained (provider terms, ZDR/private options); how do we prevent prompt injection/data leakage (layered controls, least privilege, tests); how are permissions enforced (retrieval/tool-level authz); audit trail (traces/logs); PII handling (redaction/retention); provider outage/model change (fallbacks, version pinning, evals).

> [!question]- Q32. How do you work when access is slow?
> Track requests with owners/dates, parallelize with samples/mocks, escalate with specifics and business impact, reduce the ask (read-only, narrower scope, sandbox), propose alternatives, and document approvals.

## Communication & change
> [!question]- Q33. How do you explain AI accuracy to an executive?
> Translate to business terms with measured results: "On 150 recent cases, 82% fully correct, 15% needed one quick fix, 3% manual; each fix takes 20 seconds vs 4 minutes; here are the safeguards and monitoring." Compare to human baseline and be candid about limits.

> [!question]- Q34. How do you explain hallucinations to non-technical users?
> "It can sound confident while wrong, especially without grounding. We design it to answer only from your documents, show sources, say 'not found' when unsure, and we measure and monitor how often it errs."

> [!question]- Q35. Structure of a good weekly update?
> Status (R/A/G with one-line reason), progress vs plan, metrics (quality/adoption/cost), risks/blockers with owners/ETAs, decisions/asks needed, and next week — consistent format, no surprises.

> [!question]- Q36. How do you deliver bad news?
> Tell early and directly; present facts, impact, options, a recommendation, and the ask; follow up in writing; own your part; keep tone solution-focused.

> [!question]- Q37. How do you drive adoption?
> Embed in existing workflow, recruit champions, address fears honestly, provide short task-based training, capture and act on feedback ("you said, we did"), measure usage and acceptance, and pace rollout.

> [!question]- Q38. How do you build trust quickly?
> Be reliable on small commitments, know their domain details, respect SME expertise, share failures and limitations transparently, communicate early, and multi-thread relationships.

> [!question]- Q39. A customer asks you to deploy something risky or unethical. What do you do?
> Say no clearly, explain concrete risks (legal, safety, privacy, harm), propose safer alternatives (human oversight, narrower scope, additional controls), escalate internally to legal/security/leadership, and document.

## Production & handoff
> [!question]- Q40. What's in a production readiness review?
> Quality vs targets with regression suite; SLOs and load tests; security review, authN/Z, data handling, guardrails; observability, runbooks, release/rollback; training, support model, ownership and run-cost budget; governance documentation.

> [!question]- Q41. What goes in a handoff package?
> Architecture and data flows, operations runbook, evaluation guide and latest report, configuration reference, security/compliance dossier, user guide and training, monitoring guide, known limitations and roadmap, and contacts/escalation — plus pair-build/reverse-shadowing and a 30/60/90 support plan.

> [!question]- Q42. How do you measure value realized?
> Re-measure against the baseline at 30/90/180 days (primary and guardrail metrics, adoption, cost per task, satisfaction), attribute impact (A/B or controlled pre/post), compare to projected ROI, and report lessons and next opportunities.

> [!question]- Q43. How do you end an engagement well — including a failed one?
> Run a retro, document learnings, hand over or archive per policy, revoke access, rotate secrets, confirm deletion, and give an honest recommendation; stopping a non-viable project cleanly builds trust.

## Technical scenarios
> [!question]- Q44. Design a support copilot for 400 agents with CRM and KB.
> Embed in the agent's tool; read-only CRM API + RAG over current KB with hybrid retrieval, reranker, metadata for status/effective date, citations and abstention; drafts only; golden set with SMEs; calibrated judge + A/B on handle time; permission-aware access, audit logging, monitoring, training/champions; phased rollout and handoff.

> [!question]- Q45. Design an invoice-processing pipeline for a bank.
> Classify doc type → type-specific schema extraction with evidence quotes → validation (totals, vendor/master data, dates) → confidence via disagreement → human review queue with highlights → idempotent write to ERP; VPC/private model endpoints, PII redaction, audit logs; field-level evals by slice; STP rate and cost metrics; rollout and ops.

> [!question]- Q46. The assistant answered with an HR document the user shouldn't see. Respond.
> Contain (disable/limit feature, purge caches), investigate (ACL sync, retrieval filtering, cache key scope), fix with permission-aware retrieval and tests, run an adversarial audit, notify per policy, and write a post-mortem with prevention (leakage tests in CI, audit logging).

> [!question]- Q47. Pilot results: 80% accuracy vs 90% target; deadline in two weeks. What do you do?
> Analyze failures by slice, identify top categories, propose quick wins (parsing, retrieval, prompts, validation) and options: human review on the weak slice, narrowed scope, or a short extension; present facts→impact→options→recommendation with data and get a decision.

> [!question]- Q48. How would you reduce an LLM feature's cost by 50% for a customer?
> Attribute cost; apply caching, trim context/tool outputs, shorten outputs, batch offline work, route easy cases to smaller models/lower effort validated by evals, consider distillation/self-hosting if utilization justifies; track cost per successful task and alert on anomalies.

> [!question]- Q49. Debug approach for an unfamiliar codebase failing in production?
> Reproduce, read entry points/tests and logs/traces, form hypotheses, bisect with debugger/prints, check boundary conditions/state/time/encoding, fix minimally with a regression test, communicate findings, and add monitoring.

> [!question]- Q50. A customer wants "an agent that does everything." Response?
> Reframe to a specific workflow with measurable outcomes, start with assistive/propose-then-approve patterns, define autonomy levels and risk tiers, pilot on a thin slice with evals and guardrails, and expand autonomy as evidence supports.

## Behavioral & interview
> [!question]- Q51. Tell me about a time you handled ambiguity.
> Use STAR: situation with unclear requirements; how you clarified, decomposed, tested the riskiest assumption; the measurable result; what you learned. Show structure and bias to action.

> [!question]- Q52. Tell me about a conflict with a stakeholder.
> Describe the differing goals, how you listened, found shared interests, used data/options, reached a decision, and preserved the relationship; highlight what you'd do differently.

> [!question]- Q53. Describe a failure.
> Choose a real failure with consequences; own your part; explain root cause, immediate recovery, and systemic prevention; show changed behavior.

> [!question]- Q54. Why FDE rather than product engineering?
> Reflect genuinely: energy from customer impact, breadth across domains, ownership of outcomes, closeness to real problems; acknowledge trade-offs (context switching, travel) and how you manage them.

> [!question]- Q55. Why this company?
> Connect mission, product maturity, customer types, engagement model, learning opportunities, and your specific strengths; reference concrete things you've studied.

> [!question]- Q56. What would you do in your first 90 days?
> Learn product and customers deeply, shadow senior FDEs, ship a small production contribution, build relationships with product/security/support, document playbooks, deliver a measurable win on an engagement, and propose one reusable improvement.

> [!question]- Q57. How do you prioritize across multiple customers?
> Use impact, urgency, strategic value, and risk; communicate trade-offs transparently, protect build time, escalate resourcing conflicts early, and standardize repeated work.

> [!question]- Q58. How do you stay current with AI?
> Hands-on projects, reading primary docs and papers weekly, following practitioner blogs, running small evals on new models/tools, and sharing learnings internally.

> [!question]- Q59. How do you decide when to escalate?
> When risk to safety/security/legal exposure exists, when scope/dates are threatened beyond my authority, when blockers need executive action, or when ethical concerns arise — with facts, options, and a recommendation.

> [!question]- Q60. What questions do you ask us?
> Engagement model and travel, FDE→product feedback loop, reuse vs bespoke ratio, eval/guardrail practices, deployment environments, success metrics at 6/12 months, growth paths, team size/turnover, and how scope/commercial expectations are managed.

Next → [[12-Resources-and-Practice-Links]]
