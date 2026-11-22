---
tags: [phase-1, http, api]
---
# 06 — HTTP, APIs & JSON

> Every LLM provider is an HTTP API. Every enterprise integration you do as an FDE is an API, webhook, or file drop. Master this.

## 1. What happens when you visit `https://api.example.com/v1/items?limit=5`
1. **DNS** resolves hostname → IP.
2. **TCP** handshake, then **TLS** handshake (cert verified, keys exchanged).
3. Client sends an **HTTP request**:
```
GET /v1/items?limit=5 HTTP/1.1
Host: api.example.com
Authorization: Bearer sk-…
Accept: application/json
```
4. Server routes, authenticates, queries DB, builds response:
```
HTTP/1.1 200 OK
Content-Type: application/json
{"items":[…]}
```
5. Connection reused (keep-alive/HTTP2) or closed.

## 2. HTTP essentials
- **Methods:** `GET` (read, safe, idempotent), `POST` (create/action, not idempotent), `PUT` (replace, idempotent), `PATCH` (partial update), `DELETE` (idempotent).
- **Status codes:** 2xx ok (200, 201 created, 204 no content) · 3xx redirect · **4xx client error** (400 bad req, 401 unauthenticated, 403 forbidden, 404, 409 conflict, 422 validation, **429 rate limit**) · **5xx server error** (500, 502, 503, 504 timeout; **529 overloaded** on some LLM APIs).
- **Headers:** `Content-Type`, `Accept`, `Authorization`, `User-Agent`, `Retry-After`, `Cache-Control`, `ETag`, CORS headers.
- **Body formats:** JSON (default), form-encoded, multipart (file upload), **SSE** (server-sent events — how LLM token streaming works), WebSockets (bidirectional), gRPC/Protobuf (internal services).
- **Query params** (filter/paginate), **path params** (`/users/42`).
- **HTTP/1.1 vs 2 vs 3**: multiplexing, header compression, QUIC.
- **Cookies/sessions** vs **tokens**; **CORS** (browser-only restriction).

## 3. REST design basics
- Resources as nouns: `/users`, `/users/42/orders`. Verbs via methods.
- Consistent error format: `{"error":{"code":"invalid_request","message":"…"}}`.
- **Pagination:** offset/limit vs **cursor-based** (stable, scalable). **Filtering, sorting**.
- **Versioning:** `/v1/`. **Idempotency keys** for safe POST retries (`Idempotency-Key` header).
- **OpenAPI/Swagger**: machine-readable API spec → generate docs/clients. FastAPI generates it for you.
- Alternatives: **GraphQL** (client picks fields), **gRPC**, **webhooks** (server calls *you* on events — verify signatures!), **polling**.

## 4. Authentication & authorization
| Method | Notes |
|---|---|
| API key | static secret in header; simple; rotate; scope it |
| Bearer token / JWT | signed claims; expiry; verify signature |
| **OAuth 2.0** | delegated access; auth-code + PKCE flow; scopes; refresh tokens |
| **OIDC / SAML / SSO** | enterprise identity — FDEs meet these constantly (Okta, Entra ID) |
| mTLS | both sides present certs |
| HMAC signatures | webhook verification |
- **Authn** = who are you; **Authz** = what may you do (RBAC/ABAC).
- Secrets: env vars / secret manager (AWS Secrets Manager, Vault); never in git/logs.

## 5. Calling APIs from Python
```python
import httpx, os
client = httpx.Client(base_url="https://api.example.com", timeout=httpx.Timeout(10.0, connect=5.0),
                      headers={"Authorization": f"Bearer {os.environ['API_KEY']}"})
r = client.get("/v1/items", params={"limit": 5})
r.raise_for_status()
data = r.json()
```
**Always:** set timeouts · handle non-2xx · validate the response (Pydantic) · log request IDs.

### Retries done right
- Retry only **idempotent** ops or ones with idempotency keys.
- Retry on: connection errors, 408, 429, 5xx. Don't retry 4xx (except 429).
- **Exponential backoff + jitter**: `sleep = min(cap, base * 2**attempt) * random(0.5, 1.5)`. Honor `Retry-After`.
- Libraries: `tenacity`, `backoff`. Add a **circuit breaker** for persistent failure.
- Rate limits: token bucket, client-side semaphore. LLM APIs limit **requests/min and tokens/min**.

### Pagination loop
```python
def all_items():
    cursor = None
    while True:
        r = client.get("/items", params={"cursor": cursor}).json()
        yield from r["items"]
        cursor = r.get("next_cursor")
        if not cursor: break
```

## 6. Building an API with FastAPI
```python
from fastapi import FastAPI, HTTPException, Depends
from pydantic import BaseModel

app = FastAPI()

class ItemIn(BaseModel):
    name: str
    price: float

DB: dict[int, dict] = {}

@app.post("/items", status_code=201)
def create(item: ItemIn):
    id = len(DB) + 1
    DB[id] = item.model_dump()
    return {"id": id, **DB[id]}

@app.get("/items/{id}")
def read(id: int):
    if id not in DB: raise HTTPException(404, "not found")
    return DB[id]
# run: uvicorn main:app --reload   → docs at /docs
```
Learn: dependency injection (`Depends`), middleware, background tasks, `async def` endpoints, **streaming responses (SSE)**, auth dependencies, lifespan events, testing with `TestClient`. Docs: https://fastapi.tiangolo.com/tutorial/
Deployment: `uvicorn`/`gunicorn` workers → container → behind a reverse proxy/load balancer.

## 7. JSON deeply
- Types: object, array, string, number, boolean, null. No comments, no trailing commas, no dates (use ISO-8601 strings), no NaN.
- **JSON Schema** describes the shape of JSON. It's the **language of LLM tool definitions & structured outputs** (Phase 4). Pydantic → `model_json_schema()`.
- Handling messy JSON: `json.JSONDecodeError`; LLMs sometimes emit invalid JSON → validate + repair/retry.
- **JSONL**: one JSON per line, streamable, append-friendly.
- Tools: `jq`, JSONPath, JSON Patch.

## 8. Webhooks & event-driven
- Receiver should reply **fast (2xx)** then process async (queue). Verify HMAC signature; handle duplicates (idempotency) and out-of-order delivery.
- Queues/streams: Redis, RabbitMQ, SQS, Kafka, Celery/RQ, Temporal (durable workflows). Useful for long-running LLM/agent jobs.

## 9. Web security basics (OWASP)
Injection (SQL!), broken auth, SSRF, XSS/CSRF (browser), secrets exposure, insecure deserialization, rate-limiting, input validation, least privilege, TLS everywhere. *LLM-specific* risks (prompt injection) come in Phase 7.

## 10. Debugging toolbox
`curl -v`, `httpie`, Postman/Bruno/Insomnia, browser DevTools Network tab, `mitmproxy`, request IDs, `httpbin.org`, `webhook.site`, `ngrok` (expose localhost for webhook testing).

Next → [[07-SQL-and-Databases]]
