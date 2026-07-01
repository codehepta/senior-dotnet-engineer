# Data Access and Persistence

## Contents

1. Query shape
2. EF Core execution
3. Transactions and concurrency
4. SQL and indexes
5. Dapper
6. Schema evolution

## Query Shape First

Start from the required result shape. Project with `Select` rather than materializing full entities for reads. Use `AsNoTracking` when identity resolution and updates are not required; use the appropriate identity-resolution option only when measurements and semantics justify it.

Detect N+1 behavior through query logging/counting and representative integration tests. Choose eager loading, explicit projection, batching, or a dedicated query. Compare split and single queries: split queries can avoid cartesian explosion but add round trips and consistency considerations.

Paginate deterministically with a stable tie-breaker. Prefer keyset pagination for deep, ordered traversal when random page access is unnecessary. Bound result sizes and avoid client-side evaluation.

## EF Core Execution

- Inspect generated SQL and parameterization.
- Use compiled queries only for hot, stable shapes after measuring query-compilation overhead.
- Batch writes through the normal change tracker when domain behavior matters.
- Use `ExecuteUpdateAsync`/`ExecuteDeleteAsync` for set-based operations when bypassing tracked entities, interceptors, and domain events is intentional and documented.
- Avoid per-row save loops. Define batch size, transaction, cancellation, and failure recovery.
- Treat context instances as short-lived units of work and never use one concurrently.

Do not hide database calls behind innocent-looking property access or enumerable abstractions.

## Transactions and Concurrency

Keep transactions short and include only the resources they can actually make atomic. Understand execution-strategy retries before opening manual transactions or performing external side effects.

Use optimistic concurrency by default for interactive and distributed workloads. Add a concurrency token, detect conflicts, and choose reject, retry with recomputation, or domain-specific merge. Never overwrite silently. Bound retries and reload state each attempt.

Use pessimistic locking only when contention and invariant requirements justify blocking/deadlock risk. Define lock order, isolation level, timeout, retry, and monitoring. Never hold a database transaction across remote calls or user interaction.

Use an outbox instead of attempting a false transaction between the database and broker.

## SQL Plans and Indexes

Profile with production-like cardinality and parameter distributions. Read the actual execution plan where possible. Examine estimated versus actual rows, scans/seeks, joins, sorts, spills, lookups, memory grants, implicit conversions, and parameter sensitivity.

Design indexes from predicates, join keys, ordering, selectivity, and included projections. Account for write amplification, storage, maintenance, and overlapping indexes. An index that speeds one query can damage the workload.

Measure database time separately from network and materialization time. Preserve provider-specific semantics in tests when they matter; an in-memory substitute is not a relational database.

## Dapper and Raw SQL

Adopt Dapper for a measured hot path, specialized SQL, or mapping shape that EF Core cannot express efficiently enough. Keep it behind the same application boundary rather than creating a second architecture.

Require parameterized input, explicit selected columns, bounded results, cancellation, transaction ownership, command timeout, null/type mapping, and result-shape integration tests. Never concatenate identifiers or values from untrusted input. When identifiers must vary, select from an allowlisted mapping.

Compare end-to-end performance against an optimized EF projection before accepting the maintenance cost.

## Schema Evolution

Use migration tooling supported by the repository; never improvise generated migration artifacts. Review destructive operations and generated SQL. For rolling deployments, use expand/migrate/contract:

1. Add backward-compatible schema.
2. Deploy code that handles old and new states.
3. Backfill in bounded, restartable, observable batches.
4. Switch behavior with a reversible control.
5. Remove the old shape only after compatibility telemetry proves it unused.

Verify rollback direction, lock duration, table growth, replication impact, and backup/restore implications.
