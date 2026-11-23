---
tags: [phase-6, tools, environments, computer-use]
---
# 02 — Tool Design, Environments & Computer Use

> "Tools are the agent's hands — and its eyes." Most agent failures are tool-interface failures. Treat tool design as **agent–computer interface (ACI) design**, with the same care you'd give a UI for humans. (Builds on [[../Phase-4-Building-with-LLM-APIs/03-Structured-Outputs-and-Tool-Use]].)

## 1. Designing the tool surface
### Bash vs dedicated tools
- A **bash tool** gives broad leverage (almost any action) but your harness only sees an **opaque command string** — it can't distinguish a harmless `grep` from a dangerous `git push`.
- **Promote an action to a dedicated tool when you need to:** **gate** it (security/reversibility — `send_email`, `delete_record`, `transfer_funds`), **check staleness** (an `edit` tool can reject writes if the file changed since last read), **render** custom UI (a question modal), **parallelize safely** (read-only tools marked parallel-safe), **audit** typed arguments.
- Rule of thumb: **start with bash/code for breadth; promote to dedicated tools where you need control.**

### Principles for effective tools
1. **Choose the right tools — fewer, higher-level.** Match tools to agent *goals*, not API endpoints. Consolidate chains (`list_users`+`list_events`+`create_event` → `schedule_event`) when the chain is always used together. Too many overlapping tools confuse selection.
2. **Namespacing** — consistent prefixes (`asana_search`, `jira_search`) help selection among many tools.
3. **Return meaningful, token-efficient context** — natural-language names over cryptic IDs (but include the IDs the agent needs for follow-ups); `response_format: concise|detailed` parameter; pagination, filtering, truncation with sensible defaults and guidance ("showing 10 of 240; refine your query or use `offset`").
4. **Write descriptions like onboarding a new teammate** — purpose, when to use/not use, parameter meaning and formats, examples, side effects, limits. Small description edits often yield big accuracy gains.
5. **Unambiguous parameters** — `user_email` not `user`; enums for closed sets; avoid parameters the agent can't know.
6. **Helpful errors** — what went wrong + what to try next. Avoid stack traces and opaque codes.
7. **Make misuse hard (poka-yoke)** — e.g., require absolute paths, validate inputs, make destructive options explicit.
8. **Idempotency & safety** — idempotency keys for writes, dry-run modes, soft deletes, confirmation tokens.
9. **Evaluate tools with real tasks** — measure tool-selection accuracy, argument correctness, number of calls, tokens consumed, errors; read transcripts; iterate (optionally let an agent propose tool-description improvements from failure transcripts, validated on held-out tasks).
10. **Stable & cache-friendly** — keep tool lists/order stable within a session (changing tools invalidates prompt-cache prefixes); for large catalogs use **tool search/deferred loading**.

## 2. Anthropic-provided tool types (Claude API; verify availability per platform)
| Tool | Side | Use |
|---|---|---|
| **Bash** | client | run shell commands (you execute; sandbox!) |
| **Text editor** | client | view/create/edit files with str-replace semantics |
| **Computer use** | client or server | screenshot + mouse/keyboard for GUIs |
| **Code execution** | server | Anthropic-hosted sandbox with bash/file sub-tools; can create files |
| **Web search / web fetch** | server | fresh info, URL contents with citations |
| **Memory** | client | read/write a `/memories` directory you back with storage |
| **Tool search** | server | discover tools on demand from a large set |
Other vendors offer equivalents (OpenAI: web search, file search, code interpreter, computer use; Google: search grounding, code execution, URL context). Client-side tools: **you** execute and must secure them (confine file paths, whitelist commands, time-limit, size-limit).

## 3. Programmatic tool calling (PTC) & code as the action space
Standard loop: each tool call is a round trip; intermediate results flood context. **PTC:** the model writes a **script** in a sandbox that calls tools as functions, loops/filters/aggregates, and returns only the final output → fewer round trips and far fewer tokens for multi-step/large-data work. Related idea: **"code execution with MCP"** — present MCP tools as code APIs (files in a directory the agent can read on demand), so the agent loads only the tool definitions it needs and processes data in code before it reaches the context. Trade-off: needs a secure code sandbox; harder to gate individual actions.
**CodeAct-style agents:** actions expressed as executable Python — compositional, powerful, riskier.

## 4. Scaling the instruction & tool set
- **Tool search:** many tools, few relevant per request — search over tool descriptions (regex/BM25/embeddings) and load only matched schemas; appended (not swapped) to preserve cache.
- **Skills:** task-specific instructions/resources packaged as folders (`SKILL.md` + scripts/resources). Only each skill's short description sits in context; the agent reads the full skill when relevant ("progressive disclosure"). Good for reusable procedures (e.g., "create a PPTX," "run the quarterly report").
- **Progressive disclosure** generally: keep fixed context small; load detail on demand (file reads, search, sub-agents).

