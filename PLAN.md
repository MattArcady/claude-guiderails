# Plan: Claude Guiderails Repository

## Doel

Een herbruikbare verzameling `CLAUDE.md` guardrails voor software development teams die met Claude Code werken. De repo bevat algemene development principes op root-niveau en framework-specifieke regels in subfolders.

---

## Structuur

```
claude-guiderails/
├── CLAUDE.md                          # Algemene software development guardrails
├── PLAN.md                            # Dit bestand
├── angular/
│   └── CLAUDE.md                      # Angular-specifieke regels + MCP config
├── react/
│   └── CLAUDE.md                      # React-specifieke regels + MCP config
├── dotnet/
│   └── CLAUDE.md                      # .NET-specifieke regels + MCP config
└── commands/
    └── code-review.md                 # Code review skill/command
```

---

## 1. Root `CLAUDE.md` — Algemene Software Development Regels

### Inhoud

- **SOLID Principes** — Korte, actionable regels per principe (Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, Dependency Inversion) met concrete do's en don'ts
- **DRY (Don't Repeat Yourself)** — Wanneer abstractie wel/niet gepast is, Rule of Three
- **Design Patterns** — Instructie om als senior developer te denken; gebruik van bekende patterns waar passend (Factory, Strategy, Observer, Repository, etc.) zonder over-engineering
- **Code kwaliteit** — Naamgeving conventies, kleine functies, separation of concerns, meaningful comments (niet wat maar waarom)
- **Git discipline** — **Nooit zelf committen of pushen.** Altijd aan de gebruiker vragen. Geen `--no-verify`, geen `--force`
- **Security awareness** — OWASP top 10 bewustzijn, geen secrets in code, input validatie op systeemgrenzen
- **Testing mindset** — Code schrijven met testbaarheid in gedachten, dependency injection, pure functies waar mogelijk

---

## 2. `angular/CLAUDE.md` — Angular-specifieke Regels

### MCP Server

- **Angular CLI MCP** — Officieel ondersteund door het Angular team
- Config: `{ "mcpServers": { "angular-cli": { "command": "npx", "args": ["-y", "@angular/cli", "mcp"] } } }`
- Bron: https://angular.dev/ai/mcp

### Inhoud

- Gebruik van standalone components (default sinds Angular 17+)
- Signals gebruiken voor state management (niet meer BehaviorSubject waar signals volstaan)
- `inject()` functie boven constructor injection
- Smart vs dumb component pattern (container/presentational)
- Reactive forms boven template-driven forms voor complexe scenario's
- Lazy loading van routes
- OnPush change detection als default
- Angular style guide volgen (file naming, folder structuur)
- RxJS best practices: `takeUntilDestroyed()`, vermijd memory leaks, gebruik `async` pipe
- Gebruik de Angular CLI MCP server om altijd actuele Angular documentatie en best practices te raadplegen

---

## 3. `react/CLAUDE.md` — React-specifieke Regels

### MCP Server

- **Context7** — Up-to-date documentatie voor 9000+ libraries waaronder React, Next.js, Tailwind
- Config: `{ "mcpServers": { "context7": { "command": "npx", "args": ["-y", "@upstash/context7-mcp"] } } }`
- Bron: https://github.com/upstash/context7

### Inhoud

- Functional components only (geen class components)
- Custom hooks voor herbruikbare logica
- Correcte dependency arrays in `useEffect`, `useMemo`, `useCallback`
- State management: lokale state eerst, context voor gedeelde state, externe libraries (Zustand/Redux) alleen als nodig
- Component compositie boven prop drilling
- React Server Components awareness (Next.js App Router)
- `key` prop correct gebruiken in lijsten
- Vermijd onnodige re-renders (memo, useMemo, useCallback alleen waar meetbaar verschil)
- Error boundaries implementeren
- Gebruik Context7 MCP server om altijd actuele React/Next.js documentatie te raadplegen

---

## 4. `dotnet/CLAUDE.md` — .NET-specifieke Regels

### MCP Server

- **Context7** — Ondersteunt ook .NET/C# libraries (ASP.NET Core, Entity Framework, etc.)
- Config: `{ "mcpServers": { "context7": { "command": "npx", "args": ["-y", "@upstash/context7-mcp"] } } }`
- Optioneel aanvullend: **dotnet-tools-mcp-server** voor `dotnet build` en `dotnet test` integratie
  - Bron: https://github.com/AdamTovatt/dotnet-tools-mcp-server

### Inhoud

- Clean Architecture / layered architecture (Domain, Application, Infrastructure, Presentation)
- Dependency Injection via de ingebouwde DI container
- Repository + Unit of Work pattern voor data access
- Async/await correct gebruiken (geen `.Result` of `.Wait()`, `ConfigureAwait` waar nodig)
- Entity Framework best practices: geen tracking voor read-only queries, migrations beheren, N+1 queries vermijden
- Minimal APIs vs Controllers: wanneer welke te gebruiken
- Options pattern voor configuratie (`IOptions<T>`)
- Middleware pipeline correct ordenen
- Global error handling via middleware
- Logging met `ILogger<T>` en structured logging
- Nullable reference types enabled houden
- Gebruik Context7 MCP server om altijd actuele .NET documentatie te raadplegen

---

## 5. `commands/code-review.md` — Code Review Command

### Doel

Een herbruikbare Claude Code skill/command die een gestructureerde code review uitvoert.

### Inhoud van het command

Het command zal Claude instrueren om de volgende stappen te doorlopen:

1. **Wijzigingen ophalen** — `git diff` analyseren (staged + unstaged)
2. **Per bestand reviewen** op:
   - SOLID violations
   - DRY violations / duplicatie
   - Security issues (OWASP top 10)
   - Performance problemen
   - Naming en readability
   - Ontbrekende of overbodige error handling
   - Test coverage gaps
3. **Output format** — Gestructureerd rapport met severity levels (critical / warning / suggestion)
4. **Samenvatting** — Overall assessment en top 3 aanbevelingen

### Formaat

Het command wordt geschreven als een Claude Code custom slash command (`.md` bestand in de `commands/` folder) dat gebruikers kunnen aanroepen met `/code-review`.

---

## Aanpak / Volgorde van Uitvoering

1. Root `CLAUDE.md` schrijven (fundament voor alles)
2. `angular/CLAUDE.md` schrijven
3. `react/CLAUDE.md` schrijven
4. `dotnet/CLAUDE.md` schrijven
5. `commands/code-review.md` schrijven
6. Review van het geheel op consistentie en volledigheid

---

## Ontwerpbeslissingen

| Beslissing | Keuze | Reden |
|---|---|---|
| MCP server voor .NET | Context7 (+ optioneel dotnet-tools-mcp) | Context7 dekt .NET docs af net als React; geen dedicated .NET docs MCP beschikbaar |
| Taal van de guardrails | Engels | Code en tooling zijn Engels; bereik is groter |
| Eén command als startpunt | Code review | Hoogste impact, breed toepasbaar ongeacht framework |
| CLAUDE.md per framework folder | Ja | Gebruikers kopiëren alleen wat ze nodig hebben naar hun project |
