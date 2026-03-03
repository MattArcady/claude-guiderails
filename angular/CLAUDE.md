# Angular Development Guardrails

These rules apply to Angular projects. They supplement the general guardrails in the root `CLAUDE.md` and the TypeScript guardrails in `typescript/CLAUDE.md` (which also apply to all Angular projects).

---

## MCP Server Configuration

Use the official Angular CLI MCP server for up-to-date Angular documentation and best practices. **Always consult it before generating Angular code.**

```json
{
  "mcpServers": {
    "angular-cli": {
      "command": "npx",
      "args": ["-y", "@angular/cli", "mcp"]
    }
  }
}
```

Reference: https://angular.dev/ai/mcp

---

## Component Architecture

- **Standalone components are the default.** Do not use NgModules for new components unless there is a specific legacy reason.
- Follow the **smart/dumb component pattern** (container/presentational):
  - Smart components handle data fetching, state, and orchestration.
  - Dumb components receive data via `input()` and emit events via `output()`. They contain no business logic.
- **Do not use decorators for inputs and outputs.** Use the signal-based `input()`, `input.required()`, `output()`, and `model()` functions instead of `@Input()`, `@Output()`, and `@ViewChild()`. The decorator syntax is legacy.
- Use `changeDetection: ChangeDetectionStrategy.OnPush` as the default for all components. With signals this is the natural fit and improves performance.
- Keep templates lean. Move complex logic to the component class or dedicated pipes.

---

## Signals & State Management

- Prefer **Angular Signals** over BehaviorSubject/ReplaySubject for component-level state (Angular 17+).
- Use `signal()`, `computed()`, and `effect()` for reactive state within components.
- Reserve RxJS Observables for streams that truly need operators (HTTP requests, WebSocket streams, complex event composition).
- For application-wide state, evaluate whether a simple service with signals suffices before reaching for NgRx or similar.

---

## Dependency Injection

- Use the `inject()` function over constructor injection. It is more concise and works in functional contexts.
- Provide services at the appropriate level: `providedIn: 'root'` for singletons, component-level for scoped services.
- Never use `any` as a DI token type.

---

## Forms

- Use **Reactive Forms** for forms with complex validation, dynamic fields, or cross-field validation.
- Template-driven forms are acceptable for simple, static forms (e.g., login, contact).
- Always use strongly typed forms (`FormGroup<T>`, `FormControl<T>`).

---

## Routing

- Lazy load all feature routes using `loadComponent` or `loadChildren`.
- Use route guards (`CanActivate`, `CanDeactivate`) as functional guards.
- Keep route definitions in dedicated route files (`*.routes.ts`).

---

## RxJS Best Practices

- Use `takeUntilDestroyed()` (from `@angular/core/rxjs-interop`) to manage subscription lifetimes.
- Prefer the `async` pipe in templates over manual `.subscribe()` calls.
- Avoid nested subscriptions — use `switchMap`, `mergeMap`, `concatMap` instead.
- Always handle errors in observable chains with `catchError`.
- Unsubscribe from all subscriptions. Leaked subscriptions cause memory leaks.

---

## File Structure & Naming

- Follow the Angular style guide naming conventions:
  - `feature-name.component.ts`, `feature-name.service.ts`, `feature-name.pipe.ts`
  - One class per file.
- Group by feature, not by type. Example:
  ```
  users/
    user-list.component.ts
    user-detail.component.ts
    user.service.ts
    user.routes.ts
  ```
- Barrel exports (`index.ts`) are optional — use them only when they add clarity, not by default.

---

## Performance

- Use `@defer` blocks for lazy loading heavy template sections.
- Use `trackBy` functions (or `track` in `@for`) to optimize list rendering.
- Avoid function calls in templates — use pipes or computed signals instead.
- Use Angular's built-in image optimization (`NgOptimizedImage`) for images.

---

## Testing

- Prefer **Vitest** as test runner when the project setup allows it. Fall back to **Jest** if Vitest is not feasible. Only use Jasmine/Karma for existing projects that haven't migrated yet.
- Use the Angular testing utilities (`TestBed`, `ComponentFixture`) for component tests.
- Mock services using test doubles provided via DI (or `vi.fn()` / `jest.fn()` depending on the runner).
- Test component behavior through the DOM, not internal implementation.
- For services with HTTP calls, use `provideHttpClientTesting()` and `HttpTestingController`.
