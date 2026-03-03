# .NET Development Guardrails

These rules apply to .NET / C# projects. They supplement the general guardrails in the root `CLAUDE.md`.

---

## MCP Server Configuration

Use the Context7 MCP server for up-to-date .NET, ASP.NET Core, and Entity Framework documentation.

```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"]
    }
  }
}
```

Reference: https://github.com/upstash/context7

Optionally, add the dotnet-tools MCP server for build and test integration:

```json
{
  "mcpServers": {
    "dotnet-tools": {
      "command": "npx",
      "args": ["-y", "dotnet-tools-mcp-server"]
    }
  }
}
```

Reference: https://github.com/AdamTovatt/dotnet-tools-mcp-server

---

## Architecture

- Follow **Clean Architecture** (or a layered variant) with clear separation:
  - **Domain** — Entities, value objects, domain events, interfaces. No external dependencies.
  - **Application** — Use cases, DTOs, application services, validation. Depends only on Domain.
  - **Infrastructure** — Database, external APIs, file system, messaging. Implements Domain interfaces.
  - **Presentation** — Controllers/Endpoints, view models, API contracts. As thin as possible.
- Dependencies flow inward: Presentation → Application → Domain ← Infrastructure.
- The composition root (typically `Program.cs`) is the only place that wires everything together.

---

## Dependency Injection

- Use the built-in .NET DI container. Avoid third-party containers unless there's a specific need.
- Register services with the appropriate lifetime:
  - `Singleton` — Stateless services, configuration.
  - `Scoped` — Per-request services (DbContext, repositories, unit of work).
  - `Transient` — Lightweight, stateless services with no shared state.
- Never resolve services from the container manually (`IServiceProvider.GetService`). Use constructor injection.
- Avoid the **Captive Dependency** anti-pattern: don't inject Scoped services into Singletons.

---

## Async/Await

- Use `async`/`await` throughout the call chain. Never block on async code.
- **Never** use `.Result`, `.Wait()`, or `.GetAwaiter().GetResult()` — these cause deadlocks.
- Use `ConfigureAwait(false)` in library code (but not in ASP.NET controller/middleware code).
- Suffix async methods with `Async` (e.g., `GetUserAsync`).
- Return `Task` or `ValueTask`, never `async void` (except event handlers).
- Use `CancellationToken` parameters for cancellable operations and pass them through the call chain.

---

## Entity Framework Core

- Use **code-first migrations**. Keep migrations in source control.
- Use `AsNoTracking()` for read-only queries to improve performance.
- Avoid N+1 queries — use `Include()` / `ThenInclude()` or projection with `Select()`.
- Use `IQueryable<T>` in repositories; materialize queries as late as possible.
- Don't expose `DbContext` outside the Infrastructure layer. Wrap it behind repository interfaces.
- Keep queries in the repository or dedicated query handlers — never in controllers.
- Be cautious with lazy loading. Prefer explicit loading or eager loading.

---

## API Design

### Minimal APIs vs Controllers

- **Minimal APIs** — For simple endpoints, microservices, or when you want lightweight setup.
- **Controllers** — For complex APIs with many actions, when you need filters, model binding, or conventional routing.
- Be consistent within a project. Don't mix both approaches in the same bounded context.

### General API Rules

- Use proper HTTP methods and status codes.
- Return `IActionResult` or `Results` (Minimal APIs) with explicit status codes.
- Use `ActionResult<T>` for typed responses in controllers.
- Apply `[ApiController]` attribute to enable automatic model validation and binding source inference.
- Version your APIs from day one.

---

## Configuration

- Use the **Options pattern** (`IOptions<T>`, `IOptionsSnapshot<T>`, `IOptionsMonitor<T>`) for strongly typed configuration.
- Never read from `IConfiguration` directly in services. Bind configuration to POCO classes.
- Keep secrets out of `appsettings.json`. Use User Secrets (development), environment variables, or a vault (production).
- Validate configuration at startup with `ValidateDataAnnotations()` or `ValidateOnStart()`.

---

## Middleware & Pipeline

- Order middleware correctly. The order in `Program.cs` defines the execution pipeline:
  1. Exception handling
  2. HTTPS redirection
  3. Static files
  4. Routing
  5. CORS
  6. Authentication
  7. Authorization
  8. Custom middleware
  9. Endpoints
- Keep middleware focused on cross-cutting concerns (logging, auth, error handling).
- Use `IExceptionHandler` (ASP.NET Core 8+) or custom middleware for global error handling. Do not rely on try/catch in every controller.

---

## Error Handling

- Use global exception handling middleware for unexpected errors.
- For expected failures, use the **Result pattern** (return a result object with success/failure instead of throwing exceptions).
- Never return stack traces or internal details in API responses. Log the details, return a safe error message.
- Use `ProblemDetails` (RFC 9457) for standardized API error responses.

---

## Logging

- Use `ILogger<T>` from `Microsoft.Extensions.Logging`.
- Use **structured logging** with message templates: `_logger.LogInformation("User {UserId} logged in", userId)` — not string interpolation.
- Log at appropriate levels: `Trace` for debugging detail, `Debug` for dev, `Information` for flow, `Warning` for recoverable issues, `Error` for failures, `Critical` for system-breaking issues.
- Never log sensitive data (passwords, tokens, PII).

---

## C# Language Features

- Enable **nullable reference types** (`<Nullable>enable</Nullable>`). Fix all nullable warnings.
- Use `record` types for DTOs and value objects.
- Use pattern matching for type checks and switch expressions.
- Use `init` properties for immutable configuration.
- Prefer `string.IsNullOrEmpty` / `string.IsNullOrWhiteSpace` over null checks + length checks.
- Use collection expressions (`[1, 2, 3]`) and ranges (`array[1..3]`) where they improve readability (C# 12+).

---

## Testing

- Use **xUnit** as the test framework (or NUnit — be consistent within the project).
- Use **NSubstitute** or **Moq** for mocking interfaces.
- Structure tests with **Arrange-Act-Assert**.
- Name tests descriptively: `MethodName_Scenario_ExpectedResult` or a BDD-style sentence.
- Use `WebApplicationFactory<T>` for integration tests of ASP.NET Core APIs.
- Test the behavior through the public API, not internal implementation.
- Use **Testcontainers** for integration tests that need real databases.
