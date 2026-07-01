# Senior .NET Engineer for Codex

A production-oriented Codex plugin for advanced C# and modern .NET engineering. It guides Codex through repository-first analysis, proportionate architecture, implementation, and evidence-based verification.

## Capabilities

- Advanced C#, async execution, concurrency, low-allocation code, reflection, expression trees, and source generators
- ASP.NET Core middleware, hosting, dependency injection, gRPC, SignalR, binding, and resilience
- DDD, modular monoliths, microservices, CQRS, event sourcing, reliable messaging, and contract evolution
- EF Core optimization, concurrency control, SQL plans, indexing, and Dapper
- OpenTelemetry, structured logging, .NET Aspire, containers, Kubernetes, and safe deployments
- Semantic Kernel, RAG, AI security and evaluation, and ML.NET

## Install

Add this public marketplace:

```bash
codex plugin marketplace add codehepta/senior-dotnet-engineer
```

Then open the Codex plugin directory with `/plugins`, select **Codehepta .NET Plugins**, and install **Senior .NET Engineer**.

Start a new thread after installation. Describe the desired outcome directly or invoke the plugin explicitly with `@senior-dotnet-engineer`.

## Example prompts

- `@senior-dotnet-engineer review this ASP.NET Core service for production readiness.`
- `@senior-dotnet-engineer diagnose and benchmark allocations in this C# hot path.`
- `@senior-dotnet-engineer design a backward-compatible event-driven workflow with an outbox.`

## Design principles

The plugin defaults to ordinary managed code, modular boundaries, bounded resources, explicit cancellation and timeouts, backward-compatible contracts, secure configuration, and verification proportional to risk. It recommends specialized performance or distributed-system techniques only when constraints and measurements justify them.

## License

MIT
