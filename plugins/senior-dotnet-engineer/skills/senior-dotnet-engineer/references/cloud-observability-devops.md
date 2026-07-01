# Cloud-Native Delivery, Observability, and DevOps

## Contents

1. Telemetry design
2. Configuration and secrets
3. Aspire
4. Containers
5. Kubernetes
6. Deployment safety

## Telemetry Design

Instrument user-visible and operational boundaries, not every method. Correlate traces, metrics, and structured logs with stable operation names and trace identifiers.

- Logs: use message templates and stable event identifiers. Avoid string interpolation that destroys fields.
- Traces: create spans around meaningful inbound, outbound, queue, and background operations. Propagate context through supported protocols.
- Metrics: measure rates, errors, duration, saturation, queue lag/age, retries, and business outcomes. Keep label cardinality bounded; never label with user IDs, URLs containing IDs, exception messages, or arbitrary payload values.
- Sampling: preserve errors and rare critical flows where possible; understand head versus tail sampling trade-offs.

Use `Microsoft.Extensions.Logging` abstractions and adopt Serilog when its sinks, enrichment, and structured pipeline meet the operational need. Keep event properties stable across providers. Export OpenTelemetry through supported OTLP/Azure Monitor paths to systems such as Jaeger or Application Insights; keep application instrumentation vendor-neutral where practical and verify exporter backpressure and failure behavior.

Audit all telemetry for secrets, tokens, PII, tenant data, prompt content, and large payloads. Observability must remain useful during partial failure and must not become the outage through blocking exporters or unbounded buffering.

## Configuration and Secrets

Bind cohesive settings through typed options and validate required values, ranges, URI schemes, and cross-field invariants at startup. Separate safe defaults from environment-owned secrets.

Use the platform's secret store or workload identity. Never commit secrets to source or ordinary `appsettings.json`; never bake them into image layers or print them in diagnostics. Plan rotation, least privilege, audit, expiration, and behavior when the secret provider is unavailable.

Feature flags need owners, expiry/removal plans, safe defaults, observability, and test coverage for both states.

## .NET Aspire

Use Aspire to compose local/distributed application resources, service discovery, health, telemetry, and development orchestration when it reduces environment drift. Keep production architecture independent of an opaque local-only assumption. Model dependencies and references explicitly; do not treat the AppHost as the owner of business logic or production secrets.

Verify generated configuration and the deployment target's supported integration versions.

## Containers

- Use multi-stage builds and copy only published output into the runtime image.
- Pin intentional base-image versions and rebuild for security patches.
- Run as a non-root user with a read-only filesystem where feasible.
- Set working directory, ports, globalization/time-zone needs, writable paths, and certificate trust explicitly.
- Use trimming, single-file, ReadyToRun, or AOT only after dependency compatibility and reflection/source-generation behavior are tested.
- Handle termination signals and stop accepting work before the grace period expires.

Keep build credentials out of layers and provenance. Scan dependencies and images according to the delivery policy.

## Kubernetes

Define resource requests from measurements and limits with runtime behavior in mind. Separate startup, readiness, and liveness probes. Liveness should detect a stuck process, not every remote dependency; readiness should remove an instance that cannot safely serve.

Account for rolling-update surge/unavailability, disruption budgets, topology, autoscaling signals, pre-stop behavior, termination grace, connection draining, background work, and leader/singleton semantics. Avoid using local container storage for durable state.

## Deployment Safety

Design old and new versions to coexist. Stage database and contract evolution. Validate configuration before traffic. Use canary or progressive delivery when risk justifies it.

Define success and rollback signals before deployment: error rate, latency, saturation, queue lag, compatibility failures, business outcomes, and resource regression. A rollback must account for irreversible data writes and schema changes; binary rollback alone may not be safe.

Verify container build, startup, readiness, traffic handling, telemetry export, graceful shutdown, and rollback in an environment representative of production.
