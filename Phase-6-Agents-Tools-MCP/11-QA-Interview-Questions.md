---
tags: [phase-6, qa, interview]
---
# 11 — Q&A (Phase 6)

> Answer aloud first, then expand.

## Fundamentals
> [!question]- Q1. Agent vs workflow?
> A workflow orchestrates LLM calls and tools through predefined code paths; an agent lets the LLM dynamically decide its own steps and tool use based on feedback. Agents trade predictability, cost, and latency for flexibility on open-ended tasks.

> [!question]- Q2. When should you build an agent rather than a workflow?
> When the task is multi-step and hard to specify in advance (complexity), the value justifies higher cost/latency, the model is viable at each step type, and errors are detectable/recoverable. If you can draw the flowchart, build a workflow.

> [!question]- Q3. Name the five workflow patterns.
> Prompt chaining, routing, parallelization (sectioning/voting), orchestrator–workers, evaluator–optimizer.

> [!question]- Q4. What does the harness own vs the model?
> The model reasons and chooses actions; the harness owns the loop, stop conditions/budgets, tool execution safety (validation, authz, sandboxing, timeouts), context management, state persistence/checkpointing, human-in-the-loop hooks, and observability.

> [!question]- Q5. Why do agents fail more than single calls?
> Errors compound across steps (95% per step ≈ 60% over 10), non-determinism, tool/interface problems, context rot, premature completion, and expanded security exposure.

> [!question]- Q6. What's the best single reliability lever?
> Verification against checkable feedback — tests, schema/validators, state checks — and designing tasks so success is observable.

> [!question]- Q7. What is "least agency"?
> OWASP's principle that autonomy must be earned by the task: grant the minimum decision-making freedom (not just the minimum permissions) — fewer tools, narrower scope, human approvals for risky actions.

> [!question]- Q8. ReAct vs plan-and-execute?
> ReAct interleaves reasoning, action, and observation step by step (adaptive); plan-and-execute drafts a plan first then executes and re-plans on surprises (good for long tasks; plan kept in state/file). Modern reasoning models do ReAct-style interleaving natively.

## Tools & environments
> [!question]- Q9. What makes a good tool?
> Goal-oriented (not thin API wrappers), clear name/description (when to use/not), unambiguous parameters, concise token-efficient results with pagination/filters, actionable errors, safe/idempotent behavior, and evidence from evals that agents select and use it correctly.

> [!question]- Q10. Bash vs dedicated tools?
> Bash gives breadth but opaque command strings. Promote actions to dedicated tools when you need to gate (security/reversibility), check staleness, render custom UI, parallelize safely, or audit typed args.

> [!question]- Q11. What is programmatic tool calling?
> The model writes code in a sandbox that calls tools as functions, processes results with loops/filters, and returns only the final output — fewer round trips and far fewer tokens for multi-step or data-heavy tasks; requires a secure sandbox.

> [!question]- Q12. How do you scale to hundreds of tools?
> Tool search/deferred loading (load only relevant schemas), namespacing, consolidating tools, skills with progressive disclosure, per-role tool filtering, and gateways.

> [!question]- Q13. What are Agent Skills?
> Folders with a `SKILL.md` (plus scripts/resources) packaging procedural knowledge; only short descriptions sit in context, and the agent loads the full skill when relevant.

> [!question]- Q14. How should an agent-run code sandbox be configured?
> Ephemeral container/microVM, no ambient credentials, scoped short-lived secrets (injected at egress), network allow-list, filesystem confinement, CPU/memory/time limits, non-root, audit logs — enforced by the runtime, not by prompting.

> [!question]- Q15. Computer-use agents: strengths, weaknesses, mitigations?
> Automate GUIs without APIs, but slow, costly, brittle, error-compounding, and high injection risk. Isolate in a VM, restrict domains, minimize credentials, require approvals for submits/purchases, prefer DOM/API tools where possible, verify via side channels, and cap steps.

