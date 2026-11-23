---
tags: [phase-6, mcp]
---
# 03 — MCP: Model Context Protocol

> **MCP** is an open protocol that standardizes how LLM applications connect to **tools, data, and prompts**. Think "USB-C for AI integrations": write a server once, use it from any compliant client (IDEs, chat apps, agents). By 2026 it's the de facto integration layer — and a major FDE skill because customers want their systems exposed to agents *safely*.

> [!warning] Verify against the spec
> The protocol is versioned by date. Per the official MCP blog/spec pages found in late 2026, the **current revision is 2026-07-28**; secondary guides disagree on details (e.g., whether newer client-registration mechanisms "replace" older ones). Always read the **official spec** (https://modelcontextprotocol.io/) and SDK docs for the version you target.

## 1. Why MCP exists
Before MCP: N apps × M systems = N×M bespoke integrations, each with its own tool format/auth. MCP defines one protocol so: **tools are portable across clients**, ecosystems can share servers, and enterprises can centralize auth/governance in gateways. Governance moved in Dec 2025 to the **Agentic AI Foundation** under the Linux Foundation (co-founded by Anthropic, Block, OpenAI, per reports) — signaling broad vendor adoption.

## 2. Architecture
```
 Host (the LLM app: IDE, chat app, your agent)
   ├─ MCP Client ──1:1── MCP Server A (e.g., GitHub)
   ├─ MCP Client ──1:1── MCP Server B (e.g., your CRM)
   └─ MCP Client ──1:1── MCP Server C (e.g., local filesystem)
```
- **Host** orchestrates the model and holds user-consent/security policy. **Client** maintains one connection to one server. **Server** exposes capabilities.
- Messages are **JSON-RPC 2.0**. Lifecycle: **initialize** (negotiate protocol version & capabilities) → operate → shutdown.

## 3. Server primitives (what servers offer)
| Primitive | Controlled by | What | Example |
|---|---|---|---|
| **Tools** | model | executable functions with JSON-Schema inputs (and optional output schemas) | `create_ticket`, `query_sales` |
| **Resources** | application/user | readable data identified by URIs | `file:///…`, `db://customers/42`, docs |
| **Prompts** | user | reusable prompt templates/workflows (slash-commands) | "summarize-incident" |
## 4. Client primitives (what clients offer to servers)
- **Sampling:** server asks the host's LLM to generate (server-initiated completions, with the host mediating consent) — lets servers be "agentic" without their own API keys.
- **Roots:** client tells server which filesystem/URI boundaries it may operate in.
- **Elicitation:** server asks the *user* for more input mid-operation (forms/confirmations).
Other features in recent revisions include progress notifications, cancellation, logging, completions, structured tool outputs, and (per 2026 releases) **stateless core / multi-round-trip requests, header-based routing, cacheable list results, and a formal extensions framework** — check the spec for exact semantics.

## 5. Transports
- **stdio:** host launches the server as a local subprocess; messages over stdin/stdout. Simple, great for local tools (filesystem, git, DB). *Never write logs to stdout* (corrupts protocol) — use stderr.
- **Streamable HTTP:** remote servers over HTTP (POST requests with optional SSE streaming responses). Needed for hosted/multi-tenant servers and enterprise deployments. (The older "HTTP+SSE" transport is superseded.)
- Local servers = trust the code you run; remote servers = need real **authentication/authorization**.

## 6. Authorization (remote servers)
- Based on **OAuth 2.1** with the MCP server acting as an **OAuth resource server**; discovery via protected-resource and authorization-server metadata; **PKCE** for public clients; **resource indicators** (tokens bound to the specific MCP server — prevents token passthrough/confused-deputy issues).
- Client registration: Dynamic Client Registration (DCR) and **Client ID Metadata Documents (CIMD)** are the mechanisms discussed in the spec/guides; hardening in the 2026-07-28 revision includes **validating the `iss` parameter on authorization responses (RFC 9207)** and declaring OIDC `application_type` for native/CLI clients. An **enterprise-managed authorization** extension (reported stable mid-2026, backed by major IdPs) lets organizations govern which users/agents access which servers through their identity provider.
- **Practical rules:** never pass user tokens through to downstream APIs without audience checks; use short-lived, narrowly scoped tokens; per-user authorization (the server must enforce *that user's* permissions on the backend); log access; support revocation.
- For **service/background agents:** use workload identity/client-credentials style flows; keep secrets in vaults (some platforms inject credentials at the network egress so the sandbox never sees the real secret).

## 7. Building a server (Python, FastMCP-style)
```python
# pip install "mcp[cli]"      (official Python SDK; verify current API in docs)
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("orders")

@mcp.tool()
def get_order_status(order_id: str) -> str:
    """Look up an order's status by ID (e.g., 'ORD-123'). Read-only. Returns status, carrier, ETA."""
    row = db.fetch_one("SELECT status, carrier, eta FROM orders WHERE id = %s", (order_id,))   # parameterized!
    return f"{row.status} via {row.carrier}, ETA {row.eta}" if row else f"No order {order_id}. Check the ID format ORD-#####."

@mcp.resource("policy://returns")
def returns_policy() -> str:
    """The current returns policy text."""
    return open("policies/returns.md").read()

@mcp.prompt()
def investigate_delay(order_id: str) -> str:
    return f"Investigate why order {order_id} is delayed. Check status, carrier events, and policy, then draft a customer reply."

if __name__ == "__main__":
    mcp.run()                       # stdio by default; use the SDK's HTTP transport option for remote hosting
```
Test with the **MCP Inspector** (`npx @modelcontextprotocol/inspector`) and a real client (Claude Desktop/Code, Cursor, VS Code, your own agent via the SDK or the provider's MCP connector).
**TypeScript SDK**, Java/Kotlin/C#/Go/Rust SDKs exist; frameworks: **FastMCP**, MCP-use, Spring AI MCP, etc.

## 8. Server design best practices
- **Design for agents, not API parity** — apply [[02-Tool-Design-and-Environments]] principles: goal-oriented tools, concise results, great descriptions, actionable errors, pagination.
- **Small, focused servers** (one domain) or a curated gateway; avoid exposing 200 tools at once (context bloat) — consider tool search/deferred loading, tool filtering per client/role.
- **Read vs write separation;** annotate tools (`readOnlyHint`, `destructiveHint`, `idempotentHint`, `openWorldHint` — hints for clients, **not security guarantees**).
- **Validate every input server-side;** authorize per user; confine paths; parameterize queries; rate-limit; timeouts; size caps.
- **Structured outputs** + human-readable text.
- **Idempotency** for writes; **dry-run**/confirm flows (use elicitation for confirmations).
- **Observability:** log tool calls with user/session/request IDs; metrics; tracing.
- **Versioning & deprecation:** stable tool names/schemas; changelog; breaking-change policy.
- **Package & distribute:** container image, `uvx`/`npx` launchers, registry listing (the official **MCP Registry** exists — reported as preview with tens of thousands of listed servers in late 2026; treat counts skeptically and vet servers).

## 9. Security (MCP-specific — critical)
Threats documented by the community/OWASP:
- **Prompt injection via tool results/resources** (the main risk) and **tool poisoning** (malicious instructions hidden in tool *descriptions/metadata*).
- **Rug-pull / tool redefinition:** a server changes tool behavior/descriptions after approval → pin versions, hash/diff tool definitions, re-approve on change.
- **Tool shadowing / cross-server attacks:** one server's description manipulates how the model uses another's tools.
- **Confused deputy & token passthrough**, over-broad OAuth scopes, missing audience validation.
- **Command injection / path traversal / SSRF / SQL injection** in server code.
- **Excessive permissions** (a "filesystem" server with root access; a DB server with write access).
- **Supply chain:** malicious or typosquatted community servers; unvetted `npx`/`uvx` packages with full local privileges. **Only run servers you trust and have reviewed; sandbox local servers; prefer vetted/enterprise-registered ones.**
- **Data exfiltration** by combining a private-data server with an outbound-capable server (the lethal trifecta — [[07-Agent-Reliability-Safety-and-Security]]).
**Controls:** least-privilege scopes, per-tool allow/deny lists and **user approval for sensitive tools**, sandboxing, network egress filtering, MCP gateways/proxies (central auth, policy, logging, rate limits, schema pinning), human-in-the-loop for writes, audit logs, scanning tools (e.g., MCP security scanners), separate servers by trust level.

## 10. MCP vs other approaches
| Approach | Use when |
|---|---|
| **Native/function tools in your app** | single app, tight control, simplest; no cross-client reuse needed |
| **MCP server** | multiple clients/agents/teams need the same integration; ecosystem reuse; vendor-neutral; enterprise gateway governance |
| **Provider MCP connector** (API param to attach remote MCP servers) | quick remote integration without running an MCP client yourself; limited to remote servers/tool features supported |
| **Skills (folders of instructions/scripts)** | procedural knowledge/workflows loaded on demand; complement tools |
| **CLI/bash + docs** | agents are good at shell tools (`gh`, `kubectl`, `psql`); often simpler and cheaper in context than an MCP server — but harder to gate/audit |
| **Code-API wrappers (MCP as code)** | many tools + large data; process in sandboxed code |
| **A2A (Agent2Agent protocol)** | **agent-to-agent** interoperability across vendors (discovery, task delegation), complementary to MCP (agent-to-tool) |
Heuristic: MCP for **shared, governed integrations**; native tools for app-specific logic; skills for know-how; A2A when independent agents/organizations must collaborate.

## 11. MCP in enterprise/FDE engagements
Typical asks: expose internal systems (ticketing, CRM, ERP, data warehouse, knowledge base) to agents/copilots. Playbook: inventory systems & permissions → define a **small set of goal-oriented tools per domain** → build/host servers (container, VPC) behind an **MCP gateway** (SSO/OIDC, per-user authz, audit, rate limits) → read-only first, writes behind approvals → eval tool-selection & task success → monitor → expand. Deliver: server repo, auth design, threat model, runbook.

## 12. Hands-on checklist
- [ ] Build a stdio MCP server with 3 tools, 1 resource, 1 prompt; test in Inspector.
- [ ] Connect it to a client (desktop app) and to your own agent loop (SDK).
- [ ] Add per-user auth (HTTP transport + OAuth or API-key proxy) and enforce backend permissions.
- [ ] Write an injection test: a tool result with hostile instructions; mitigate.
- [ ] Compare MCP vs native tool for the same task on tokens/latency/accuracy.

## Self-check
1. Draw host/client/server and name the three server primitives. 2. stdio vs Streamable HTTP — when each? 3. Explain the MCP auth model and why resource indicators matter. 4. What is tool poisoning and how do you defend? 5. When would you prefer a CLI/skill over an MCP server? 6. How does A2A differ from MCP?

Next → [[04-Memory-Context-and-State]]
