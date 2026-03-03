# TypeScript Development Guardrails

These rules apply to all TypeScript projects regardless of framework. They supplement the general guardrails in the root `CLAUDE.md`.

---

## Strict Mode

- Always enable `strict: true` in `tsconfig.json`. This includes `strictNullChecks`, `noImplicitAny`, `strictFunctionTypes`, and more.
- Never disable strict checks to "fix" type errors. Fix the types instead.

---

## Type Safety Fundamentals

- **Never use `any`.** Use `unknown` when the type is genuinely uncertain, then narrow with type guards.
- **Minimize type assertions (`as`).** If you need more than 3 `as` casts in a file, the types are wrong — fix the source.
- **Non-null assertions (`!`)** must be justified. Every `!` should have an obvious reason or a comment explaining why it's safe.
- **Explicit types on public APIs.** All exported functions must have explicit parameter and return types. Internal/private functions can rely on inference.
- Use `import type` for type-only imports to ensure zero runtime cost.

---

## Utility Types

Use TypeScript's built-in utility types instead of writing manual types:

- `Partial<T>` — Make all properties optional (e.g., update/patch payloads).
- `Required<T>` — Make all properties required.
- `Pick<T, K>` — Select specific properties from a type.
- `Omit<T, K>` — Remove specific properties from a type.
- `Record<K, V>` — Define a dictionary/map type.
- `Readonly<T>` — Make all properties readonly.
- `ReturnType<T>` — Extract the return type of a function.
- `Parameters<T>` — Extract the parameter types of a function.

Combine them: `Partial<Pick<User, 'name' | 'email'>>` is better than defining a separate `UserUpdate` type that duplicates fields.

---

## Type Guards

Use type guards to narrow `unknown` or union types safely:

```typescript
// User-defined type guard
function isUser(value: unknown): value is User {
  return (
    typeof value === 'object' &&
    value !== null &&
    'id' in value &&
    'name' in value
  );
}

// Discriminated union
type Result<T> =
  | { success: true; data: T }
  | { success: false; error: string };

function handleResult(result: Result<User>) {
  if (result.success) {
    // TypeScript knows result.data is User here
    console.log(result.data.name);
  } else {
    // TypeScript knows result.error is string here
    console.error(result.error);
  }
}
```

---

## Discriminated Unions

Use discriminated unions for state management and variant types. Always include a literal discriminant field:

```typescript
type LoadingState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: string };
```

Handle all variants exhaustively with `switch` or `if` chains. Use the `never` type to catch unhandled cases at compile time:

```typescript
function assertNever(value: never): never {
  throw new Error(`Unhandled value: ${value}`);
}
```

---

## The `satisfies` Operator

Use `satisfies` to validate that a value conforms to a type while preserving the narrower inferred type:

```typescript
const config = {
  api: 'https://api.example.com',
  timeout: 5000,
  retries: 3,
} satisfies Record<string, string | number>;
// TypeScript still knows config.api is string, config.timeout is number
```

Prefer `satisfies` over `as` when you want type checking without widening.

---

## Branded Types

Use branded types for type-safe IDs and domain-specific primitives that should not be interchangeable:

```typescript
type UserId = string & { readonly __brand: 'UserId' };
type OrderId = string & { readonly __brand: 'OrderId' };

function createUserId(id: string): UserId {
  return id as UserId;
}

function getUser(id: UserId): User { /* ... */ }

// getUser(orderId) → compile error, even though both are strings
```

---

## Advanced Patterns

### Template Literal Types

```typescript
type EventName = `on${Capitalize<string>}`;
type HttpMethod = 'GET' | 'POST' | 'PUT' | 'DELETE';
type Endpoint = `/${string}`;
```

### Mapped Types

```typescript
type Optional<T> = { [K in keyof T]?: T[K] };
type Immutable<T> = { readonly [K in keyof T]: T[K] };
```

### Conditional Types

```typescript
type ArrayElement<T> = T extends (infer U)[] ? U : never;
type Awaited<T> = T extends Promise<infer U> ? U : T;
```

Use these sparingly — only when they genuinely reduce duplication or prevent errors. Complex type gymnastics hurt readability.

---

## Error Handling

Define custom error classes with context for domain-specific errors:

```typescript
class AppError extends Error {
  constructor(
    message: string,
    public readonly code: string,
    public readonly context?: Record<string, unknown>
  ) {
    super(message);
    this.name = 'AppError';
  }
}

class NotFoundError extends AppError {
  constructor(entity: string, id: string) {
    super(`${entity} not found: ${id}`, 'NOT_FOUND', { entity, id });
    this.name = 'NotFoundError';
  }
}
```

---

## Performance

- Use `import type` to avoid importing runtime code when only types are needed.
- Avoid circular dependencies — they cause subtle bugs and break tree-shaking.
- Prefer `const enum` for internal numeric enums (zero runtime cost). Use regular `enum` only when the values need to be iterable at runtime.
- Use `as const` for constant objects and arrays to get the narrowest possible type.

---

## Common Anti-Patterns to Avoid

| Anti-Pattern | Do This Instead |
|---|---|
| `any` everywhere | `unknown` + type guards |
| `obj as SomeType` without validation | Type guard function |
| `value!` without justification | Null check or optional chaining |
| `enum` for string constants | Union type: `type Status = 'active' \| 'inactive'` |
| Manual type that mirrors another | Utility type: `Pick<T, K>`, `Omit<T, K>` |
| `interface {}` or `type Any = any` | Proper domain types |
| `Function` type | Explicit signature: `(arg: string) => void` |
| `Object` type | `Record<string, unknown>` or specific type |
