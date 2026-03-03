# Code Review Command

**Purpose:** Multi-agent code review with cross-verification. Three models (Opus, Sonnet, Haiku) independently review the same code, then verify each other's findings to eliminate false positives/negatives.

**Prerequisites:** Changes exist on the current branch vs the base branch.

---

## Expected Output

```
Multi-Agent Code Review: [Branch Name]
Branch: feature/xxx | Status: Ready / Needs Work / Blocking | Commits: X | Files: Y
Review Method: Opus + Sonnet + Haiku with Cross-Verification
Confidence: N HIGH · N MEDIUM · N LOW · N DISPUTED
Strengths · Suggestions · Issues · Coverage · Security · Performance
```

---

## Phase 1: Context Gathering (Steps 1-4)

**Outputs:** {branch_name}, {base_branch}, {commit_count}, {commit_list}

1. Run `git rev-parse --abbrev-ref HEAD` → SAVE as {branch_name}

2. **CHECKPOINT** — Determine base branch
   - Run `git log --oneline --all --decorate | head -20` and inspect branch topology
   - Common bases: `main`, `master`, `develop` — SAVE as {base_branch}
   - IF unclear: ASK "What is the base branch for this review?" → SAVE response as {base_branch}

3. Run `git log {base_branch}..HEAD --oneline` → SAVE as {commit_list}, COUNT lines → {commit_count}

4. **CHECKPOINT** — IF {commit_count} === 0: REPORT "No commits found on branch vs {base_branch}" → STOP

---

## Phase 2: Diff Analysis (Steps 5-8)

**Outputs:** {diff_output}, {diff_stats}, {file_categories}

5. Run `git diff {base_branch}...HEAD --stat` → {diff_stats}
6. Run `git diff {base_branch}...HEAD` → {diff_output}

7. **CHECKPOINT** — IF diff empty: STOP · IF >100K chars: TRUNCATE to source files only, WARN "Diff truncated for agent processing"

8. CATEGORIZE files from {diff_stats} by extension/path:
   - {source_files}: All source code files (`.ts`, `.js`, `.cs`, `.py`, `.java`, etc.) excluding test files
   - {test_files}: All test files (`*.spec.ts`, `*.test.ts`, `*.test.js`, `*Tests.cs`, `*_test.py`, etc.)
   - {templates}: Template files (`.html`, `.vue`, `.jsx`, `.tsx`, `.razor`, etc.)
   - {styles}: Style files (`.css`, `.scss`, `.less`, etc.)
   - {config}: Configuration files (`.json`, `.yaml`, `.yml`, `.toml`, `.xml`, etc.)
   - {other}: Anything else
   - OUTPUT file counts per category

---

## Phase 3: Review Context Assembly (Step 9)

**Outputs:** {review_context_payload}, {active_review_domains}

9. BUILD {review_context_payload} and determine {active_review_domains}:
   - Architecture & SOLID: ALWAYS active
   - Type Safety: ALWAYS active
   - Security: ALWAYS active
   - Performance: ALWAYS active
   - Tests: ALWAYS active
   - Framework-specific (Angular, React, .NET, etc.): IF relevant files detected
   - OUTPUT "Review plan: [list of active domains]"
   - OUTPUT "Launching 3 independent reviewers: Opus, Sonnet, Haiku..."

---

## Phase 4: Parallel Independent Review (Steps 10-12)

> **Launch all three Agent calls IN PARALLEL. Each agent runs the same checklist independently.**

**Outputs:** {opus_findings}, {sonnet_findings}, {haiku_findings}

FOR EACH of steps 10, 11, 12 — use the `Agent` tool with `subagent_type="general-purpose"` and the specified `model`. Pass the IDENTICAL prompt below, substituting the actual values for all `{variables}`.

10. **Launch Opus Review** — `Agent(model="opus")`
11. **Launch Sonnet Review** — `Agent(model="sonnet")`
12. **Launch Haiku Review** — `Agent(model="haiku")`

### Subagent Review Prompt Template

