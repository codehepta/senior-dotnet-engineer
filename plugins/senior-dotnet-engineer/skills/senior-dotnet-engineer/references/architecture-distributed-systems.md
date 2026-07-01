# Architecture and Distributed Systems

## Contents

1. Domain boundaries
2. Deployment boundaries
3. CQRS and event sourcing
4. Reliable messaging
5. Contract evolution
6. Decision checklist

## Domain Boundaries

Model behavior, not database tables. Put invariants inside aggregates, use value objects for concepts with validation and equality, and keep aggregate transactions small. Reference other aggregates by identity. Domain events describe completed domain facts; handlers must not make an in-memory transaction appear atomically distributed.

Use bounded contexts where language, rules, data ownership, or change cadence differ. Define context maps and translation boundaries. Do not share persistence entities across contexts as a shortcut.

## Modular Monolith or Microservices

Default to a modular monolith with enforced module APIs, internal data ownership, dependency direction, and architecture tests. It preserves local transactions and operational simplicity while allowing later extraction.

Choose microservices only when evidence requires independent deployment, scaling, ownership, compliance isolation, technology constraints, or fault containment. Price in network failure, latency, duplicated data, eventual consistency, observability, version skew, incident response, and platform ownership. A distributed monolith pays these costs without gaining autonomy.

When extracting, first establish a stable boundary, then separate data ownership, add compatible contracts and observability, migrate traffic incrementally, and retain rollback.

## CQRS and Event Sourcing

Use CQRS when command invariants and read shapes materially differ. CQRS does not require separate services, databases, or event sourcing. Keep simple CRUD simple.

Use event sourcing only when immutable history, temporal reconstruction, audit semantics, or domain behavior justifies the cost. Define event identity, ordering, versioning, upcasting, snapshot policy, replay safety, side-effect suppression, privacy/retention handling, and projection rebuild operations before adoption.

Do not expose domain events directly as public integration contracts. Translate them into deliberately versioned integration events.

## Reliable Messaging

Assume at-least-once delivery unless the infrastructure and end-to-end protocol prove otherwise. Make consumers idempotent with a stable message/idempotency key and durable deduplication inside the same transaction as the business effect.

Select broker semantics before vendor features. RabbitMQ commonly fits routed work queues and explicit acknowledgements; Kafka fits partitioned durable logs, replay, and stream processing; Azure Service Bus fits managed queues/topics with Azure identity and dead-letter integration. Confirm the repository's client version and the platform's actual delivery, ordering, transaction, lock/visibility, partition, retention, and retry behavior rather than assuming the products are interchangeable.

- Use an outbox to atomically persist state and messages, then publish with observable retries.
- Use an inbox/deduplication record when duplicate consumption can repeat effects.
- Define ordering scope; avoid global ordering when per-aggregate ordering suffices.
- Bound retry attempts and delay. Route poison messages to a dead-letter path with reason, payload reference, alerting, inspection, and replay controls.
- Use sagas/process managers for multi-step workflows. Persist state, correlate messages, make steps idempotent, and define compensation semantics rather than pretending rollback is global.
- Apply backpressure and partitioning. Monitor lag, age, retries, duplicates, dead letters, and processing duration.

Exactly-once marketing does not eliminate application-level duplicate effects.

## Contract Evolution

Design for version skew because deployments are not atomic:

- HTTP/JSON: prefer additive optional fields; preserve meanings and defaults; deploy tolerant readers first.
- Protobuf: never reuse field numbers; reserve removed fields; avoid incompatible wire-type changes.
- Events: include stable event type, schema version, event identity, occurred time, producer, and correlation/causation where useful. Version semantics deliberately.
- Databases: use expand/migrate/contract. Add compatible schema, deploy dual-capable code, backfill observably, switch reads/writes, then remove old shape later.

Contract tests supplement—not replace—consumer coordination and production telemetry.

## Decision Checklist

- What owns each invariant and datum?
- What consistency is required, and where is eventual consistency acceptable?
- How are duplicates, reordering, partial failure, and replay handled?
- Are timeouts, retry budgets, dead-letter behavior, and backpressure explicit?
- Can old and new producers, consumers, schemas, and binaries coexist during rollout?
- What metrics and traces reveal stuck workflows or compatibility failures?
- Is the operational complexity justified by current evidence?
