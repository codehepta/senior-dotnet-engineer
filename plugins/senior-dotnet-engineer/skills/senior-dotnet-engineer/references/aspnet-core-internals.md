# ASP.NET Core Internals

## Contents

1. Request pipeline
2. Hosting and background work
3. Dependency injection
4. HTTP resilience
5. gRPC and SignalR
6. Binding and formatting
7. Operational checks

## Request Pipeline

Treat middleware order as behavior. Map exception handling and request correlation early; place routing, CORS, authentication, authorization, rate limiting, endpoints, and response transformation according to their documented dependencies. A middleware that short-circuits must produce a complete response and preserve required diagnostics.

Do not read or buffer request/response bodies without size limits, ownership, disposal, encoding, and streaming implications. Keep authorization at the server boundary and before sensitive work. URL rewriting must avoid open redirects, loops, path confusion, and canonicalization bugs.

For custom authentication, separate credential extraction, validation, identity construction, challenge, and forbid behavior. Never log credentials or tokens. Use policy-based authorization for business access rules; test success, unauthenticated, forbidden, and cross-tenant cases.

## Hosting and Background Work

Use `BackgroundService`/`IHostedService` only for work owned by the process. Create scopes per unit of work; do not capture scoped services in the singleton worker. Bound queues and concurrency. Observe every task. Propagate `stoppingToken`, complete channels, checkpoint safely, and fit shutdown within the host termination budget.

For scheduled work, define single-instance versus multi-replica semantics, overlap policy, clock/time-zone handling, missed runs, and durable state. Use external scheduling when execution must survive process loss or coordinate across replicas.

## Dependency Injection

- Singleton: thread-safe, process lifetime, no captured scoped dependency.
- Scoped: request/unit-of-work lifetime; never use after scope disposal.
- Transient: lightweight and explicitly disposed when created outside container ownership.
- Factories: use when selection depends on runtime input while keeping dependencies explicit.
- Decorators: use for cross-cutting behavior without hiding order or double-applying policies.
- Keyed services: use for a finite, explicit strategy set; validate unknown keys and avoid turning DI into a service locator.

Validate the service provider in development/test and add tests for lifetime-sensitive registrations.

## HTTP Resilience

Use `IHttpClientFactory` or the repository's approved client abstraction. Every outbound operation needs an explicit deadline consistent with the caller's remaining budget. Distinguish timeout from caller cancellation.

Retry only transient failures and only when replay is safe or protected by idempotency. Bound attempts and total duration; add jitter. Respect server retry hints when safe. Add circuit breakers only with defined open-state behavior and observable recovery. A fallback must be semantically acceptable, bounded, and visible—never silently return stale or fabricated success.

Stream large payloads, bound response size, dispose responses, validate status and media type, and avoid per-request `HttpClient` construction.

## gRPC and SignalR

For gRPC, set deadlines, propagate cancellation and trace context, map domain failures to stable status/details, and test streaming backpressure. Evolve protobuf additively: never reuse field numbers, reserve removed numbers/names, preserve wire types, and deploy readers before writers.

For SignalR, treat connections as transient. Authenticate the connection and authorize sensitive hub methods. Keep group membership and presence semantics explicit; groups are not durable storage. Bound client messages and invocation concurrency. Use one shared connection per intended client context rather than silently creating competing connections.

## Model Binding and Formatters

Prefer ordinary DTO binding first. Introduce custom binders/formatters only for stable cross-cutting representations. Keep parsing culture, case, nullability, defaulting, size limits, validation, error paths, content negotiation, and security explicit. Never deserialize arbitrary runtime types from untrusted input.

Separate transport validation from domain invariants. Return consistent problem details without internal exception or sensitive-data leakage.

## Operational Checks

- Test middleware ordering and short-circuit paths with the real test host.
- Verify authentication, authorization, rate limiting, body limits, and cancellation.
- Test worker startup failure, poison work, cancellation, and graceful shutdown.
- Verify outbound deadline, retry count, idempotency, breaker, and fallback semantics with fault injection.
- Test protobuf compatibility and SignalR reconnect behavior.
- Distinguish liveness from readiness; do not make liveness depend on every remote service.