```
# Independent Code Review Analysis

You are performing an independent code review. Analyze the diff and context below thoroughly.
Two other reviewers are analyzing the same code independently. Your findings will be cross-verified,
so be precise and honest — do not inflate or understate issues.

## Context

**Branch:** {branch_name}
**Base:** {base_branch}
**Commits:** {commit_count}

## File Categories

- Source files: {source_files list}
- Test files: {test_files list}
- Templates: {templates list}
- Styles: {styles list}
- Config: {config list}

## Active Review Domains

{active_review_domains}

## Diff Output

{diff_output}

## Review Checklist

Analyze the diff against EACH applicable checklist item. For every finding, provide the exact
file path and line reference from the diff (e.g., `file.ts:L42`).

### Architecture & SOLID
- Single Responsibility: Does each file/class/function have one clear purpose?
- Open/Closed: Are new features added through extension rather than modification of existing code?
- Liskov Substitution: Do subtypes properly substitute base types?
- Interface Segregation: Are interfaces small and specific?
- Dependency Inversion: Are dependencies injected, not hardcoded?
- Separation of Concerns: Business logic in services (not UI), no side effects in presentation layer
- Error Handling: Errors handled at appropriate level, user notification on failures
- Maintainability: Self-documenting names, no magic numbers/strings

### Type Safety
- No `any`, `object`, `dynamic` or equivalent catch-all types — use proper types
- Use `unknown` with type guards when type is genuinely uncertain
- Type assertions (`as` keyword in TS, casts in C#): flag if excessive (>3 occurrences)
- Non-null assertions (`!`): must be justified, flag if unjustified
- Explicit parameter and return types on public functions
- Use utility types where appropriate (Record, Partial, Pick, Omit, Readonly, etc.)

### DRY & Design Patterns
- Duplicated logic across files (apply Rule of Three — only flag if 3+ occurrences)
- Magic numbers or strings that should be constants
- Opportunities for well-known patterns (Factory, Strategy, Observer, etc.) — only if clearly beneficial
- Over-engineering: unnecessary abstractions, premature generalization

### Security
- No hardcoded credentials (password, apiKey, secret, token as string literals)
- No string concatenation in database queries (injection risk)
- No innerHTML, dangerouslySetInnerHTML, or equivalent (XSS risk)
- Input validation present on user-facing inputs
- No insecure HTTP URLs in non-comment code
- Sensitive data not logged or exposed in error responses

### Performance
- No nested loops on large datasets
- Derived/computed state used instead of recalculating in render/template
- No expensive operations (HTTP calls, complex calculations) inside loops
- Efficient data fetching (no N+1 queries, no unused data fetched)

### Test Coverage
- Each source file should have a corresponding test file
- Calculate coverage: (files with tests) / (total source files) * 100
- Test quality: Arrange-Act-Assert pattern, descriptive names ("should X when Y")
- Behavior-focused (not implementation details), proper mocking
- Edge cases: null/undefined inputs, empty collections, async errors

### Completeness & Scope
- Any TODO/FIXME/HACK/TEMP comments left in new code?
- Edge cases handled: null/undefined inputs, empty arrays, async errors
- All changes related to the branch purpose? (flag scope drift)

## Required Output Format

Return findings in EXACTLY this format. Every finding MUST have a file:line reference.

### STRENGTHS
- [CATEGORY] file.ts:L42 - Description
(list all strengths found)

### WARNINGS
- [CATEGORY] file.ts:L12 - Description
(list all warnings found)

### ISSUES
- [CATEGORY] file.ts:L88 - Description of blocking issue
(list all blocking issues found)

### TEST_COVERAGE
- coverage_percent: XX%
- missing_tests: [list of files without corresponding test file, or "None"]
- test_quality_notes: Brief assessment of test quality

### SUMMARY
- total_strengths: N
- total_warnings: N
- total_issues: N
- overall_assessment: EXCELLENT | GOOD | NEEDS_WORK | BLOCKING
- key_concern: One sentence summary of the biggest concern, or "None"
```

**CHECKPOINT** — After all three agents return:
- IF any agent returned empty: WARN "{agent_name} returned no results — continuing with remaining agents"
- IF all agents failed: REPORT "Multi-agent review failed" → Fallback: run single-agent review in main thread
- ELSE: OUTPUT "All 3 reviewers completed. Opus: {n} findings, Sonnet: {n} findings, Haiku: {n} findings"

---

## Phase 5: Cross-Verification (Steps 13-15)

> **Launch all three Agent calls IN PARALLEL. Each agent verifies the OTHER two agents' findings.**

