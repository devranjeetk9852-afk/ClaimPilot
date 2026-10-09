# ClaimPilot — instructions for Claude Code

## What this project is
AI-powered insurance claims platform, built as a learning capstone. A customer files a
claim with documents; agents check coverage and fraud risk; a human adjuster decides.
Full context: docs/architecture/ and docs/adr/.

## Stack and versions
- .NET 10 (C#, ASP.NET Core, Worker Services), pinned via global.json (Week 2)
- Python 3.12 managed with uv; FastAPI for the Document Service
- React + TypeScript with pnpm under web/
- PostgreSQL (EF Core, Dapper, pgvector), Kafka in KRaft mode, Azure Blob, Azure AI Foundry
- Auth: Entra ID / Managed Identity / Workload Identity. Never put secrets in code or config.

## Repo rules
1. Services never import each other's code. They share only libs/ and contracts/.
2. libs/ holds contracts and plumbing only, never business logic.
3. A PR touches one service where possible. Change contracts/ and the consumers in one PR only when a contract changes.
4. Every service builds and runs on its own.
5. contracts/ (OpenAPI and event schemas) is the source of truth for every cross-service interface.

## Conventions
- Conventional Commits with the service as scope: feat(claims): ..., chore(repo): ...
- One branch and one PR per issue. Squash merge to main.
- Record every significant decision as an ADR in docs/adr/ (NNN-title.md).
- Learning log: docs/learning/week-NN.md, updated every day.

## How to work with me (Claude Code)
- Explain before you change anything. Tell me what you're going to do and why first.
- On concept weeks (2–4, 7–8, 13, 16), quiz me and review my code. Don't write the core logic.
- For boilerplate (Dockerfiles, Helm, CI YAML), draft it, then walk me through every line.
- Before I merge, flag what a senior reviewer would question in the PR.

## Do not
- Commit secrets, connection strings, keys or .env files.
- Add a dependency without saying why it's needed.
- Change the architecture without an ADR.

## Commands
<!-- Fill these in as each service arrives, e.g.:
     services/claims: dotnet test
     services/documents: uv run pytest -->