## 5. The filesystem as an agent's workspace
Powerful, simple pattern: give the agent a **working directory** with files it can read/write/search (`grep`, `glob`, `cat`). Uses: **context offloading** (large tool outputs saved to files, agent reads slices), **notes/plan files**, **intermediate artifacts**, **memory**, **skills**, **inputs/outputs** for the human. Agentic search over a filesystem often beats pre-embedding everything for codebases and structured corpora (just-in-time retrieval; Phase 5 note 06).

## 6. Code execution environments (sandboxes)
Where agent-run code executes determines both capability and risk.
- **Options:** provider-hosted code-execution tools; **containers** (Docker) with resource limits and no network/secrets; microVMs (Firecracker/Kata/gVisor) for stronger isolation; cloud sandboxes (E2B, Modal, Daytona, Cloudflare/Vercel sandboxes, etc.); **OS-level sandboxing** (seccomp/Landlock/macOS Seatbelt) for local coding agents; WebAssembly sandboxes.
- **Controls:** no ambient credentials; scoped, short-lived secrets injected only where needed (vault-style egress substitution so the sandbox sees placeholders); **network allow-lists/egress filtering**; filesystem confinement; CPU/memory/time limits; ephemeral per-session containers; output size caps; audit logs.
- **State:** persist workspace between turns (container reuse/snapshots) only when needed.
- Self-hosted vs vendor-hosted sandboxes: data residency & compliance often push FDE engagements to run tool execution **inside the customer's VPC**.

## 7. Computer use & browser agents
**Loop:** screenshot → model decides action (click x,y / type / scroll / key) → execute → new screenshot. Variants: **vision-based** (pixels), **DOM/accessibility-tree based** (structured, cheaper, more reliable on web), hybrid (Playwright/Selenium + LLM), **API-first** (prefer direct APIs when they exist).
- **Strengths:** automate legacy GUIs with no API (ERP, mainframe emulators, internal portals), cross-app workflows, QA testing, data entry.
- **Weaknesses:** **slow and costly** (many screenshot tokens per step), brittle to UI changes, coordinate errors, CAPTCHAs/auth walls, error compounding over long horizons; **highest prompt-injection risk** (arbitrary web content in view; hidden text/overlays).
- **Mitigations:** run in an **isolated VM/container** with minimal credentials; restrict domains; human approval for submits/purchases/sends; avoid logged-in personal profiles; screenshot redaction; step limits; record sessions for audit; prefer **structured/DOM tools** where possible; verify outcomes via side channels (database/API) rather than trusting the screen.
- **Evaluation:** OSWorld-style tasks, WebArena/WebVoyager, BrowseComp (research browsing). Benchmarks saturate fast — build your own task suite on the target apps.
- Newer computer-use tool versions/toolsets keep changing (some older tool type identifiers are rejected on new models) — **check current docs before coding**.

## 8. Human-facing tools
- **Ask-user tool** (clarifying questions with options; blocks the loop until answered) — better than guessing; render as UI modal.
- **Plan/todo tool** (agent maintains visible checklist; improves focus and user trust).
- **Approval tool/hook** (see [[07-Agent-Reliability-Safety-and-Security]]).
- **Notify/handoff** (escalate to a human with a summary and evidence).

## 9. Tool-result hygiene (context economics)
Cap output size; return diffs not whole files; summarize logs; paginate; strip HTML; include only fields needed; offload big artifacts to files and return paths + summaries; clear stale tool results via context editing ([[04-Memory-Context-and-State]]).

## 10. Tool evaluation recipe
1. Collect 30–100 realistic tasks (from real usage; include hard/ambiguous ones). 2. Run the agent; log **tool calls, args, results, tokens, runtime, errors**. 3. Metrics: task success, **tool-selection accuracy**, **argument validity**, redundant/failed calls, tokens/task, latency. 4. Read transcripts for confusion patterns. 5. Improve descriptions/results/consolidate tools. 6. Re-run on held-out tasks to avoid overfitting.

## Self-check
1. When do you promote a bash action to a dedicated tool? 2. List five tool-design principles with an example each. 3. What is programmatic tool calling and its trade-off? 4. Describe sandbox controls for a code-executing agent. 5. Why is computer use the riskiest tool class? 6. How would you evaluate a tool set?

Next → [[03-MCP-Model-Context-Protocol]]