**Outputs:** {opus_verification}, {sonnet_verification}, {haiku_verification}

13. **Launch Opus Cross-Verification** — `Agent(model="opus")` — reviews {sonnet_findings} + {haiku_findings}
14. **Launch Sonnet Cross-Verification** — `Agent(model="sonnet")` — reviews {opus_findings} + {haiku_findings}
15. **Launch Haiku Cross-Verification** — `Agent(model="haiku")` — reviews {opus_findings} + {sonnet_findings}

### Subagent Cross-Verification Prompt Template

```
# Cross-Verification Review

You are cross-verifying the code review findings of two other reviewers. Your job is to check each
finding against the actual diff and provide an honest verdict:

1. **CONFIRMED** — The finding is correct and well-reasoned
2. **DISPUTED** — The finding is incorrect, exaggerated, or misidentified (you MUST explain why)
3. **NEW** — A finding that BOTH reviewers missed (you see it in the diff but neither reported it)

Be rigorous. Do NOT rubber-stamp. Actually check each finding against the diff.
A false positive wastes developer time. A false negative lets bugs through.

## Original Context

**Branch:** {branch_name}
**Base:** {base_branch}

## Diff Output

{diff_output}

## Reviewer A Findings ({reviewer_a_name})

{reviewer_a_findings}

## Reviewer B Findings ({reviewer_b_name})

{reviewer_b_findings}

## Required Output Format

For EACH finding from both reviewers, provide a verdict. Then list any NEW findings both missed.
Use the exact format below.

### VERIFICATION_OF_{reviewer_a_name}

#### STRENGTHS
- [CONFIRMED] file.ts:L42 - {original description}
- [DISPUTED] file.ts:L15 - {original description} -- Reason: {why this is wrong}

#### WARNINGS
- [CONFIRMED] file.ts:L30 - {original description}
- [DISPUTED] file.ts:L12 - {original description} -- Reason: {why this is wrong}

#### ISSUES
- [CONFIRMED] file.ts:L88 - {original description}
- [DISPUTED] file.ts:L102 - {original description} -- Reason: {why this is wrong}

### VERIFICATION_OF_{reviewer_b_name}

(same structure as above)

### NEW_FINDINGS

#### STRENGTHS
- [NEW] file.ts:L55 - Description of missed strength

#### WARNINGS
- [NEW] file.ts:L77 - Description of missed warning

#### ISSUES
- [NEW] file.ts:L99 - Description of missed blocking issue

### CROSS_VERIFICATION_SUMMARY
- confirmed_count: N
- disputed_count: N
- new_findings_count: N
```

**CHECKPOINT** — After all three cross-verifiers return:
- IF any cross-verifier failed: WARN "{agent_name} cross-verification failed — computing confidence with available verifiers"
- ELSE: OUTPUT "Cross-verification complete. Confirmed: {n}, Disputed: {n}, New: {n}"

---

## Phase 6: Synthesis & Confidence Scoring (Steps 16-18)

**Outputs:** {synthesized_findings}, {confidence_summary}, {assessment}

16. **Merge findings** — FOR EACH unique finding (deduplicated by file:line + category):
    - COUNT how many agents originally found it (sources: 0-3)
    - COUNT how many cross-verifiers confirmed it (confirmations: 0-2)
    - CHECK if any cross-verifier disputed it (disputed: true/false + reason)
    - RECORD attribution: which agent(s) found it, which confirmed/disputed

17. **Compute confidence** — FOR EACH finding apply:

    | Confidence | Rule |
    |-----------|------|
    | **HIGH** | Found by 2+ agents, OR found by 1 + confirmed by both cross-verifiers, no disputes |
    | **MEDIUM** | Found by 1 + confirmed by 1 cross-verifier, OR found by 2+ but 1 disputes |
    | **LOW** | Found by 1 agent only, no confirmations from cross-verifiers |
    | **DISPUTED** | Found by 1 agent, explicitly disputed by a cross-verifier with reasoning |

    - NEW findings from cross-verification: START at LOW, upgrade to MEDIUM if found by 2+ cross-verifiers