> [!question]- Q16. How do you evaluate a tool set?
> Realistic task suite; log tool calls/args/results/tokens/errors; measure tool-selection accuracy, argument validity, redundant calls, tokens/task, success; read transcripts; refine descriptions/results; validate on held-out tasks.

## MCP
> [!question]- Q17. What is MCP and why does it matter?
> An open protocol standardizing how LLM apps connect to tools, data, and prompts via servers, so integrations are written once and reused across clients, with centralized governance options. It's become the de facto integration layer for agents.

> [!question]- Q18. Explain host, client, and server, and the server primitives.
> The host is the LLM app; it runs one client per server connection; servers expose tools (model-controlled actions), resources (app-controlled data by URI), and prompts (user-controlled templates). Clients can offer sampling, roots, and elicitation.

> [!question]- Q19. stdio vs Streamable HTTP?
> stdio: local subprocess, simple, trust the code you run (log to stderr). Streamable HTTP: remote/multi-tenant servers needing real authentication/authorization.

> [!question]- Q20. How does MCP authorization work at a high level?
> OAuth 2.1-style: the MCP server is a resource server; clients discover metadata, use PKCE, bind tokens to the server with resource indicators (audience), and register via mechanisms like DCR/client metadata documents; recent revisions harden validation (e.g., issuer checks) and add enterprise-managed authorization extensions. Verify details in the current spec.

> [!question]- Q21. What is tool poisoning / a rug pull?
> Tool poisoning: malicious instructions hidden in tool descriptions/metadata manipulate the model. Rug pull: a server changes tool behavior/definitions after approval. Defenses: vet and pin servers, hash/diff tool definitions, re-approve on change, allow-lists, sandboxing, approvals.

> [!question]- Q22. MCP vs plain function tools vs CLI?
> Native tools for app-specific logic and tight control; MCP when multiple clients/teams need the same governed integration; CLI/bash for tools agents already know (often cheaper in context but harder to gate/audit). Skills carry procedures; A2A connects independent agents.

> [!question]- Q23. A2A vs MCP?
> MCP connects an agent to tools/data; A2A lets independent agents (possibly across vendors) discover each other and delegate tasks. They're complementary.

> [!question]- Q24. Design an enterprise MCP rollout.
> Inventory systems/permissions; define goal-oriented tools per domain; host servers in the customer's VPC behind a gateway (SSO/OIDC, per-user authz, rate limits, audit, schema pinning); read-only first, writes behind approvals; eval and monitor; threat-model and document.

## Memory, context, state
> [!question]- Q25. Compaction vs context editing vs memory?
> Compaction summarizes older context (lossy); context editing deletes stale blocks like old tool results; memory persists information outside the window across sessions. Use combinations, and test fact survival.

> [!question]- Q26. How do you make a long-running agent crash-safe?
> Explicit serializable state, checkpoint after each step, idempotent tools/idempotency keys, durable execution engine, human approval as a persisted state, and resume logic.

> [!question]- Q27. What is memory poisoning?
> Untrusted content (e.g., an injected "preference") gets written to long-term memory and influences future sessions. Mitigate: provenance, sanitize/summarize, confirm writes derived from untrusted sources, per-tenant stores, review diffs, TTL/rollback, tests.

> [!question]- Q28. Describe a harness for tasks spanning many context windows.
> Initializer sets up environment and a task list with statuses; each session picks one item, implements, verifies, updates progress notes and the task list, and commits, ending in a clean handoff state; guardrails prevent premature "done."

> [!question]- Q29. Why can mutating the system prompt or tool list mid-run hurt?
> Prompt caches are prefix-based, so changes invalidate cached prefixes, raising cost/latency. Prefer append-only history, mid-conversation system messages where supported, and tool search for dynamic tools.

## Multi-agent
> [!question]- Q30. When is multi-agent justified?
> Context isolation (subagents return summaries), parallelizable breadth, specialization, separation of duties/safety, and scale beyond one window — and only if it measurably beats a single agent for the cost.

