# ADR-003: Event-Driven Messaging with Kafka

**Status:** Accepted
**Date:** 2026-10-09
**Note:** This is the early, architecture-level version of this decision, written in Week 1 before any Kafka hands-on work. It will be revisited with implementation detail (partitioning, delivery guarantees, DLQs) in Weeks 7–8 once the broker is actually running.

## Context

Once a claim is submitted, several things need to happen that don't require an immediate response to the caller: documents need processing, agents need to run, notifications need to go out, and a decision letter eventually needs generating. These steps are also owned by different services (Claims, Documents, Agent Orchestrator, Insights, Notification Worker). The two realistic options are synchronous HTTP calls between every pair of services that needs to react to a claim event, or an asynchronous event backbone that services publish to and consume from independently. Kafka is also an explicit learning requirement for this project, which is a factor in its own right, alongside the architectural fit.

## Decision

Use Kafka (KRaft mode, no ZooKeeper) as the backbone for everything downstream of claim submission, keeping only the one call that genuinely needs an immediate answer — Claims checking coverage with Policy — as synchronous HTTP. Claims publishes events like `ClaimSubmitted` and `ClaimDecided` via a transactional outbox; every other service (Documents, Agent Orchestrator, Insights, Notification Worker) reacts to events instead of being called directly.

Most of the downstream work is naturally "fire and move on" rather than "ask and wait": nobody calling `POST /claims` needs to block until fraud scoring or a decision letter is finished. Coupling every one of those steps to a direct HTTP call from Claims would mean Claims needs to know about, call, and handle the failure of every downstream service — exactly the kind of tight coupling bounded contexts are meant to avoid. An event backbone lets each downstream service own its own trigger and its own failure handling (retry, DLQ) without Claims knowing any of them exist.

## Consequences

**Benefits**
- Claims stays decoupled from every downstream consumer; adding a new consumer of `ClaimSubmitted` later requires no change to Claims at all.
- Natural fit for the services that are inherently background work (notifications, decision letters, agent runs) rather than request/response.
- The outbox pattern (Week 7) gives transactional consistency between "claim saved" and "event published" without a distributed transaction.
- Directly satisfies the project's Kafka learning goals: consumer groups, partitioning, rebalancing, exactly-once-adjacent patterns.
- Scales independently — Notification Worker or the agent pipeline can be scaled (and later autoscaled with KEDA) based on queue depth, with no change to Claims.

**Costs**
- Eventual consistency: there is a window where a claim is "submitted" but its documents haven't been processed yet, which has to be modeled explicitly (status fields, UI state) rather than assumed away.
- Operational overhead of running and later securing a Kafka cluster (schema registry, KRaft, eventually Strimzi on AKS) — meaningfully more than calling an HTTP endpoint.
- Debugging a flow now means tracing across a broker, not just following a call stack; distributed tracing (OpenTelemetry, Week 24) becomes necessary rather than optional.
- At-least-once delivery is the realistic default, which pushes idempotent-consumer design (Week 7) onto every consumer, not just Claims.
- For the one place a synchronous answer actually is needed (Claims→Policy coverage check), Kafka would be the wrong tool — hence that path deliberately stays HTTP rather than being forced into events for consistency's sake.

## Alternatives considered

- **Synchronous HTTP everywhere** — simpler to reason about step by step, but would couple Claims to every downstream service and make background work (agent runs, notifications) block on the request path or require Claims to fire-and-forget its own HTTP calls with no delivery guarantee. Rejected.
- **RabbitMQ** — a more traditional message queue, simpler operationally than Kafka, fine for task distribution. Rejected because the project's requirements call for Kafka specifically (consumer groups, partitioning, log compaction, replay), none of which a queue-oriented broker demonstrates as directly.
- **Azure Service Bus** — managed, less operational burden, strong .NET integration. Rejected for the same reason as RabbitMQ: it would reduce the hands-on Kafka learning this project is built around, even though it would likely be the pragmatic choice for a real production system without that constraint. (This trade-off — Kafka vs. RabbitMQ vs. Service Bus — is revisited with more evidence in the Week 7 ADR once the broker is actually running.)
