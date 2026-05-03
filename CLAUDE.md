# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run

```bash
# All services (CPU only)
./start_services.sh

# All services including GPU
./start_services.sh --gpu

# Specific services
./start_services.sh hmm megadna

# Stop all
./start_services.sh --stop

# Docker Compose
docker compose up                   # CPU services
docker compose --profile gpu up     # include GPU services

# Individual service
cd <service-name> && uv sync && uv run uvicorn service:app --host 0.0.0.0 --port <port>
```

## Tests

Each service has its own test suite with mocked dependencies (no GPU/model/DB needed):
```bash
cd <service-name> && uv run pytest tests/ -v
```

## Service Map

| Service | Port | Python | GPU | External deps |
|---------|------|--------|-----|---------------|
| megadna-service | 8000 | 3.12 | ~2GB VRAM | megaDNA model weights (`external/megaDNA`) |
| hmm-service | 8002 | 3.12 | no | PHROGs HMM database |
| bacphlip-service | 8003 | 3.9 | no | HMMER3 binary (`apt-get install hmmer`) |
| deeppl-service | 8004 | 3.10+ | ~1GB VRAM | DNABERT model weights |
| phabox-service | 8005 | 3.10+ | no | PhaBOX2 conda env + database (`download_db.sh`) |

## Architecture

- Each service is a standalone FastAPI app in `service.py` with `settings.py` (pydantic-settings, env var prefix `<SERVICE>_*`). **Exception:** hmm-service uses `src/hmm_service/main.py` + `config.py` instead of the root-level `service.py`/`settings.py` pattern.
- Services and `phages_dataset` clients define their request/response models locally. The unused shared `contracts/` package was removed to avoid implying guarantees that are not wired into the services.
- **Config loading priority**: env vars > `.env` file > `config.yaml` > defaults in `settings.py`.
- All services expose `GET /health` for health checks.
- `start_services.sh` writes PIDs to `.service_pids` for process management.

## Key Constraints

- **bacphlip-service requires Python 3.9** due to pinned `scikit-learn==0.24.2` dependency from the upstream bacphlip package.
- **megadna-service** depends on a local package at `external/megaDNA` (referenced via `[tool.uv.sources]`).
- **phabox-service** additionally requires a conda environment (`environment.yaml`) for binary tools (DIAMOND, BLAST, MCL, prodigal-gv).

<!-- gitnexus:start -->
# GitNexus — Code Intelligence

This project is indexed by GitNexus as **phages-services** (1078 symbols, 1431 relationships, 9 execution flows). Use the GitNexus MCP tools to understand code, assess impact, and navigate safely.

> If any GitNexus tool warns the index is stale, run `npx gitnexus analyze` in terminal first.

## Always Do

- **MUST run impact analysis before editing any symbol.** Before modifying a function, class, or method, run `gitnexus_impact({target: "symbolName", direction: "upstream"})` and report the blast radius (direct callers, affected processes, risk level) to the user.
- **MUST run `gitnexus_detect_changes()` before committing** to verify your changes only affect expected symbols and execution flows.
- **MUST warn the user** if impact analysis returns HIGH or CRITICAL risk before proceeding with edits.
- When exploring unfamiliar code, use `gitnexus_query({query: "concept"})` to find execution flows instead of grepping. It returns process-grouped results ranked by relevance.
- When you need full context on a specific symbol — callers, callees, which execution flows it participates in — use `gitnexus_context({name: "symbolName"})`.

## Never Do

- NEVER edit a function, class, or method without first running `gitnexus_impact` on it.
- NEVER ignore HIGH or CRITICAL risk warnings from impact analysis.
- NEVER rename symbols with find-and-replace — use `gitnexus_rename` which understands the call graph.
- NEVER commit changes without running `gitnexus_detect_changes()` to check affected scope.

## Resources

| Resource | Use for |
|----------|---------|
| `gitnexus://repo/phages-services/context` | Codebase overview, check index freshness |
| `gitnexus://repo/phages-services/clusters` | All functional areas |
| `gitnexus://repo/phages-services/processes` | All execution flows |
| `gitnexus://repo/phages-services/process/{name}` | Step-by-step execution trace |

## CLI

| Task | Read this skill file |
|------|---------------------|
| Understand architecture / "How does X work?" | `.claude/skills/gitnexus/gitnexus-exploring/SKILL.md` |
| Blast radius / "What breaks if I change X?" | `.claude/skills/gitnexus/gitnexus-impact-analysis/SKILL.md` |
| Trace bugs / "Why is X failing?" | `.claude/skills/gitnexus/gitnexus-debugging/SKILL.md` |
| Rename / extract / split / refactor | `.claude/skills/gitnexus/gitnexus-refactoring/SKILL.md` |
| Tools, resources, schema reference | `.claude/skills/gitnexus/gitnexus-guide/SKILL.md` |
| Index, status, clean, wiki CLI commands | `.claude/skills/gitnexus/gitnexus-cli/SKILL.md` |

<!-- gitnexus:end -->