> [!question]- Q31. Cost of multi-agent?
> Several times the tokens of a single agent (vendor-reported figures for research systems: ~4× an agent, ~15× a chat); worthwhile only for high-value, breadth-first tasks.

> [!question]- Q32. Describe an orchestrator–worker research system.
> Lead plans and saves the plan, spawns parallel workers with clear objectives, output formats, and boundaries; workers search and return condensed sourced findings (ideally via shared artifacts); lead synthesizes, spawns follow-ups, and a verification/citation pass checks claims; effort-scaling rules and budgets prevent over-spend.

> [!question]- Q33. Name multi-agent failure modes.
> Duplicate work, vague delegation, telephone-game information loss, runaway spawning/cost, infinite ping-pong, accepting bad claims, premature termination, injection spreading across agents, and hard debugging.

> [!question]- Q34. How can multi-agent design improve security?
> Separate a quarantined reader (untrusted input, no tools) from a privileged actor that only receives schema-validated data; give each agent least-privilege credentials; add an independent verifier.

> [!question]- Q35. How do you decide, with data, whether to go multi-agent?
> Run the same eval suite on single vs multi-agent variants, comparing quality, coverage, citation accuracy, tokens/cost, latency, and variance; choose the Pareto-best option.

## Frameworks
> [!question]- Q36. When do you not need a framework?
> When a workflow of a few functions suffices, when you need full transparency/control, or while learning. Add frameworks for persistence/HITL/tracing needs, deliberately, with an escape hatch.

> [!question]- Q37. LangGraph vs a vendor SDK?
> LangGraph: explicit state graphs, persistence/interrupts/time-travel, model-agnostic, more boilerplate. Vendor SDKs: faster to start and tightly integrated with the provider's models/tools, but less control and more lock-in.

> [!question]- Q38. "Harness-only" vs "harness + deployment"?
> SDKs like the Claude Agent SDK or OpenAI Agents SDK supply the loop/tools but you host and deploy; hosted agent runtimes (e.g., Managed Agents) also provide the sandbox/infrastructure, state, and event streaming — at the cost of vendor dependence and platform constraints.

> [!question]- Q39. How do you keep an agent portable across frameworks?
> Keep prompts, tool schemas, and eval suites framework-independent; isolate framework code behind a thin interface; maintain a golden task suite to prove parity when porting.

> [!question]- Q40. Build vs buy for an enterprise support agent?
> Consider differentiation, data sensitivity, customization/integration depth, security review burden, total cost, vendor stability, time-to-value; often buy the platform/runtime and build tools, workflows, and evals on top.

## Security
> [!question]- Q41. State the lethal trifecta.
> An agent with access to private data, exposure to untrusted content, and the ability to communicate externally can be induced to exfiltrate data. Break at least one leg.

> [!question]- Q42. Why can't prompting alone stop prompt injection?
> Instructions and data share one text channel and models can't perfectly separate them; classifiers and training reduce but don't eliminate attacks. Use architectural controls (least privilege, isolation, egress control, approvals).

> [!question]- Q43. Direct vs indirect injection?
> Direct: the user supplies malicious instructions. Indirect: instructions hidden in content the agent reads — web pages, emails, documents, tool descriptions/outputs, other agents' messages.

> [!question]- Q44. List five defenses in depth.
> Least privilege/agency, breaking the trifecta (reader/actor separation), sandboxing with egress allow-lists, approval gates with exact action display, output/argument validation and monitors, credential isolation, budgets/rate limits, audit logging and red-teaming.

> [!question]- Q45. How do you prevent exfiltration via rendering?
> Sanitize/disable remote image/link rendering from agent output, allow-list domains, block auto-fetch of URLs in privileged contexts, and monitor outbound requests.

> [!question]- Q46. Confused deputy in agents?
> A powerful agent performs an action for a low-privilege user because it uses its own broad credentials. Fix with delegated user-scoped tokens and backend authorization checks on every tool call.

