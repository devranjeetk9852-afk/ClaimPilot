# ADR-002: PostgreSQL as the Primary Datastore

**Status:** Accepted
**Date:** 2026-10-09

## Context

ClaimPilot needs a relational store for Claims and Policy service data (structured, transactional, with foreign-key relationships between claims, claim items, status history and documents), and later a vector store for RAG over policy documents (Week 15). PostgreSQL is one of the project's explicit learning requirements. The realistic alternatives for the relational workload are SQL Server (idiomatic default in a .NET-heavy stack) and a document store such as MongoDB; for vectors, a dedicated vector database (e.g., Pinecone, Qdrant) is the common production alternative to `pgvector`.

## Decision

Use PostgreSQL as the single relational engine for every service that needs one (Claims, Policy, Document Service's chunk/embedding tables), accessed via EF Core (with Dapper for one hot read path) from C#, and SQLAlchemy async from Python. Add the `pgvector` extension in Week 15 so the same database also serves as the vector store for RAG, instead of introducing a separate vector database.

The data itself is naturally relational — claims, claim items, status history and policy coverage rules have clear keys and foreign-key relationships, which fits PostgreSQL (or SQL Server) better than a document store. Between PostgreSQL and SQL Server, PostgreSQL wins on three practical grounds for this project specifically: it is free to run in Docker/AKS without a licensing concern, it is the engine named in the project's own requirements, and `pgvector` lets the RAG work (Week 15) reuse the same database and connection patterns already built in Week 5, instead of standing up and learning a second, dedicated vector database under a separate time budget.

## Consequences

**Benefits**
- One engine to learn deeply (indexing, `EXPLAIN ANALYZE`, isolation levels, connection pooling) instead of splitting attention across two.
- No licensing cost, which matters for a project budgeted at 1.5 hours/day with its own Azure cost alerts.
- `pgvector` means the RAG implementation (Week 15) is "add an extension and an HNSW index," not a new infrastructure component, new SDK, and new deployment story.
- Works identically in Docker locally (Week 11) and as Azure Database for PostgreSQL Flexible Server in the cloud (Week 22) — one mental model end to end.
- Reinforces "database per service" — Claims and Policy get logically separate PostgreSQL databases rather than sharing a SQL Server instance, which is cleaner to demonstrate in interviews.

**Costs**
- `pgvector`'s HNSW index is good but not best-in-class next to a purpose-built vector database on very large corpora or high-QPS vector search — not a concern at this project's document volume, but worth being able to say explicitly in an interview.
- Less idiomatic than SQL Server in a pure-.NET shop, so some .NET-specific tooling assumptions (e.g., certain EF Core provider conveniences) don't apply as smoothly; Npgsql closes most of this gap.
- Running two engines (Postgres for OLTP, something else for vectors) was the "more standard" production pattern at larger scale — this decision explicitly trades some production realism for lower operational cost and shorter time-to-learning on RAG.

## Alternatives considered

- **SQL Server** — most idiomatic with EF Core and Azure, but licensing/cost overhead for a side project and not the engine the project's requirements call for. Rejected.
- **MongoDB** — would fit loosely structured document data, but claim/policy data is relational by nature (foreign keys, joins, transactions across claim items and status history), so a document model would fight the data rather than fit it. Rejected.
- **PostgreSQL + dedicated vector DB (e.g., Qdrant)** — more representative of some large-scale production setups. Rejected for this project: it would consume learning time on a second piece of infrastructure without a corresponding requirement, when `pgvector` covers the Week 15 goals directly.
