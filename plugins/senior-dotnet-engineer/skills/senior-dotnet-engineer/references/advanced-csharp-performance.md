# Advanced C# and High-Performance Code

## Contents

1. Measurement
2. Async execution
3. Concurrency
4. Low-allocation code
5. Runtime metaprogramming
6. Source generation
7. Review checklist

## Measurement First

Define the target metric and representative workload. Capture a baseline in Release mode without an attached debugger. Use BenchmarkDotNet for microbenchmarks and runtime traces/counters for whole-system behavior. Validate that an apparent CPU or allocation hot path matters to end-to-end latency, throughput, or cost.

Do not trade clarity for allocation tricks on cold paths. Record environment, runtime, payload distribution, concurrency, warmup, and statistical spread.

## Async Execution

- Use `Task` as the default asynchronous contract.
- Use `ValueTask` only when synchronous completion is common, measurements show allocation impact, and callers obey its consumption constraints. Avoid multiple awaits or storing it unless converted to `Task`.
- Propagate `CancellationToken`; distinguish caller cancellation from dependency timeout and unexpected failure.
- Never use `.Result`, `.Wait()`, or `GetAwaiter().GetResult()` in request or worker paths.
- In reusable libraries, use `ConfigureAwait(false)` when no caller context is required. In ASP.NET Core application code, do not sprinkle it without a demonstrated semantic need.
- Detect starvation through thread-pool queueing, blocked threads, long synchronous sections, and uncontrolled fan-out. Async does not make CPU work free.
- Observe fire-and-forget work through a durable queue or an owned background service; never discard the returned task.

## Concurrency and Threading

Choose the ownership model before the primitive:

- Immutable snapshots for read-heavy state.
- `lock`/`Monitor` for short synchronous critical sections with no awaits.
- `SemaphoreSlim` for bounded asynchronous admission, always released in `finally`.
- `Interlocked`/`Volatile` only when the memory-ordering argument is explicit and tested.
- `ConcurrentDictionary` for atomic per-key operations; ensure factories are safe to invoke more than once.
- Bounded `Channel<T>` for producer/consumer pipelines; define full mode, capacity, completion, cancellation, and poison-item handling.
- TPL parallelism only for independent work with a measured degree of parallelism.

Test races with barriers, task-completion sources, or controlled schedulers. Do not use arbitrary sleeps as correctness proof. Document lock ordering and never await while holding a monitor lock.

## Low-Allocation Code

- Prefer projections, streaming, reuse, and fewer copies before specialized primitives.
- Use `Span<T>` only within synchronous lifetime-safe scopes. Never attempt to store it on the heap or cross an async/yield boundary.
- Use `Memory<T>` for heap-storable or asynchronous buffers; specify who owns the underlying memory and for how long.
- Keep `stackalloc` bounded by a small, validated maximum. Never size it directly from untrusted input.
- Use `ref struct` when stack-only lifetime enforcement is part of the API contract, accepting its composition limits.
- Rent from `ArrayPool<T>` only with one explicit owner. Return exactly once from `finally`, never expose the rented array beyond the lease, and clear the used region before return when it held secrets or PII.
- Avoid hidden allocations from closures, boxing, interface enumeration, params arrays, exceptions on normal paths, and repeated encoding/conversion only after a profiler identifies them.

Benchmark the simpler alternative. Pooling can increase retained memory, contention, data leakage risk, and operational complexity.

## Expression Trees, Reflection, and Emitted Code

Use reflection for cold startup/configuration paths when it remains simplest. Cache validated metadata where repeated lookup is measured. Prefer compiled delegates over repeated `MethodInfo.Invoke` on hot paths.

Use expression trees for composable query representation or generated accessors, not merely to appear dynamic. When building LINQ providers, separate expression normalization, translation, parameterization, execution, and result materialization. Reject unsupported nodes explicitly; never silently switch large queries to client evaluation.

Use dynamic IL or runtime code emission only when profiling proves the benefit and AOT/trimming constraints are understood. Validate untrusted input before it influences generated code.

## Source Generators

Choose a source generator when compile-time knowledge can replace material reflection at scale and generated output can remain deterministic, diagnosable, incremental, and testable. Emit useful diagnostics for invalid user code. Avoid file-system, clock, network, or unordered-input dependence. Test generated sources and consuming compilation.

Prefer established generators for serialization and logging. Account for incremental-build cost, IDE behavior, binary compatibility, trimming, and generated-code debuggability.

## Review Checklist

- Is there baseline and after evidence for the claimed bottleneck?
- Are concurrency and buffer ownership explicit?
- Are all queues, buffers, batches, and degrees of parallelism bounded?
- Does cancellation reach blocking or remote operations?
- Are pooled buffers returned once and sensitive regions cleared?
- Does the design work under trimming/AOT if the application requires it?
- Are benchmark results representative rather than microbenchmark theater?