> [!question]- Q47. How do you keep credentials out of the agent's reach?
> Store in vaults; inject at network egress or via a token broker so the sandbox sees placeholders; short-lived scoped tokens; never put secrets in prompts, memory, or tool results.

> [!question]- Q48. How do you avoid approval fatigue?
> Auto-allow read-only actions, batch low-risk approvals, risk-tier policies by tool/args, show concise exact actions with provenance, and reserve prompts for high-impact operations.

> [!question]- Q49. What is the OWASP Top 10 for Agentic Applications' main message?
> Prompt injection/goal hijack underlies many risks; other themes include tool misuse, over-privileged identities, supply chain compromise, unexpected code execution, memory/context poisoning, insecure inter-agent communication, cascading failures, human trust exploitation, and rogue agents — mitigated by least agency and layered controls.

## Evaluation
> [!question]- Q50. pass@k vs pass^k?
> pass@k: at least one of k attempts succeeds (capability); pass^k: all k succeed (consistency). Production agents need pass^k — e.g., pass@1 70% can coexist with pass^4 ~30%.

> [!question]- Q51. Why grade outcomes rather than trajectories?
> Agents may find valid unexpected paths; strict golden-path matching penalizes them. Grade the final environment state, but still check hard constraints and safety in the trajectory.

> [!question]- Q52. Code vs LLM vs human graders?
> Code/state-based graders are objective and cheap (preferred); LLM judges handle soft criteria but need rubrics, calibration, version pinning, and bias checks; humans calibrate and review ambiguous/high-stakes cases.

> [!question]- Q53. How do you start an agent eval with little data?
> Begin with 20–50 real or realistic tasks from failures and manual tests, unambiguous pass/fail, a sandbox environment that resets, multiple trials, and iterate by reading transcripts.

> [!question]- Q54. Flaws of public agent benchmarks?
> Contamination (e.g., reports of solution leakage in SWE-bench Verified), saturation, scaffold/harness dependence, cherry-picked settings, benchmark bugs and gameable graders. Use them as rough screens; build your own suite.

> [!question]- Q55. What do you capture in an agent trace?
> LLM calls (model, prompt version, tokens, latency, stop reason), tool calls (args, results, latency, errors), retrieval steps, sub-agent spans, approvals, memory ops, checkpoints, and cost — with correlation IDs and PII redaction.

> [!question]- Q56. Your agent works in demos but fails 30% in production. What do you do?
> Gather traces and failures, categorize (tool errors, ambiguity, injection, context issues, env drift), build/expand an eval from them, measure pass^k, fix top categories (tools, verification, prompts, guardrails), canary changes, and monitor with feedback loops.

## FDE scenarios
> [!question]- Q57. A customer wants an agent to "automate procurement." First steps?
> Map the process and decision points, identify systems/permissions, classify steps (deterministic workflow vs agentic), define success metrics and risk tiers, pick a thin vertical slice with human approvals, build tools/MCP servers with least privilege, create an eval from historical cases, pilot, then expand autonomy as evidence supports.

> [!question]- Q58. How do you explain agent risk to a CISO?
> Use the lethal trifecta: show which legs the design has, how you broke one, the least-privilege/sandbox/egress/approval controls, audit logging, red-team results (attack success rates), and residual risk with an incident plan — not "the prompt tells it to be safe."

> [!question]- Q59. The customer wants fully autonomous operation on day one. Respond.
> Propose a graduated autonomy plan: start read-only/suggest-only, add approvals for writes, measure reliability (pass^k, error rates, policy violations) on their real cases, widen autonomy per action class when thresholds are met, with monitoring and rollback — autonomy earned by evidence.

> [!question]- Q60. How do you estimate cost for a support agent handling 50k conversations/day?
> Profile tokens per conversation (system/tools cached, history, tool results, output/thinking), loop counts including retries, model tiers/routing, cache hit rates, human handoff rate; multiply by volume and add overhead; validate on a pilot; set per-conversation budgets and alerts.

Next → [[12-Resources-and-Practice-Links]]
