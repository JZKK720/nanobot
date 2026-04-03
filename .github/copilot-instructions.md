# nanobot — Workspace Instructions

> Ultra-lightweight personal AI assistant framework (Python 3.11+/3.12). MIT.  
> Repo: HKUDS/nanobot | Fork: JZKK720/nanobot | Current version: `0.1.4.post6`

---

## Build & Test

```bash
# Install dev dependencies
pip install -e ".[dev]"

# Run full test suite
pytest

# Lint / format (Ruff)
ruff check nanobot/
ruff format nanobot/

# Build Docker image (single image, two services)
docker compose build

# Start gateway (port 18790)
docker compose up nanobot-gateway

# One-off CLI command via Docker
docker compose run --rm nanobot-cli status
# Override command:
docker compose run --rm nanobot-cli nanobot <args>
```

## Architecture

```
nanobot/
├── agent/          # Core loop, runner, hooks, memory, subagent, skills
│   └── tools/      # Built-in tool registry
├── providers/      # LLM provider adapters (Anthropic, OpenAI, Azure, Codex, …)
├── channels/       # Chat platform connectors (Telegram, Slack, DingTalk, Feishu, …)
├── bus/            # Internal event bus and message queue
├── session/        # Per-user session state
├── cron/           # Scheduled task engine (croniter-based)
├── heartbeat/      # Periodic HEARTBEAT.md task runner
├── command/        # Built-in slash-command routing
├── config/         # Config schema (JSON), loader, path resolution
├── cli/            # Typer CLI (gateway, status, channels login …)
├── api/            # Optional REST API (aiohttp extra)
└── templates/      # Default SOUL.md, TOOLS.md, USER.md, AGENTS.md, HEARTBEAT.md
bridge/             # WhatsApp bridge (Node.js / TypeScript, built during Docker build)
skills/             # Bundled agent skills (github, memory, weather, cron, …)
```

Key entry points:
- `nanobot/nanobot.py` — `Nanobot` programmatic facade
- `nanobot/agent/loop.py` — `AgentLoop` (tool-calling iteration engine)
- `nanobot/agent/runner.py` — `AgentRunSpec` and per-turn execution
- `nanobot/cli/commands.py` — CLI surface (`gateway`, `status`, `channels`)

## Docker Compose Services

| Service | Profile | Command | Port |
|---|---|---|---|
| `nanobot-gateway` | *(default)* | `gateway` | `18790` |
| `nanobot-cli` | `cli` | `status` (overridable) | — |

Both services share the same image built from `Dockerfile` and mount `~/.nanobot` at `/root/.nanobot`.

## Config

Config lives at `~/.nanobot/config.json`. Generate with `nanobot onboard` or copy schema from `nanobot/config/schema.py`.  
Optional extras: `api`, `wecom`, `weixin`, `matrix`, `discord`, `langsmith`.

## Conventions

- **No litellm** — removed in `v0.1.4.post6` due to supply-chain poisoning. Use native `openai` / `anthropic` SDKs.
- **Branching**: bug fixes → `main`; new features / refactors → `nightly`. PRs cherry-picked from nightly to main ~weekly.
- **Code style**: small, calm, readable. Avoid unnecessary abstractions. Ruff enforced.
- **Tests**: `pytest-asyncio` + `pytest-cov`. Async tests use `@pytest.mark.asyncio`.
- **Security**: see [SECURITY.md](../SECURITY.md). Network filtering in `nanobot/security/network.py`.

## Sync with Upstream

Fork `origin` = JZKK720/nanobot | Upstream = HKUDS/nanobot

```bash
git fetch upstream
# Count commits behind upstream/main:
git rev-list --count HEAD..upstream/main
# Merge upstream into local main (prefer-fork workflow: cherry-pick instead of merge):
git cherry-pick <sha>
```

## Docs

- [CONTRIBUTING.md](../CONTRIBUTING.md) — branching model, code style, PR process
- [docs/CHANNEL_PLUGIN_GUIDE.md](../docs/CHANNEL_PLUGIN_GUIDE.md) — add a new chat channel
- [docs/PYTHON_SDK.md](../docs/PYTHON_SDK.md) — programmatic SDK usage
- [SECURITY.md](../SECURITY.md) — security policy
