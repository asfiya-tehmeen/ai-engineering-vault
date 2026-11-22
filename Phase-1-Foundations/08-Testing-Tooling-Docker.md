---
tags: [phase-1, testing, docker, tooling]
---
# 08 — Testing, Tooling, Docker & CI

## 1. Environments & dependencies
- A **virtual environment** isolates a project's packages. Never `pip install` globally.
- **`uv`** (Astral) is the fast modern tool: `uv init`, `uv add httpx`, `uv run pytest`, `uv sync`, lockfile `uv.lock`, manages Python versions. Docs: https://docs.astral.sh/uv/
- Alternatives: `venv + pip`, `poetry`, `pdm`, `conda/mamba` (common in data science).
- **Pin & lock** dependencies for reproducibility. `pyproject.toml` is the single config file.

## 2. Code quality
- **ruff** — linter + formatter (replaces flake8/black/isort). `ruff check --fix`, `ruff format`.
- **mypy / pyright** — static type checking.
- **pre-commit** hooks run these before each commit.
- Code review habits: small PRs, descriptive titles, tests included.

## 3. Testing with pytest
```python
# tests/test_math.py
import pytest
from mypkg.calc import divide

def test_divide():
    assert divide(6, 3) == 2

def test_divide_by_zero():
    with pytest.raises(ZeroDivisionError):
        divide(1, 0)

@pytest.mark.parametrize("a,b,exp", [(4,2,2), (9,3,3), (1,1,1)])
def test_many(a, b, exp): assert divide(a,b) == exp

@pytest.fixture
def client(): 
    with TestClient(app) as c: yield c
```
- **Test pyramid:** many unit tests → some integration → few end-to-end.
- **Arrange–Act–Assert.** Test behavior, not implementation. One reason to fail per test.
- **Mocking:** `unittest.mock`, `pytest-mock`, `monkeypatch`. Mock **at the boundary** (HTTP, DB, LLM) — never call paid/non-deterministic APIs in unit tests. `respx`/`responses`/`vcrpy` record/replay HTTP.
- `pytest-asyncio` / `anyio` for async tests. `hypothesis` for property-based testing. `pytest-cov` for coverage (coverage ≠ quality).
- **TDD** loop: red → green → refactor.
- 🔑 **Bridge to AI:** LLM outputs are non-deterministic, so you add **evals** (Phase 7) on top of ordinary tests; the ordinary tests still cover parsing, tool functions, retries, and routing.

## 4. Logging, config, secrets
- `logging` (not `print`) with levels; JSON logs in prod.
- 12-factor config: **env vars** → `pydantic-settings` (`BaseSettings`).
- `.env` locally (git-ignored); **secret managers** in prod. Rotate keys; least privilege; never log keys.

## 5. Docker
**Idea:** package app + OS libs + deps into an **image**; run as a **container** (isolated process). "Works on my machine" → works everywhere.
```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY pyproject.toml uv.lock ./
RUN pip install uv && uv sync --frozen --no-dev
COPY src ./src
ENV PYTHONUNBUFFERED=1
EXPOSE 8000
CMD ["uv","run","uvicorn","src.main:app","--host","0.0.0.0","--port","8000"]
```
```bash
docker build -t myapp .
docker run -p 8000:8000 --env-file .env myapp
docker ps ; docker logs -f <id> ; docker exec -it <id> sh
docker compose up           # multi-container: app + postgres + redis
```
Concepts: image layers & cache (order Dockerfile from least→most frequently changing), `.dockerignore`, volumes (persist data), networks, multi-stage builds (small images), non-root user, healthchecks, registries (Docker Hub, GHCR, ECR), image scanning.
**Advanced:** Kubernetes (pods, deployments, services, ingress, HPA), Helm, container security, GPU containers (NVIDIA Container Toolkit). Know K8s concepts; deep ops is Phase 7/8.
Docs: https://docs.docker.com/get-started/

## 6. CI/CD
- **CI** (continuous integration): on every push run lint → type check → tests → build. **CD**: deploy automatically.
```yaml
# .github/workflows/ci.yml
name: ci
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v5
      - run: uv sync
      - run: uv run ruff check . && uv run pytest -q
```
- Concepts: artifacts, caching, matrix builds, secrets in CI, environments, branch protection, deploy previews, rollbacks, blue/green & canary deploys, feature flags.

## 7. Cloud primer (broad strokes)
- **Compute**: VMs (EC2), containers (ECS/Cloud Run/Fargate), serverless (Lambda/Cloud Functions), K8s.
- **Storage**: S3/GCS/Blob. **DB**: RDS/Cloud SQL. **Secrets**, **IAM** (roles/policies — a frequent FDE headache), **VPC** (private networks), **load balancers**, **CDN**.
- Get hands-on with one provider's free tier (AWS or GCP). AI services: AWS Bedrock, Azure OpenAI/AI Foundry, Google Vertex AI.
- **IaC**: Terraform / Pulumi / CloudFormation.

## 8. Observability basics
Logs, metrics, traces (**OpenTelemetry**), dashboards (Grafana), alerts, error tracking (Sentry), SLOs. LLM-specific observability comes in Phase 7.

Next → [[09-NumPy-Pandas-Data-Wrangling]]
