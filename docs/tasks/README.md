# Tasks

Implementation work is broken into milestones, and each milestone into small steps. Every step
ends in a working, tested state and its own commit.

## How a step works

1. **Read the task file** — goal, reasoning, instructions, and acceptance criteria.
2. **Build it.** Hints come before full solutions.
3. **Review.** The step is reviewed against its acceptance criteria before it is marked done.
4. **Commit** using [Conventional Commits](https://www.conventionalcommits.org).

Task files for later steps are written when the previous step is done, so each one can build on
what was actually implemented.

## Naming

Task files are named `NNN_<task_name>.md`: a three-digit number that keeps counting across
milestones, then the task name in snake_case — for example `001_repo_foundations.md`.

## Milestone M0 — Foundation

Goal: a production-grade skeleton with no product features yet. Everything runs: tests pass,
`docker compose up` starts the stack, and CI is green.

| # | Step | You'll learn | Status |
|---|---|---|---|
| 001 | [Repo foundations](001_repo_foundations.md) — uv project, `src/` layout, license, git hygiene | Modern Python packaging | ☐ |
| 002 | Quality gates — ruff, strict mypy, pytest, pre-commit, justfile | Automated standards | ☐ |
| 003 | Settings — typed config from environment variables, secrets hidden | 12-factor config | ☐ |
| 004 | Logging — structured JSON logs, request IDs, secret redaction | Observability basics | ☐ |
| 005 | App factory — three services from one codebase, `/healthz`, standard error format | FastAPI architecture | ☐ |
| 006 | Metrics — Prometheus middleware, `/metrics` | Low-cardinality metrics | ☐ |
| 007 | Database — async SQLAlchemy, Alembic, locked migrations, `/readyz` | Safe schema management | ☐ |
| 008 | CLI — `recomet serve`, `migrate`, `worker`, `version` | Operator-facing tools | ☐ |
| 009 | Celery — queues, safe serialization, logging | Background job reliability | ☐ |
| 010 | Ports and contract tests — first swappable interface | Clean architecture in practice | ☐ |
| 011 | Docker — multi-stage non-root image, Compose with health checks | Container best practices | ☐ |
| 012 | CI/CD — GitHub Actions, Dependabot, CodeQL, image publishing | Supply-chain security | ☐ |
| 013 | Repo documents — README, CONTRIBUTING, SECURITY, templates | Open-source maintainer standards | ☐ |
