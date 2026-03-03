# React Development Guardrails

These rules apply to React projects. They supplement the general guardrails in the root `CLAUDE.md`.

---

## MCP Server Configuration

Use the Context7 MCP server for up-to-date React, Next.js, and related library documentation. **Always consult it before generating React code.**

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

---

## Component Design

- **Functional components only.** Never create class components.
- Keep components small and focused on a single responsibility.
- Follow the **container/presentational pattern**:
  - Container components handle data fetching and state.
  - Presentational components receive data via props and render UI.
- Extract reusable logic into **custom hooks** (`use` prefix).
- Prefer composition over prop drilling — use component children and render props where appropriate.

---

## Hooks

- Follow the Rules of Hooks: only call hooks at the top level, only in function components or custom hooks.
- **`useEffect` dependency arrays must be correct and complete.** Never suppress the exhaustive-deps lint rule without a documented reason.
- Use `useEffect` only for synchronization with external systems (API calls, subscriptions, DOM manipulation). Do not use it for derived state — use `useMemo` or compute inline instead.
- `useMemo` and `useCallback` are for performance optimization. Only use them when:
  - The computation is measurably expensive, or
  - The value is passed as a prop/dependency and referential stability matters.
- Clean up side effects: return a cleanup function from `useEffect` when subscribing or adding listeners.

---

## State Management

Apply the **least powerful tool** principle:

1. **Local state** (`useState`) — default choice for component-scoped state.
2. **Derived state** — compute from existing state/props, don't duplicate.
3. **React Context** — for state shared across a subtree (theme, auth, locale). Not for frequently changing data.
4. **External library** (Zustand, Redux Toolkit, Jotai) — only when Context is insufficient (high-frequency updates, complex logic, middleware needs).

- Lift state up only as far as needed.
- Never put server cache in client state — use React Query / TanStack Query or SWR for server state.

---

## React Server Components (Next.js App Router)

- Understand the boundary between Server Components and Client Components.
- Default to Server Components. Only add `'use client'` when the component needs:
  - Event handlers (`onClick`, `onChange`, etc.)
  - Hooks (`useState`, `useEffect`, etc.)
  - Browser-only APIs
- Never import server-only code into client components.
- Use `'use server'` for Server Actions (form handling, mutations).

---

## Rendering & Performance

- Always use a stable, unique `key` prop for list items. Never use array index as key unless the list is static and never reordered.
- Avoid unnecessary re-renders:
  - Don't create objects or functions inline in JSX when they're passed as props.
  - Use `React.memo` only when profiling shows a measurable improvement.
- Use `React.lazy` and `Suspense` for code splitting of heavy components.
- Use the React Profiler to verify performance assumptions — don't optimize blindly.

---

## Error Handling

- Implement **Error Boundaries** at appropriate levels (layout, feature, page).
- Use `react-error-boundary` or custom Error Boundary classes to catch rendering errors gracefully.
- Never let a single component error crash the entire app.

---

## File Structure & Naming

- Use PascalCase for component files: `UserProfile.tsx`, `UserProfile.test.tsx`.
- Group by feature, not by type:
  ```
  features/
    users/
      UserList.tsx
      UserDetail.tsx
      useUsers.ts
      users.api.ts
  ```
- Co-locate tests, styles, and utilities with their components.
- One component per file (small helper components in the same file are acceptable if they're only used there).

---

## TypeScript

- Always use TypeScript in React projects.
- Define prop types with `interface` (not `type` alias, unless union/intersection is needed).
- Avoid `any`. Use `unknown` and narrow types when the type is genuinely unknown.
- Use discriminated unions for complex component variants or state machines.
- Export prop types when components are consumed by others.

---

## Testing

- Use React Testing Library (`@testing-library/react`) — test user behavior, not implementation.
- Query by role, label, or text — avoid `data-testid` unless no semantic alternative exists.
- Never test internal state or hook implementation directly. Test what the user sees and does.
- For hooks, use `renderHook` from `@testing-library/react`.
- Mock API calls with MSW (Mock Service Worker) at the network level, not by mocking fetch/axios.
