---
tags: [phase-8, production, handoff, adoption]
---
# 07 — Production, Adoption & Handoff

> A successful pilot is not the finish line — **production, adoption, and sustainable ownership** are. This is where many FDE engagements stall ("pilot purgatory") or quietly fail (shelfware). This note is the playbook for crossing that gap and leaving the customer (and your own product team) better off.

## 1. Why pilots die (and what to do about each)
| Cause | Symptom | Countermeasure |
|---|---|---|
| **No production path planned** | great demo, no budget/owner | write production criteria & cost *before* the pilot; get sponsor sign-off on the path |
| **No owner** | "who runs this?" | name a customer-side product/engineering owner early; RACI; shadow period |
| **Security/compliance late** | blocked at final review | start day 1; evidence pack; staged approvals |
| **Value unproven or unmeasured** | "nice, but so what?" | baseline + pilot metrics + ROI with measured inputs |
| **Poor adoption** | users revert to old way | workflow embedding, champions, training, feedback loops |
| **Fragile prototype** | breaks under real load/data | harden: tests, monitoring, error handling, scaling, CI/CD |
| **Cost surprise** | production bill ≫ pilot | cost model with volume forecast; caching/routing; budgets/alerts |
| **Integration debt** | manual steps hidden | automate pipelines; document; idempotent jobs |
| **Champion leaves / priorities shift** | project orphaned | multi-thread stakeholders; tie to strategic KPIs; exec sponsor |
| **Vendor/model change** | behavior shifts | version pinning, eval gates, migration plan ([[../Phase-7-Evals-Observability-LLMOps/04-Release-Management-and-Experimentation]]) |

## 2. Production readiness review (PRR) — the checklist
**Functionality & quality**
- [ ] Meets agreed success metrics on frozen test set (CIs, slices); known limitations documented
- [ ] Regression suite + CI gate; sentinel/must-never-fail cases pass
- [ ] Human-in-the-loop & escalation paths implemented and tested
**Reliability & performance**
- [ ] SLOs defined (availability, latency, quality) with error budgets
- [ ] Load tested at expected + peak volume; capacity plan; rate-limit handling; fallbacks tested
- [ ] Failure modes tested (provider outage, timeouts, bad inputs, dependency failures); graceful degradation
**Security & compliance**
- [ ] Security review/penetration test findings addressed; threat model + red-team report
- [ ] AuthN/Z (SSO/RBAC), least privilege, secrets management, network controls
- [ ] Data classification, retention, deletion, residency; DPA/BAA as needed; audit logging
- [ ] Guardrails measured (precision/recall, false-block rate)
**Observability & operations**
- [ ] Tracing/logging/metrics/dashboards; alerts routed to named owners; cost attribution
- [ ] Runbooks (common incidents, rollback, key rotation, reindex, model change), on-call/escalation
- [ ] Release process (versioned prompts/config, canary/rollback), change-control scheduled
**People & process**
- [ ] User training, documentation, support channel & SLAs; champions identified
- [ ] Feedback capture and triage; adoption metrics dashboard
- [ ] Ownership & RACI signed; budget approved for run costs
- [ ] Communication plan for rollout; success criteria for each phase
**Governance**
- [ ] System card, risk assessment, human-oversight design, vendor assessments ([[../Phase-7-Evals-Observability-LLMOps/08-Governance-Compliance-and-FinOps]])

## 3. Rollout strategy
- **Phased:** pilot cohort → one department → region → company; each phase with exit criteria and monitoring.
- **Shadow → assist → automate:** start in suggestion mode; measure acceptance; progressively automate low-risk subsets with oversight.
- **Feature flags & kill switch;** staged traffic (see canary patterns).
- **Hypercare period** (2–4 weeks post-go-live): daily check-ins, rapid fixes, extra monitoring, office hours.
- **Communication:** launch note, FAQ, "what to do when it's wrong," where to get help, how to give feedback.
- **Measure and publish early wins** (with honesty about limits).

## 4. Operating model & support
- **Tiers:** L1 (customer help desk: access, how-to), L2 (customer app/ops team: config, data issues, known playbooks), L3 (vendor engineering/FDE: defects, model/prompt changes, deep debugging).
- **Define:** hours of coverage, severity levels and response targets, escalation paths, communication channels, maintenance windows, **who can change prompts/models/guardrails and via what process**.
- **Cost ownership:** who pays for inference/infra/seats; budgets and alerts; chargeback.
- **Incident management:** playbooks for quality regressions, outages, security events; post-mortems shared.
- **Ongoing improvement:** monthly review of metrics, failure taxonomy, new use cases; quarterly red-team & governance review.

