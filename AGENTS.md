## Tracer Development Reference

## Build and Run

- Build: `make install` (sets up the environment via `uv sync` and installs this repo in editable mode)
- Run: **`uv run opensre …`** from the repo root — preferred approach, uses this checkout even if another `opensre` is on your `PATH`.
- Python commands: **`uv run python …`**

## Code Style

- 100-character line length (enforced by ruff and `.editorconfig`)
- 4-space indentation for Python; 2-space for YAML/JSON/TOML; tabs for Makefiles
- `from __future__ import annotations` on every Python file
- Absolute imports only: `from app.tools.base import BaseTool`
- Type hints on all function parameters and return types
- `TypedDict` for graph state, Pydantic `StrictConfigModel` for configs
- One clear purpose per file (separation of concerns); DRY

## Quality Commands

```bash
make lint          # ruff check app/ tests/
make format-check  # ruff format --check (read-only; CI enforces this)
make format        # ruff format app/ tests/ (write fixes locally)
make typecheck     # mypy app/
make test-cov      # pytest with coverage
make check         # lint + format-check + typecheck + test-full
```

## Before Push

Follow the mandatory checklist in [CI.md](CI.md) — it is the source of truth for push/PR readiness. Do not skip required checks.

PR checklist: link the issue (`Fixes #123`), all local checks pass, tests added, docs updated if behavior changed, self-reviewed, breaking changes called out.

## 1. Repo Map

| Path | What it does |
| --- | --- |
| `app/` | Core agent logic, CLI, tools, integrations, services, pipeline, and runtime state |
| `tests/` | Unit, integration, synthetic, deployment, e2e, chaos, and support tests |
| `docs/` | User-facing documentation, integration guides, and docs-site assets |
| `.github/` | CI workflows, issue templates, PR template, and repository automation |
| `pyproject.toml` | Python project metadata, dependencies, tooling, and package settings |
| `Makefile` | Canonical local automation for install, test, verify, deploy, and cleanup |
| `CI.md` | Mandatory pre-push checklist — agents MUST follow before pushing |
| `CONTRIBUTING.md` | Contribution workflow, branch/PR guidance, and quality expectations |
| `SETUP.md` | Machine setup (all platforms, Windows, MCP/OpenClaw, troubleshooting) |
| `docs/DEVELOPMENT.md` | Contributor workflows: CI parity, dev container, benchmark, deployment |
| `docs/investigation-tool-calling.md` | Investigation ReAct tool schemas and LLM invoke payloads (all providers) |

`app/` key subdirectories and their nested guides:

- `app/tools/` — Tool registry, base classes, `@tool` decorator, per-tool packages. Guide: `app/tools/AGENTS.md`
- `app/integrations/` — Integration config, verification, catalog, and store. Guide: `app/integrations/AGENTS.md`
- `app/integrations/llm_cli/` — Subprocess-backed LLM CLIs (Codex, etc.). Guide: `app/integrations/llm_cli/AGENTS.md`
- `app/pipeline/` — Investigation orchestration and runner entry points. Guide: `app/pipeline/AGENTS.md`
- `app/services/` — Reusable API clients and adapters. Guide: `app/services/AGENTS.md`
- `app/cli/interactive_shell/` — REPL loop, slash commands, routing, UI. Guide: `app/cli/interactive_shell/AGENTS.md`
- `app/agent/` — Alert extraction, investigation loop, chat, and result parsing
- `app/state/` — `AgentState` TypedDict and state factories
- `app/delivery/` — Report rendering and external delivery
- `app/guardrails/` — Guardrail rules, evaluation engine, and CLI bindings
- `app/analytics/` — Analytics event plumbing and install helpers
- `app/auth/` — JWT and authentication helpers
- `app/masking/` — Masking utilities for redacting sensitive content
- `app/utils/` — Cross-cutting utility helpers

`tests/` key subdirectories:

- `tests/tools/`, `tests/integrations/`, `tests/services/` — Unit and mock tests by feature area
- `tests/cli/` — CLI behavior, smoke tests, and command wiring
- `tests/e2e/` — Live end-to-end scenarios against real services (`tests/AGENTS.md` — e2e RCA principles)
- `tests/synthetic/` — Fixture-driven synthetic RCA scenarios with no live infrastructure
- `tests/deployment/`, `tests/chaos_engineering/` — Deployment and chaos lab tests

## 2. Rules (if X → do Y)

- If core agent or pipeline logic changes → run `make test-cov` and `make typecheck`.
- If a new feature is shipped (tool, CLI command, pipeline behavior, integration) → add a `docs/` page before the PR is opened.
- If a new `docs/` page is added or renamed → register it in `docs/docs.json` under the correct `pages` array in the same PR.
- If an existing feature changes behavior, flags, or config shape → update the relevant `docs/` page in the same PR.
- If a tool's API or schema changes → update `docs/` and `tests/tools/`. For investigation LLM tool-calling see [docs/investigation-tool-calling.md](docs/investigation-tool-calling.md).
- If an integration changes → update `tests/integrations/` and run `make verify-integrations`.
- If adding a new integration → follow `app/integrations/AGENTS.md` checklist before opening the PR.
- If adding a new tool → follow `app/tools/AGENTS.md` checklist before opening the PR.
- If adding new tests → always place them in `tests/`, never in `app/`.
- If CI-only tests are added → mark with the right pytest marker or place in e2e/synthetic/chaos folder.
- If investigation branching or loop behavior changes → update `app/pipeline/pipeline.py` and tests.
- If pushing or creating a PR → follow the full pre-push checklist in [CI.md](CI.md).
- Routing live tests: always run with live coverage enabled. Do not use `-k "not live_llm"` deselection. Fix failures by correcting planner/tool behavior, not by bypassing live checks.

## 3. Footguns

- **Secrets:** Never commit `.env` — use `.env.example`. Use read-only credentials for production integrations.
- **Docs navigation:** Adding `.mdx` under `docs/` is not enough — register the path in `docs/docs.json` or the page is unreachable from the sidebar.
- **Tool schemas:** Draft-07 JSON Schema that passes local checks can fail the LLM API at invoke time because all investigation tools are sent together. See [docs/investigation-tool-calling.md](docs/investigation-tool-calling.md).
- **CI-only tests:** Kubernetes, EKS, and chaos paths require live infrastructure — do not expect them to pass locally.
- **Docker:** Grafana local stack and Chaos Mesh workflows require a running Docker daemon.
- **Vendored deps:** Manage Python dependencies in `pyproject.toml`; do not vendor libraries without a strong reason.
