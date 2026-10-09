# ADR-001: Microservices vs. a Modular Monolith

**Status:** Accepted
**Date:** 2026-10-09

## Context

ClaimPilot is a 27-week, 1.5-hour-a-day capstone built by a single engineer, with explicit learning goals around Kafka, Kubernetes/AKS, polyglot services (C#, Python, React), and comparing two different Microsoft AI frameworks (Semantic Kernel vs. Agent Framework) side by side. The functional requirements (claim intake, coverage check, fraud scoring, agent recommendation, human approval, decision letters) could be built as a single well-structured ASP.NET Core application with internal module boundaries — a modular monolith — with meaningfully less operational overhead.

## Decision

Build ClaimPilot as microservices from the start: API Gateway, Claims, Policy, Document Service, Agent Orchestrator, Insights Service, two MCP servers, and a Notification Worker, each independently deployable, each owning its own data.

The deciding factor is not the business problem — a modular monolith would serve the business requirements just as well, arguably better for a team of one. The deciding factor is the learning plan: Kafka consumer groups and rebalancing, Kubernetes service-to-service networking, KEDA scaling on queue depth, and independent CI/CD pipelines per service are all explicit requirements, and none of them are learnable in a meaningful way inside a single deployable unit. A Python document-processing service and a React portal also need their own runtimes regardless of how the C# code is organized, which pushes toward service boundaries anyway.

## Consequences

**Benefits**
- Forces hands-on practice with the distributed-systems topics the project exists to teach (Kafka, K8s, service boundaries, contract-first APIs).
- Each service can be deployed, scaled and reasoned about independently — directly interview-relevant.
- Polyglot boundaries (C# vs. Python) are natural at the service edge instead of awkward in-process interop.
- Enables the explicit Semantic Kernel vs. Agent Framework comparison as two separate services rather than two code paths in one app.

**Costs**
- Significantly more operational surface for a solo developer: nine services, each with its own Dockerfile, Helm chart, health checks and CI pipeline.
- Distributed debugging from day one — a bug can be a network issue, a serialization mismatch, or an event ordering problem instead of a stack trace.
- Eventual consistency and the outbox pattern are required even for simple flows that a monolith would handle in one transaction.
- More boilerplate per unit of business logic, especially early (Weeks 1–11) before the AI functionality — the "interesting" part — even exists.
- Real risk of over-engineering relative to the actual business problem; this is accepted explicitly because the project's purpose is the learning, not the business outcome.

## Alternatives considered

- **Modular monolith** — single deployable, strong internal module boundaries (namespaces/projects), in-process calls instead of HTTP/Kafka. Rejected because it would not exercise Kafka or multi-service Kubernetes deployment, both hard requirements of the learning plan, and because Python/React still need separate processes regardless.
- **Monolith + a couple of true services** (e.g., only Document Service and Agents split out, everything else in one app) — a middle ground. Rejected for the same reason: it would under-deliver on the Kubernetes and CI/CD-per-service goals, and the services most worth splitting (Claims, Policy) are exactly the ones a "minimal split" approach would keep together.
