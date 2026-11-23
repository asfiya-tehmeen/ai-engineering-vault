---
tags: [phase-7, guardrails, security, red-team]
---
# 07 — Guardrails & Security Hardening

> Agent-specific security (trifecta, least agency, sandboxing) is in [[../Phase-6-Agents-Tools-MCP/07-Agent-Reliability-Safety-and-Security]]. This note covers the **application-level guardrail stack, data protection, red-teaming, and security operations** for any LLM feature (chat, RAG, extraction, agents). Principle: **no single control is sufficient — layer them, measure each layer, and design for blast-radius limits.**

## 1. Guardrails: what and where
**Guardrails** = checks/controls around the model that enforce policy and reduce harm. Place them at multiple points:
```
user input → [input guardrails] → prompt assembly (+ retrieval/tools) → [context guardrails] → model
          → [output guardrails] → [action guardrails: tool-call/argument checks, approvals] → user / downstream systems
```
| Layer | Examples |
|---|---|
| **Input** | auth/rate limits, length caps, language/format checks, PII detection/redaction, jailbreak/prompt-injection classifiers, topic/scope filters, toxicity/abuse |
| **Context** | trust labeling & delimiting of retrieved/tool content, ACL filtering of retrieval, injection scanning of documents, source allow-lists |
| **Model-level** | system prompt policy, safety-tuned models, constrained/structured outputs, refusal handling, provider safety classifiers/fallback |
| **Output** | content-safety classification, PII/secret leakage detection, policy compliance (judge/regex), groundedness/citation checks, schema validation, link/image sanitization, brand/tone checks |
| **Action** | tool-call allow/deny lists, argument validation, spend/rate limits, human approval for risky actions, sandboxing |
| **Operational** | logging, anomaly detection, abuse monitoring, kill switches |

## 2. Guardrail implementation options
- **Rules/regex/allow-lists:** fast, deterministic, explainable; brittle; great for structured constraints (SSN pattern, forbidden words, URL allow-list).
- **Classifiers (small models):** Llama Guard–style safety classifiers, Prompt Guard–style injection/jailbreak detectors, toxicity models, PII NER (Presidio, spaCy, cloud DLP). Low latency/cost; need **threshold tuning** and eval.
- **LLM-as-guard:** policy-prompted judge checking input/output; flexible, handles nuance; adds latency/cost and can itself be attacked — use small fast models, binary outputs, structured responses, and treat as one layer.
- **Programmable frameworks:** **NeMo Guardrails** (Colang flows), **Guardrails AI** (validators + schemas), LangChain/LlamaIndex hooks, **cloud guardrail services** (AWS Bedrock Guardrails, Azure AI Content Safety/Prompt Shields, Google safety filters/Model Armor, OpenAI moderation endpoints), vendor security products (Lakera and others). Features and efficacy vary — **evaluate on your data**.
- **Constrained decoding/structured outputs** to prevent malformed actions.
- **Policy engines (OPA/Cedar)** for authorization of tool calls, separate from the model.

## 3. Latency, cost & UX of guardrails
- Run independent checks **in parallel** with the main call; **stream with output moderation** carefully (buffer/chunk-level checks; ability to retract); cache classifier results for repeated inputs; use small models; skip expensive checks for low-risk routes.
- **False positives are a product cost:** over-blocking frustrates users and masks real issues. Measure **block rate, false-block rate, and user friction**; provide graceful messages and alternatives; allow appeals/overrides in enterprise settings.
- **Fail-open vs fail-closed:** decide per risk (e.g., fail-closed for actions/PII leaks; fail-open-with-logging for low-risk content filters).

## 4. Evaluating guardrails (they need evals too)
- Build labeled sets: **harmful/benign pairs, near-miss benign prompts, injection samples (direct & indirect), PII samples, domain policy violations**, multilingual and obfuscated variants (encoding, typos, role-play, many-shot).
- Metrics: **precision/recall (per category), false-block rate on benign traffic, ASR end-to-end (not just classifier recall), latency, cost**; slice by language and channel.
- Test **combined stack** (attackers adapt to individual layers); regression-test after model/prompt changes.
- Track **drift:** new jailbreak techniques appear continuously; refresh datasets (community red-team feeds, internal findings).

## 5. Threats to LLM applications (OWASP Top 10 for LLM Apps — themes; verify current list)
1. **Prompt injection** (direct/indirect) · 2. **Sensitive information disclosure** (training data/PII/secrets/system prompt) · 3. **Supply chain** (models, datasets, plugins, packages, MCP servers) · 4. **Data & model poisoning** (training/fine-tune/RAG corpora/memory) · 5. **Improper output handling** (XSS/SQLi/command injection from model output; unsafe rendering) · 6. **Excessive agency** (over-permissioned tools/autonomy) · 7. **System prompt leakage** · 8. **Vector/embedding weaknesses** (retrieval leakage, poisoning, inversion) · 9. **Misinformation** (hallucination/overreliance) · 10. **Unbounded consumption** (denial of wallet/service, model extraction).
Map each threat to controls and owners in your threat model.

