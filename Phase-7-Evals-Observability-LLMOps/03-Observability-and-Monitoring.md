---
tags: [phase-7, observability, monitoring, tracing]
---
# 03 — Observability & Monitoring

> Traditional monitoring asks "is it up and fast?" LLM observability must also answer **"is it right, safe, and worth the cost?"** — for outputs that are non-deterministic and often unverifiable in real time. You need three layers: **telemetry** (logs/traces/metrics), **quality signals** (online evals, feedback), and **processes** (alerts, triage, flywheel).

## 1. What to capture (per request/run)
**Identity & context:** request/trace ID, user/tenant (pseudonymized), session/conversation ID, feature/route, environment, app version/commit.
**Versions:** prompt template ID+version, model name/version, tool/schema versions, retrieval index version, embedding model, guardrail policy version, feature flags.
**Inputs/outputs:** user input, rendered prompt (or hash + template vars), retrieved docs (IDs/scores), tool calls (name, args, results/size, errors), model output, final response, stop reason, refusal flags, structured-output validation results.
**Performance:** TTFT, total latency, per-stage latency (retrieval, rerank, LLM, tools), retries, queue time, cache hit/miss.
**Cost:** input/output/cached/thinking tokens, estimated $ (computed from a price table), per-tenant/feature attribution.
**Quality & safety signals:** judge scores (sampled), guardrail verdicts, user feedback (thumbs, edits, regenerate, copy, escalation), downstream outcomes (ticket reopened, task completed).
**Privacy controls:** redact PII/secrets before storage; separate access tiers; retention TTLs; legal basis/consent; ability to delete by user.

## 2. Logs vs metrics vs traces
- **Metrics** (aggregates over time): request rate, error rate, latency percentiles, tokens/s, $/hour, cache hit rate, refusal rate, judge pass rate, guardrail block rate, thumbs-down rate. → dashboards & alerts.
- **Logs** (events): discrete records with context; good for audit.
- **Traces** (causal span trees): one request → spans for retrieval, LLM calls, tool calls, sub-agents. **Essential for RAG and agents** to see *where* time/cost/errors occur and to replay failures.
Design so **a single trace ID** links metrics exemplars, logs, user feedback, eval scores, and cost.

## 3. Tracing standards: OpenTelemetry for GenAI
- **OpenTelemetry (OTel)** is the vendor-neutral tracing standard. Its **GenAI semantic conventions** define attribute names for LLM calls (`gen_ai.request.model`, token usage, operation names), **agent spans** (agent id/name/description/version, tool definitions), tool-execution spans, events for prompts/completions, and metrics (token usage, operation duration), plus MCP conventions.
- **Status (important):** per mid-2026 sources, the GenAI conventions are still **"Development"** (not stable), were moved to a dedicated `semantic-conventions-genai` repository in June 2026, and evolve on `main` without tagged releases yet. Some blog posts claim stability — primary sources contradict them. **Expect attribute renames; pin instrumentation versions; isolate naming behind a thin layer.**
- **Instrumentation options:** `opentelemetry-instrumentation-openai-v2` (most mature per sources), OpenLLMetry (Traceloop), OpenInference (Arize), vendor SDK integrations (Langfuse, LangSmith, Logfire, Datadog), framework callbacks (LangGraph/LlamaIndex/Pydantic AI), or **manual spans** with a decorator.
- **Backends that ingest OTel GenAI spans (per sources):** Datadog LLM Observability, Langfuse, Arize Phoenix/AX, plus general APMs (Grafana Tempo, Honeycomb, New Relic, etc.).
- **Content capture:** prompts/completions may contain sensitive data; most instrumentations make content capture **opt-in**; decide per environment, redact, and respect retention.

```python
# Manual span example (OTel Python) — attribute names follow GenAI conventions; verify current names
from opentelemetry import trace
tracer = trace.get_tracer("support-bot")
with tracer.start_as_current_span("chat claude-sonnet-5-5") as span:
    span.set_attribute("gen_ai.operation.name", "chat")
    span.set_attribute("gen_ai.request.model", "claude-sonnet-5-5")
    r = client.messages.create(...)
    span.set_attribute("gen_ai.usage.input_tokens", r.usage.input_tokens)
    span.set_attribute("gen_ai.usage.output_tokens", r.usage.output_tokens)
    span.set_attribute("app.prompt_version", PROMPT_VERSION)          # custom attrs: prefix with your namespace
    span.set_attribute("app.cost_usd", cost(r.usage))
```

## 4. Tooling landscape (verify features/pricing)
| Tool | Notes |
|---|---|
| **Langfuse** | open-source; tracing, prompt management, datasets/evals, scores, cost; self-host or cloud |
| **LangSmith** | tracing/evals/prompt hub; deep LangChain/LangGraph integration; framework-agnostic ingestion too |
| **Arize Phoenix / AX** | OTel/OpenInference-based tracing, evals, embeddings analysis |
| **Braintrust** | evals-first platform with logging, datasets, experiments, online scoring |
| **Pydantic Logfire** | OTel-based observability tied to Python/Pydantic AI |
| **Helicone / Portkey / LiteLLM proxies** | gateway-based logging, caching, budgets, routing |
| **Datadog / New Relic / Grafana / Honeycomb** | enterprise APMs adding LLM observability |
| **W&B Weave, MLflow Tracing, Comet Opik** | experiment-tracking vendors' LLM tracing |
| **Provider consoles** | usage/cost dashboards; agent session viewers (e.g., per-tool stats and costs) — good starting point, export for retention/governance |
Selection: OTel compatibility, data residency/self-host, PII redaction, eval/judge integration, cost attribution, UI for trace review (a *great* trace viewer saves hours), retention/export, pricing at your volume.