18. **Overall assessment** — COUNT across all synthesized findings:
    - `blocking_issues` = COUNT of ISSUES with confidence HIGH or MEDIUM
    - `suggestions` = COUNT of WARNINGS with confidence HIGH or MEDIUM
    - `strengths` = COUNT of STRENGTHS with confidence HIGH or MEDIUM
    - `disputed_count` = COUNT of DISPUTED findings
    - IF blocking > 0 → "Blocking Issues"
    - IF suggestions > 5 → "Needs Work"
    - IF suggestions > 0 → "Good"
    - ELSE → "Excellent"
    - **CHECKPOINT**: OUTPUT summary with confidence distribution

---

## Phase 7: Final Verification Pass (Steps 19-24)

**Approach:** Act as a skeptical senior engineer — assume things can go wrong. Verify, don't trust.

19. **Edge cases handled?** — FOR EACH function: null/undefined inputs? empty arrays? async errors caught?
    - MISSING → "Unhandled edge case: {describe}"

20. **Run tests** — Detect test runner and execute: `npm test`, `dotnet test`, `pytest`, etc. SHOW actual output, do not summarize.
    - FAILING → "Tests failing: {output}"

21. **Regressions** — IDENTIFY changed functions/components → search for callers in codebase
    - IF callers could break: "Potential regression: {file} calls {changed_function}"

22. **Scope drift** — Are ALL changes related to the branch purpose?
    - Unrelated changes → "Scope drift: {describe}"

23. **DRY & design patterns** — Duplicated logic? Magic numbers? Pattern opportunities?
    - Violations → "DRY violation: {describe}" · Opportunity → "Consider {pattern}: {describe}"

24. **CHECKPOINT — Final Verdict**:
    - No gaps → "Implementation is solid." → {final_verdict} = "approved"
    - Warnings only → "Minor gaps — review before merging." → {final_verdict} = "approved with notes"
    - Blocking issues → "Address before PR." → {final_verdict} = "blocked"

---

## Phase 8: Report Compilation (Step 25)

25. **ASSEMBLE {review_report}**:

```markdown
# Multi-Agent Code Review: {branch_name}
**Branch:** {branch_name} | **Base:** {base_branch} | **Commits:** {commit_count} | **Files:** {total} | **Date:** {date}
**Status:** {assessment} | **Final Verdict:** {final_verdict}
**Review Method:** Multi-Agent (Opus + Sonnet + Haiku) with Cross-Verification

## Summary
- Overall: {assessment} | Coverage: {coverage_percent}%
- Quality: {blocking} blocking · {suggestions} suggestions · {strengths} strengths
- Confidence: {high_count} HIGH · {medium_count} MEDIUM · {low_count} LOW · {disputed_count} DISPUTED

## Agent Agreement Overview
| Category | Opus | Sonnet | Haiku | Agreement |
|----------|------|--------|-------|-----------|
| Issues Found | {n} | {n} | {n} | {overlap%}% |
| Warnings | {n} | {n} | {n} | {overlap%}% |
| Strengths | {n} | {n} | {n} | {overlap%}% |

## Issues (Must Fix)
| # | Confidence | Category | Location | Description | Found By | Verified By |
|---|-----------|----------|----------|-------------|----------|-------------|
(table of issues sorted by confidence)

## Suggestions
| # | Confidence | Category | Location | Description | Found By | Verified By |
|---|-----------|----------|----------|-------------|----------|-------------|
(table of warnings sorted by confidence)

## Strengths
| # | Confidence | Category | Location | Description | Found By |
|---|-----------|----------|----------|-------------|----------|
(table of strengths)

## Disputed Findings
(Only include if {disputed_count} > 0)

### Dispute 1: {file}:{line} — {category}
- **Opus says:** {opus_perspective}
- **Sonnet says:** {sonnet_perspective}
- **Haiku says:** {haiku_perspective}
- **Recommendation:** {which side is more convincing and why}

## Test Coverage
Coverage: {coverage_percent}% ({tested}/{total} files) — {status}
Missing tests: {missing_tests or "All files covered"}
Test quality: {test_quality_summary}

## Security: {security_findings}
## Performance: {performance_findings}

## Detailed Findings by Domain
(one section per active review domain with confidence levels)

## Stats
Source:{n} Templates:{n} Styles:{n} | Tests:{n} | Config:{n} | Commits:{n}
Review Agents: Opus + Sonnet + Haiku | Total Findings: {n} | Cross-Verified: {n}

## Recommendations
Before merge: {high_confidence_blocking_issues or "No blocking issues"}
Investigate disputes: {disputed_findings_summary or "No disputes"}
Nice to have: {medium_confidence_suggestions or "Code meets all standards"}
Follow-up: {technical_debt or "None"}

## Next Steps
- [ ] Fix blocking issues (HIGH confidence): {count}
- [ ] Investigate disputed findings: {disputed_count}
- [ ] Add missing tests: {missing_test_count}
- [ ] Review suggestions (MEDIUM+ confidence): {count}
- [ ] Run manual testing checklist

**Generated by:** Claude Code Multi-Agent Review (Opus + Sonnet + Haiku)
```

