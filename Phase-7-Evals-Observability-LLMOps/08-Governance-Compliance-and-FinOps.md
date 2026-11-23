---
tags: [phase-7, governance, compliance, finops, eu-ai-act]
---
# 08 — Governance, Compliance & FinOps

> **Not legal advice.** This is an engineer's orientation so you can ask the right questions, design for compliance, and speak credibly with customers' legal/security teams. Regulations and standards change; **always involve counsel and verify against primary sources.** My research for this note covered the EU AI Act timeline (via secondary legal commentary) but **did not retrieve primary text on ISO/IEC 42001 or NIST AI RMF**, so those parts reflect established general knowledge — verify details.

## 1. Why governance matters to AI engineers & FDEs
Enterprise AI deals stall or die on **security reviews, data-protection questions, and risk sign-offs**. FDEs who can produce a **clear governance pack** (data flows, controls, evals, human oversight, incident plan) unblock deployments. Regulation also shapes architecture (residency, logging, explainability, human-in-the-loop).

## 2. Regulatory & standards landscape (orientation)
### 2.1 EU AI Act (risk-based regulation)
- **Risk tiers:** *unacceptable risk* (prohibited practices), **high-risk** (listed use cases — e.g., hiring, credit scoring, education, biometrics, critical infrastructure, essential services; and AI embedded in regulated products like medical devices), **limited risk** (transparency duties — e.g., disclosing AI interaction, labeling synthetic content), **minimal risk**. Separate obligations for **general-purpose AI (GPAI) model providers** (documentation, copyright policy, training-data summary; extra duties for models with systemic risk).
- **Roles:** *provider* (develops/places on market), *deployer* (uses under its authority), *importer/distributor*. Many FDE customers are **deployers** (and sometimes providers if they substantially modify/white-label) — determine role early.
- **High-risk obligations (provider-side, summarized):** risk-management system, data governance, technical documentation, record-keeping/logging, transparency to deployers, **human oversight**, accuracy/robustness/cybersecurity, quality-management system, conformity assessment & registration. **Deployer duties:** use per instructions, human oversight, monitoring, logs retention, informing affected people, fundamental-rights impact assessment in some cases.
- **Timeline (per secondary sources in 2026 — verify with the Official Journal / EU AI Office):** GPAI obligations applied from **2 Aug 2025**. A **"Digital Omnibus on AI" (Regulation (EU) 2026/1744)** reportedly entered into force **27 July 2026** and **postponed high-risk deadlines**: *Annex III stand-alone high-risk systems → 2 Dec 2027*; *Annex I product-embedded systems → 2 Aug 2028*; some public-authority systems later; **Article 50 transparency** duties reportedly applied from **2 Aug 2026** with a transitional period for machine-readable watermarking until **2 Dec 2026**; new prohibitions reportedly from 2 Dec 2026. The delay was attributed largely to unfinished harmonised standards. **Takeaway:** deadlines moved, but inventory/classification/governance work still takes time — start now.
- **Penalties:** significant fines scaled to global turnover (see regulation).
### 2.2 Data protection & privacy
- **GDPR (EU/UK GDPR):** lawful basis, purpose limitation, data minimization, transparency, **data-subject rights** (access, deletion, objection), **automated decision-making restrictions (Art. 22)**, **DPIA** for high-risk processing, **cross-border transfer** rules, processor agreements (**DPA**), breach notification (72 h), data residency pressures. LLM-specific issues: personal data in prompts/logs/embeddings/fine-tuning data; deletion across derived stores; provider subprocessors.
- **CCPA/CPRA and US state laws**, **HIPAA** (PHI; need BAAs; minimum necessary), **GLBA/SOX/PCI-DSS** (finance), **FERPA** (education), **COPPA** (children), sector regulators (SR 11-7 model risk management in banking; FDA for medical AI/SaMD; FINRA/SEC guidance on AI). Local rules in other regions (UK, Canada, India DPDP Act, Singapore, Japan, Brazil LGPD, China's generative-AI measures, etc.) — consult counsel.
### 2.3 Security & assurance frameworks
- **SOC 2 / ISO 27001** (security management) — customers expect vendors (and sub-vendors like model providers) to hold reports.
- **ISO/IEC 42001** (AI management system standard): a certifiable management-system framework for governing AI (policies, roles, risk assessment, lifecycle controls, monitoring, continual improvement) — analogous to ISO 27001 for AI. *(General knowledge; verify scope/clauses.)*
- **NIST AI Risk Management Framework (AI RMF 1.0)** and its **Generative AI Profile**: voluntary US framework organized around **Govern, Map, Measure, Manage** functions; widely used as a vocabulary for AI risk programs. *(General knowledge; verify current versions/profiles.)*
- **Other references:** OWASP LLM/Agentic Top 10 (technical risks), MITRE ATLAS (adversarial tactics), Google SAIF, CSA AI controls, ISO/IEC 23894 (AI risk management), IEEE standards, industry-specific model-risk guidance.
**Practical stance:** map your controls to **one** umbrella framework (e.g., NIST AI RMF or ISO 42001) and cross-reference regulations, instead of building separate checklists per law.

## 3. Building an AI governance program (what "good" looks like)
1. **Inventory (AI registry):** every AI system/use case, owner, purpose, model(s)/vendors, data sources, users, risk tier, status.
2. **Risk classification & intake process:** questionnaire at project start → tiering (low/medium/high) → required controls & approvals per tier.
3. **Roles & accountability:** executive sponsor, AI governance board/council, product owner, model/system owner, security, privacy/DPO, legal, risk/compliance, domain SME, red-team; RACI.
4. **Policies & standards:** acceptable use, data handling for AI, vendor/third-party AI, human oversight, transparency/disclosure, evaluation & release gates, incident response, retention.
5. **Lifecycle controls:** design review → threat model/risk assessment → data review → evaluation (quality, safety, fairness, robustness) → approval → monitored deployment → periodic review → decommissioning.
6. **Documentation:** system cards/model cards, data sheets, DPIAs/AI impact assessments, architecture & data-flow diagrams, eval reports, change logs, audit trails.
7. **Monitoring & assurance:** KPIs/KRIs, periodic re-evaluation, internal audit, third-party audits/certifications.
8. **Training & culture:** user training (limits/overreliance), developer training (secure/ethical AI), clear escalation channels.

## 4. Technical controls that satisfy governance asks
| Governance ask | Engineering response |
|---|---|
| "Where does our data go?" | data-flow diagram; provider terms (retention/training/ZDR); region pinning; VPC/private endpoints; on-prem option |
| "Can we audit what the AI did?" | structured logs/traces with versions, inputs/outputs (redacted), tool calls, approvals; retention policy |
| "How do you prevent harmful/unauthorized output?" | layered guardrails + evals + red-team results ([[07-Guardrails-and-Security-Hardening]]) |
| "Is it accurate/fair?" | task-specific eval reports with CIs and slice analysis; bias tests; monitoring |
| "Who can access what?" | SSO, RBAC, per-user authorization in retrieval/tools; tenant isolation |
| "Human oversight?" | approval gates, escalation paths, kill switch, review UX, sampling QA |
| "Explain a decision" | citations/evidence, structured rationales, decision logs (note: LLM self-explanations ≠ faithful causal explanations) |
| "Right to delete?" | deletion workflows across stores (prompts, traces, embeddings, caches, fine-tuning data) |
| "Change control?" | versioned prompts/models, eval gates, canary, rollback, release notes |
| "Incident readiness?" | playbooks, on-call, notification procedures |
| "Vendor risk?" | vendor assessments (SOC2/ISO, DPAs, subprocessors), exit plan/portability, multi-provider fallback |

## 5. Documentation artifacts to know how to write
- **System card / model card:** purpose, intended & out-of-scope uses, users, model(s) and versions, data sources, evaluation methods and results (incl. slices, limitations), safety measures, human oversight, known failure modes, monitoring, contacts. (Origin: Mitchell et al., "Model Cards for Model Reporting.")
- **AI impact / risk assessment (and DPIA):** benefits, stakeholders, harms (privacy, fairness, safety, security, IP, over-reliance), likelihood/severity, mitigations, residual risk, sign-off.
- **Data-flow diagram & data inventory:** categories (PII/PHI/confidential), purposes, processors/subprocessors, locations, retention.
- **Threat model & red-team report.**
- **Evaluation report:** datasets, metrics, thresholds, results, limitations, re-evaluation schedule.
- **Operational runbook & incident plan.**
- **Third-party/vendor assessment.**

## 6. Fairness, transparency & human oversight (engineering view)
- **Fairness testing:** identify protected/sensitive attributes relevant to the use case; evaluate performance/outcome disparities across slices (counterfactual prompts swapping attributes, matched pairs); document trade-offs (parity vs calibration vs equalized odds conflict); mitigate via data/prompt/guardrails/human review; reassess periodically.
- **Transparency:** disclose AI use; label generated content where required; provide sources/citations; explain limitations and confidence honestly.
- **Human oversight:** *human-in-the-loop* (approve each action), *on-the-loop* (monitor and intervene), *in-command* (set boundaries and shut down). Design for **meaningful** oversight (reviewers have time, context, authority; avoid rubber-stamping and automation bias).
- **Contestability/appeals** for consequential decisions.

## 7. Third-party/vendor & supply-chain governance
Assess model/API providers (security attestations, data policies, uptime/SLAs, deprecation policies, incident history, subprocessors, regional options, indemnities for IP claims), open-model **licenses** (permissive vs use-restricted; acceptable-use policies), dataset licenses, library/MCP-server provenance. Maintain an **exit/portability plan** (abstraction layer, evals to re-baseline on alternatives).

## 8. IP, copyright & content provenance
Risks: training-data claims, output infringement, license contamination in generated code, confidential-data exposure. Controls: provider indemnities (where offered), code-scanning/licence scanners for generated code, citation/provenance for retrieved content, policies on using outputs, watermark/provenance (C2PA) where required, content filters for known-copyright regurgitation.

## 9. FinOps for LLM systems (cost governance)
- **Unit economics first:** cost per request → **cost per successful task** → cost per customer/tenant → gross margin; compare to value delivered.
- **Visibility:** tag/attribute spend by feature/tenant/environment; dashboards; per-request cost in traces ([[03-Observability-and-Monitoring]]); reconcile with provider invoices.
- **Controls:** budgets and alerts, per-tenant quotas, max tokens/steps, kill switches for runaway agents, rate limits, approval for expensive features.
- **Optimization levers (ordered):** prompt caching → trim context/tool output → shorter outputs → Batch APIs for offline work → smaller models/routing validated by evals → lower effort per route → fine-tune/distill for the highest-volume tasks → self-host when utilization justifies ([[06-Serving-and-Infrastructure]]).
- **Commitments:** negotiate committed-use/enterprise discounts only after you understand baseline load; avoid lock-in via abstraction; consider priority/provisioned throughput for critical paths where available.
- **Forecasting:** model growth × tokens/task × price × cache hit; scenario analysis (price changes, model swaps).
- **Chargeback/showback** to internal teams; tie spend to outcomes (tickets resolved, hours saved).
- **Governance of experimentation:** sandbox budgets for R&D; cleanup of idle GPUs/endpoints.

## 10. FDE playbook: the "AI governance pack" for a customer deployment
1. **Executive summary** (purpose, scope, risk tier, benefits). 
2. **Architecture & data-flow diagram** with trust boundaries, data categories, regions, retention. 
3. **Roles & responsibilities** (customer vs vendor vs model provider). 
4. **Risk assessment** (+ DPIA if needed) with mitigations/residual risk. 
5. **Evaluation report** (quality, safety, fairness, robustness; limitations). 
6. **Security overview** (authN/Z, encryption, tenancy, guardrails, red-team summary, supply chain). 
7. **Human oversight design** and escalation. 
8. **Monitoring & incident response**; SLOs. 
9. **Change management** (versioning, release gates, model-deprecation plan). 
10. **Compliance mapping** (e.g., GDPR/HIPAA/EU AI Act role & tier, SOC 2 controls, internal policies). 
11. **Vendor appendix** (provider attestations, DPAs, subprocessors). 
12. **Open risks & roadmap.**

## Self-check
1. Distinguish provider vs deployer under the EU AI Act and why it matters. 2. What changed in the EU AI Act timeline in 2026 (per secondary sources) and what should you still do now? 3. Map five customer governance questions to technical controls. 4. What goes in a system card? 5. Define human-in/on/in-command oversight. 6. Build a cost-per-successful-task model and list your top three levers.

Next → [[09-Build-an-Eval-and-LLMOps-Pipeline]]