## 5. Dashboards you actually need
1. **Health:** request volume, success/error rates by type (4xx/5xx/timeouts/validation failures), latency p50/p95/p99 (TTFT & total), by route/model.
2. **Cost:** $/day by model/feature/tenant; tokens per request; cache hit rate; top-N expensive prompts/sessions; budget burn-down.
3. **Quality:** sampled judge pass rates by failure mode; user feedback rates; escalation/handoff rate; abstention/refusal rate; citation-validity (RAG); task success (agents).
4. **Safety/security:** guardrail triggers, injection detections, policy violations, PII-leak detections, suspicious tool calls, blocked outputs.
5. **Retrieval (RAG):** empty-result rate, score distributions, freshness lag, top-queries with low scores.
6. **Agents:** steps/run, loops detected, tool error rates, approvals pending/denied, budget exhaustions.
7. **Drift:** input length/language/topic mix, embedding-distribution shift, new-intent clusters, model-version changes.

## 6. Online evaluation (quality in production)
- **Sampled judge scoring:** run calibrated judges on a sample (commonly 5–20% of traces plus all errors/outliers/flagged items) asynchronously to control cost; stratify by tenant/feature/length.
- **Rule-based online checks:** schema validity, citation exists in retrieved set, forbidden phrases/PII regexes, response length bounds, tool-argument sanity.
- **Implicit user signals:** edit distance between draft and final, regenerate clicks, copy/accept, task completion, session abandonment, repeated questions, escalation to human.
- **Explicit feedback:** thumbs up/down with reason codes; "report issue."
- **Human review queues:** sample low-score/high-risk traces; reviewers label → new test cases + judge-calibration data.
- **Outcome metrics:** business KPIs (resolution rate, handle time, conversion, CSAT) — the ultimate truth, lagging.
- Keep **online judge = offline judge** (same prompt/version) so metrics are comparable.

## 7. Drift & anomaly detection
- **Input drift:** shifts in topic/language/length/intents (cluster embeddings; PSI/KS on features; new-cluster alerts).
- **Output drift:** changes in length, refusal rate, tool-use patterns, format-validation failures, judge scores.
- **Upstream changes:** provider model updates/deprecations/regressions, API behavior changes, tool/backend changes, retrieval index changes, data pipeline failures.
- **Alerting:** SLO-based (error budget burn), anomaly detection on cost/latency/quality metrics with sensible seasonality; **avoid alert fatigue** (page on user-impacting, ticket on trends).
- **Canary prompts/golden probes:** scheduled synthetic requests (known inputs → known expectations) to detect silent regressions even with no traffic.

## 8. SLOs for LLM features (examples)
- Availability ≥ 99.5% non-error responses; p95 TTFT ≤ 1.5 s; p95 total ≤ 8 s (for chat);
- Quality: calibrated-judge pass rate ≥ target on "policy compliance" ≥ 98%; hallucination-rate ≤ x%; citation validity ≥ 95%;
- Safety: 0 tolerated PII leaks/permission violations; ASR on red-team suite ≤ y%;
- Cost: ≤ $z per resolved ticket; cache hit rate ≥ w%.
Define **error budgets** and what happens when they're spent (freeze launches, fix reliability).

## 9. Incident response for LLM systems
1. **Detect** (alert/user report). 2. **Triage:** type — outage, quality regression, safety/security event, cost spike, data leak. 3. **Contain:** feature flag off, route to fallback model/human, tighten guardrails, disable a tool, rotate keys, block a tenant. 4. **Diagnose** from traces (what changed? prompt/model/index/tool/data/provider?), reproduce with saved inputs. 5. **Fix & verify** with evals; add regression test(s). 6. **Communicate** (internal + customer); 7. **Post-mortem** (blameless): root cause, detection gap, prevention. For security/privacy incidents, follow legal notification timelines (e.g., GDPR 72h breach notification) — involve counsel.

## 10. Feedback loops → continuous improvement (the data flywheel)
`production traces → flagged/sampled → human review & labels → failure taxonomy updated → new eval cases + judge calibration data → prompt/tool/retrieval/model changes → offline eval → canary → monitor` (and optionally → fine-tuning data). Measure **cycle time** (issue found → fix in prod) as a team KPI.

## 11. Cost observability (FinOps tie-in)
Tag every request with feature/tenant/environment; compute cost from usage × price table (version it); track **cost per successful task**, not per call; surface outliers (runaway agents, huge contexts); budgets/alerts per tenant; show cache savings; forecast with traffic growth. (See [[08-Governance-Compliance-and-FinOps]].)

## 12. Privacy & security of telemetry
Traces are a **data store of sensitive user content**: encrypt, restrict access (RBAC), redact PII/secrets at ingestion, set retention, support deletion requests, separate prod/dev, control vendor access (DPAs), avoid logging credentials/tool secrets, consider sampling full content only for flagged traces.

## Self-check
1. List what you log per LLM call (≥12 fields). 2. What does a trace show that metrics can't? 3. State the status of OTel GenAI conventions and how you'd mitigate churn. 4. Design online evaluation with a 10% sample and a human review queue. 5. Define three SLOs for a support bot. 6. Walk through incident response for "answers suddenly got worse."

Next → [[04-Release-Management-and-Experimentation]]