## 6. Data protection & privacy engineering
- **Data minimization:** send only necessary fields; mask/pseudonymize identifiers; tokenization/vault for sensitive IDs; avoid putting secrets in prompts.
- **PII/PHI handling:** detect & redact before logging/sending where policy requires; consider on-prem/VPC models for high-sensitivity data; **retention** limits; **right-to-delete** across prompts, traces, caches, embeddings, fine-tuning data, backups.
- **Provider terms:** data-retention and training policies, **zero data retention (ZDR)** availability (some features/models are ineligible), regional processing/data residency, subprocessors, DPAs, BAAs (HIPAA) where applicable.
- **Encryption** in transit/at rest; **customer-managed keys** where required (note: some provider features restrict combinations — check docs).
- **Access control & audit:** RBAC on prompts/logs/datasets; audit trails for who viewed sensitive traces.
- **Output leakage:** guard against reproducing training/context secrets; canary tokens in system prompts/data to detect leakage; don't rely on "don't reveal" instructions as security.
- **System prompt confidentiality:** assume prompts can be extracted; keep secrets/credentials/authz logic **out** of prompts.

## 7. Hardening the stack (engineering checklist)
- **Authentication/authorization** at gateway; per-user quotas; **abuse detection** (scraping, extraction, jailbreak farming); CAPTCHA/bot defenses where public.
- **Input validation:** size limits, file-type validation, malware scanning for uploads, SSRF protections for URL fetchers, sandboxed document parsers (PDF/Office exploits).
- **Output handling:** treat model output as **untrusted**: escape before rendering (XSS), parameterize SQL, validate shell/args, strip active content/links/images, enforce Content-Security-Policy.
- **Secrets:** vaults, short-lived tokens, rotation, no secrets in logs/prompts/traces; scan repos/prompts for secrets.
- **Network:** egress allow-lists for tools/fetchers; private connectivity to model endpoints; WAF.
- **Dependencies/supply chain:** pin & scan packages/containers; verify model checksums/signatures; vet third-party plugins/MCP servers/skills; private mirrors; avoid `pickle`-based weights from untrusted sources (use safetensors).
- **Resource controls:** max tokens, tool-call budgets, timeouts, per-tenant spend caps (denial-of-wallet defenses).
- **Monitoring:** security-focused dashboards; alerts on injection-detector spikes, unusual tool calls, data exfil patterns, cost anomalies; SIEM integration.
- **Incident response:** playbooks for prompt-injection incidents, data leakage, jailbreak virality, compromised keys; kill switches; customer notification procedures.

## 8. Red-teaming program
- **Scope & threat model first** (assets, attackers, abuse cases per feature).
- **Methods:** manual expert red-teaming (creative, multi-turn, multilingual), **automated attack generation** (Garak, PyRIT, promptfoo red-team plugins, LLM attackers), jailbreak corpora, indirect-injection harnesses (poisoned docs/web/email/tool outputs), agent-specific suites (AgentDojo), fuzzing of tool arguments, privilege-escalation scenarios.
- **Scoring:** attack success rate by category/severity, **time-to-bypass**, utility retention (don't "win" by refusing everything).
- **Process:** findings → severity triage → fixes (architecture first; guardrail tweaks second) → **regression tests** → retest; schedule recurring red-teams and after major changes; consider **bug-bounty/disclosure** channels; document residual risk for stakeholders.
- **Human factors:** social-engineering the *operators* (support staff approving agent actions), over-trust, approval fatigue.

## 9. Safety policy & content governance
Define a **usage policy** (what the system will/won't do), **escalation paths** (self-harm, abuse, legal threats), **age/region restrictions**, **disclosure** that users interact with AI, **human review** for sensitive categories, **feedback & appeals**, and **auditability**. Align prompts/classifiers/evals to the written policy (policy → test cases → measured compliance).

## 10. Common mistakes
Relying on the system prompt for security · one guardrail layer · no eval of the guardrails themselves · blocking benign traffic without measuring · logging raw PII/secrets · trusting model output in downstream interpreters · ignoring indirect injection (only testing user-typed attacks) · public endpoints without quotas · no incident playbook · treating provider safety features as complete compliance.

## Self-check
1. Draw the guardrail stack and place five controls. 2. How do you evaluate a prompt-injection classifier end to end? 3. Fail-open vs fail-closed — examples of each. 4. List OWASP LLM Top 10 themes and a control for each of five. 5. Why must model output be treated as untrusted input? 6. Outline a quarterly red-team program.

Next → [[08-Governance-Compliance-and-FinOps]]