**CHECKPOINT**: All sections present, placeholders resolved → OUTPUT {review_report} to user

26. **SAVE report** — WRITE {review_report} to `.claude/reviews/{branch_name}-review-{YYYY-MM-DD}.md` (create `.claude/reviews/` directory if it doesn't exist) → OUTPUT "Report saved to .claude/reviews/{branch_name}-review-{YYYY-MM-DD}.md"

---

## Phase 9: Interactive Options (Steps 27-28)

27. **PRESENT options:**
    ```
    Multi-agent review complete! Report saved. Next action?
    1. Create TODO list           2. Show code examples
    3. Generate PR description    4. Review file in detail
    5. Show agent disagreements   6. Show agent perspectives
    7. Exit
    ```
    WAIT → SAVE as {user_choice}

28. **EXECUTE choice:**

    - `1` / "todo" → FORMAT each issue as "[{confidence}] [File:Line] {desc}" + each warning similarly → LOOP to 27
    - `2` / "examples" → FOR each HIGH/MEDIUM suggestion: READ file section → SHOW current + proposed → LOOP to 27
    - `3` / "pr" → BUILD PR template: Summary/Changes/Testing/Review Notes (include multi-agent confidence summary) → OUTPUT → LOOP to 27
    - `4` / "detail" → ASK which file → READ → ANALYZE vs standards → LOOP to 27
    - `5` / "disagreements" → LIST all DISPUTED findings with each agent's perspective + reasoning → LOOP to 27
    - `6` / "perspectives" → SHOW side-by-side: what each agent focused on, unique findings per agent, agreement areas → LOOP to 27
    - `7` / "exit" → "Review complete. Good luck with your PR!" → END

---

## Error Handling

| Error | Detection | Action |
|---|---|---|
| No commits | {commit_count} === 0 | REPORT "no changes to review" → STOP |
| Git command fails | Non-zero exit | REPORT error → SUGGEST manual fix |
| Diff too large (>100K) | Exceeds threshold | TRUNCATE to source files only, note in report |
| Diff large (>50K) | Exceeds threshold | WARN "agents may take longer" → CONTINUE |
| Unknown file types | No category match | ADD to "Other" → continue |
| Agent returns empty | Empty Agent response | WARN "agent {name} returned no results" → continue with remaining agents |
| Agent timeout | Agent takes >5 min | WARN "agent {name} timed out" → continue with available results |
| Agent output malformed | Cannot parse structured format | WARN "agent {name} output could not be parsed" → show raw output in report addendum |
| All agents fail | All 3 Agent calls fail | REPORT "Multi-agent review failed" → fallback: run single-agent review in main thread |
| Cross-verification fails | 1+ cross-verification Agents fail | Compute confidence with available data (2/3 instead of 3/3) |

---

## Success Criteria

- [ ] All commits and diff analyzed
- [ ] Files categorized correctly
- [ ] Three independent review agents launched in parallel (Opus, Sonnet, Haiku)
- [ ] All three agents returned structured findings
- [ ] Three cross-verification agents launched in parallel
- [ ] Cross-verification completed with CONFIRMED/DISPUTED/NEW verdicts
- [ ] Findings synthesized with confidence levels (HIGH/MEDIUM/LOW/DISPUTED)
- [ ] Final verification pass completed (tests, regressions, scope)
- [ ] Comprehensive report with confidence levels, agent attribution, and file:line references
- [ ] Disputed findings presented with each agent's perspective
- [ ] Assessment (Excellent/Good/Needs Work/Blocking) accurate and justified

---

## Token Budget

6 Agent calls total (3 reviews + 3 cross-verifications). Large diffs (>50K chars) increase token usage significantly.
