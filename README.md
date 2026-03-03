# Claude Guiderails

A reusable collection of `CLAUDE.md` guardrails for software development teams working with [Claude Code](https://docs.anthropic.com/en/docs/claude-code). Copy what you need into your project and get consistent, high-quality AI-assisted development out of the box.

## What's included

| Path | Description |
|---|---|
| `CLAUDE.md` | General software development guardrails — SOLID, DRY, design patterns, security, git discipline, testing mindset |
| `typescript/CLAUDE.md` | TypeScript-specific rules — strict mode, type safety, generics, error handling, module design |
| `angular/CLAUDE.md` | Angular-specific rules — standalone components, signals, OnPush, RxJS best practices |
| `react/CLAUDE.md` | React-specific rules — functional components, hooks, state management, Server Components |
| `dotnet/CLAUDE.md` | .NET/C#-specific rules — Clean Architecture, async/await, EF Core, middleware, DI |
| `commands/code-review.md` | Multi-agent code review command (Opus + Sonnet + Haiku with cross-verification) |

## How to use

### 1. Copy the guardrails you need

Pick the files relevant to your stack and copy them into your project, preserving the folder structure:

```bash
# Example: React + TypeScript project
cp claude-guiderails/CLAUDE.md ./CLAUDE.md
mkdir -p typescript react
cp claude-guiderails/typescript/CLAUDE.md ./typescript/CLAUDE.md
cp claude-guiderails/react/CLAUDE.md ./react/CLAUDE.md
```

The root `CLAUDE.md` contains the general rules that apply to all projects. The framework-specific files supplement (not replace) the root file.

### 2. Add MCP server configuration

Each framework guardrail file includes a recommended MCP server config for up-to-date documentation. Add the relevant config to your project's `.mcp.json`:

- **Angular** — [Angular CLI MCP](https://angular.dev/ai/mcp) (official, maintained by the Angular team)
- **React** — [Context7](https://github.com/upstash/context7) (covers React, Next.js, Tailwind, and 9000+ libraries)
- **.NET** — [Context7](https://github.com/upstash/context7) (covers ASP.NET Core, Entity Framework, etc.)

### 3. Use the code review command

Copy `commands/code-review.md` to your project's `.claude/commands/` folder to enable the `/code-review` slash command in Claude Code:

```bash
mkdir -p .claude/commands
cp claude-guiderails/commands/code-review.md .claude/commands/code-review.md
```

## Structure

```
claude-guiderails/
├── CLAUDE.md                 # General software development guardrails
├── typescript/
│   └── CLAUDE.md             # TypeScript-specific rules
├── angular/
│   └── CLAUDE.md             # Angular-specific rules + MCP config
├── react/
│   └── CLAUDE.md             # React-specific rules + MCP config
├── dotnet/
│   └── CLAUDE.md             # .NET-specific rules + MCP config
└── commands/
    └── code-review.md        # Multi-agent code review command
```

## Key principles

The guardrails are opinionated but pragmatic. Some highlights:

- **Plan before you code** — Claude must present a plan and wait for approval before making non-trivial changes.
- **Never commit autonomously** — All git operations require explicit developer approval.
- **Safety nets stay** — Input validation, auth checks, error handlers, and type guards are never removed without approval.
- **No `any`** — Strict type safety is enforced across TypeScript, Angular, and React guardrails.
- **SOLID + DRY with restraint** — Apply the Rule of Three before abstracting; don't over-engineer.

## Customization

These guardrails are a starting point. Feel free to:

- Remove rules that don't match your team's conventions
- Add project-specific rules (naming conventions, folder structure, etc.)
- Adjust strictness levels to fit your workflow
- Add more framework-specific files for other stacks

## License

MIT
