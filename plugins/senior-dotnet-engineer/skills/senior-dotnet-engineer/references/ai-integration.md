# AI Integration in .NET

## Contents

1. System boundary
2. Semantic Kernel
3. Retrieval-augmented generation
4. Security and privacy
5. Evaluation and operations
6. ML.NET

## Treat AI as an External Probabilistic Dependency

Put model access behind application-owned interfaces. Keep domain policy, authorization, validation, persistence, and deterministic calculations outside prompts. Define timeout, retry, concurrency, token, latency, cost, data-region, and fallback budgets.

Model output is untrusted input. Parse into bounded schemas, validate, authorize resulting actions, and require confirmation for consequential effects. Never claim determinism or correctness from fluent output.

## Semantic Kernel

Use Semantic Kernel when its orchestration, plugins, filters, memory, or provider abstraction materially reduces application complexity. Keep kernels and plugin registration explicit. Expose narrow, intention-revealing functions with typed inputs, validation, authorization, cancellation, and observable outcomes.

Do not expose arbitrary service-container access, filesystem access, SQL, HTTP, or reflection as a generic model tool. Separate planning from execution and cap tool-call depth, count, duration, and payload. Version prompts and orchestration configuration like code.

Avoid binding core business logic to framework-specific context types; wrap provider and orchestration behavior behind owned contracts.

## Retrieval-Augmented Generation

Design ingestion and retrieval as a data system:

- Record source identity, tenant, permissions, version, timestamps, and deletion lineage.
- Choose chunking from document structure and query needs; preserve useful context without uncontrolled token growth.
- Evaluate embedding model, distance metric, filters, hybrid search, reranking, top-k, freshness, and multilingual behavior.
- Enforce authorization before retrieval and again before presenting citations.
- Return verifiable source references and distinguish retrieved evidence from generated synthesis.
- Make re-indexing, tombstones, model migration, and partial failure observable and restartable.

Do not let retrieved text override system policy. Treat it as untrusted content.

## Security and Privacy

Defend against direct and indirect prompt injection, tool abuse, data exfiltration, cross-tenant retrieval, secret disclosure, malicious documents, and oversized inputs. Apply content and file validation before indexing. Minimize data sent to providers and document retention/training settings.

Never place secrets in prompts. Redact sensitive logs and traces. Use allowlisted tools, least-privilege identities, egress controls, bounded arguments, and human approval for irreversible or high-impact actions.

## Evaluation and Operations

Build a task-specific evaluation set with normal, edge, adversarial, multilingual, and refusal cases. Track groundedness, answer/task quality, retrieval recall/precision, citation validity, safety, latency, token usage, and cost. Compare against a baseline and inspect regressions rather than optimizing a single aggregate score.

Monitor provider errors, throttling, timeout, fallback rate, model/version changes, retrieval drift, and cost. Define deterministic degradation: cached safe response, search-only results, queued processing, human review, or explicit unavailability. Retry only transient failures within the caller's budget and do not duplicate side effects.

## ML.NET

Use ML.NET when the problem fits supported trainers/transforms and in-process .NET training or inference offers operational value. Establish a simple baseline first. Split data without leakage, preserve representative class/time distributions, seed reproducible steps where supported, and tune only against held-out validation.

Version training data, feature schema, pipeline, model artifact, runtime, and evaluation results. Validate input schema at inference, bound resources, and load models safely. Monitor feature drift, prediction distribution, latency, failures, and outcome quality. Define retraining, promotion, rollback, and stale-model behavior before production deployment.