## 5. Documentation & knowledge transfer
Deliver a **handoff package**:
1. **Architecture overview** (diagram, components, data flows, trust boundaries).
2. **Operations runbook** (start/stop, deploy/rollback, common incidents, key rotation, reindexing, model/prompt change procedure, backups/DR).
3. **Evaluation guide** (datasets, graders, how to run, thresholds, how to add cases) + latest report.
4. **Configuration reference** (environment variables, routes, flags, prompts, models, limits).
5. **Security & compliance dossier** (threat model, red-team results, data-handling, approvals, system card).
6. **User guide & training materials** (task-based).
7. **Monitoring guide** (dashboards, alerts, what "normal" looks like).
8. **Known limitations & roadmap** (honest).
9. **Contact & escalation matrix.**
**Transfer methods:** pair-build with customer engineers during the last third of the engagement; **reverse shadowing** (they drive, you watch); recorded walkthroughs; game-day incident drills; a **30/60/90-day support plan**; a sign-off checklist.
**Goal:** the customer can operate and evolve it **without you** (or knows exactly when to call you).

## 6. Measuring value realized (close the loop)
- Re-measure against **baseline** at 30/90/180 days: primary metric, guardrails, adoption, cost per task, user satisfaction.
- **Attribution:** isolate the system's effect (A/B or pre/post with controls; beware seasonality).
- **Value report:** realized vs projected ROI with explanations; **lessons learned**; next opportunities.
- **Quarterly business review (QBR) inputs:** usage trends, quality trends, incidents, roadmap, expansion candidates.
- Share **customer-approved stories** (with permission) — they fuel expansion and your company's case studies.

## 7. Expansion & the land-and-expand motion
- **Look for adjacent workflows** (same data/integration, new use case), **adjacent teams** (same pain), and **depth** (more automation within the workflow).
- **Reuse assets:** connectors, eval harness, guardrails → faster time-to-value on use case #2.
- **Champion network:** have the first team pitch the next.
- **Keep discipline:** each expansion goes through discovery → scope → gates; don't let success inflate scope.
- **Technical enablers:** platform patterns (shared gateway, shared eval service, shared knowledge index) to avoid one-off silos.

## 8. Feedback to product & productization decisions
- Capture **patterns** across engagements; quantify (frequency, ARR impact, time saved by a feature).
- **Decide what to productize:** repeated ≥3 times, generalizable, aligned with product strategy, maintainable. Otherwise keep as customer-specific or partner-delivered.
- **Contribution paths:** internal libraries/templates, upstream PRs, reference architectures, docs/cookbooks, field notes.
- **Be a good product partner:** bring evidence, propose designs, test early builds in customer environments, celebrate shipped features that came from the field.

## 9. Handling the end of an engagement (transitions)
- **Planned transition:** shadow period, runbook walk-throughs, SLA to the support org, success criteria met → "graduate" the account.
- **Pivot or stop:** if gates fail, **stop cleanly** — write an honest retrospective, preserve learnings, archive code/data per policy, secure credentials revocation, and recommend alternatives. Stopping a bad project well builds long-term trust.
- **Offboarding hygiene:** revoke access, rotate secrets, remove temp data, confirm deletion per DPA, hand over documentation, final stakeholder thank-you & summary.

## 10. Retrospectives & personal learning
Run a blameless **engagement retro** (what went well, what didn't, what we'd change, what to productize); update your **playbooks/templates**; log actuals vs estimates; add stories to your **portfolio** (sanitized; permission) — see [[09-FDE-Interview-Prep]].

## 11. Anti-patterns
Declaring victory at demo · "throw over the wall" handoff · no run-cost owner · hidden manual steps · tribal knowledge · adoption ignored · hypercare absent · expansion without gates · no stop criteria · poor offboarding (lingering access) · customer-specific hacks that never reach product · over-dependence on the FDE ("hero mode").

## Self-check
1. List five reasons pilots die and the countermeasure for each. 2. Write a production readiness checklist from memory (≥20 items). 3. Define L1/L2/L3 for a copilot deployment. 4. Compose the handoff package contents. 5. How do you measure value realized at 90 days? 6. When do you productize vs keep custom?

Next → [[08-Case-Studies-and-Playbooks]]
