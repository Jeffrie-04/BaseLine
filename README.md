# BaseLine

Self-hosted fitness tracker that replaces Google Health, Bevel and Liftcard: sleep, strain, recovery and lifting progress in one place, running on my homelab.

**Status:** Phase 0, project setup. The backend serves a health check.

## Quickstart

**Prerequisite:** [uv](https://docs.astral.sh/uv/) (`brew install uv`). uv installs Python 3.12 and all dependencies automatically.

```bash
git clone https://github.com/Jeffrie-04/BaseLine.git
cd BaseLine/backend
uv run fastapi dev app/main.py
```

Open http://127.0.0.1:8000/health → `{"status":"ok"}`. Interactive API docs: http://127.0.0.1:8000/docs

## Development

```bash
uv tool install pre-commit && pre-commit install   # from the repo root: lint + format on every commit
cd backend && uv run pytest                         # run tests
```

## Project structure

```
backend/    FastAPI API (Python 3.12, uv)
frontend/   Web UI (coming soon)
infra/      Docker / deployment config (coming soon)
docs/       Design docs, decisions, dev log
```

## Workflow

- One issue → one branch (`feat/12-log-sets`) → one PR. `main` is protected.
- Commits follow [Conventional Commits](https://www.conventionalcommits.org/) (`feat:`, `fix:`, `chore:`, `docs:`).
- PRs are squash-merged.

## License

MIT
