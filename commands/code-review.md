# Code Review

Perform a thorough code review of the current changes. Follow the steps below systematically.

## Step 1: Gather Changes

Run `git diff` to see all unstaged changes and `git diff --cached` to see staged changes. If there are no changes, check recent commits with `git log -5 --oneline` and review the latest commit with `git show HEAD`.

## Step 2: Review Each Changed File

For every modified file, evaluate the following categories:

### SOLID Violations
- Does each class/function have a single responsibility?
- Are new features added through extension rather than modification?
- Are interfaces small and specific?
- Are dependencies injected, not hardcoded?

### DRY Violations
- Is there duplicated logic that should be extracted?
- Are there copy-pasted blocks that could be a shared function?
- Apply the Rule of Three — only flag if the pattern appears 3+ times.

### Security Issues
- Is user input validated and sanitized?
- Are there hardcoded secrets, keys, or tokens?
- Is SQL built with string concatenation instead of parameterized queries?
- Are there XSS, CSRF, or injection vulnerabilities?
- Is sensitive data logged or exposed in error responses?

### Performance
- Are there N+1 query patterns?
- Are there unnecessary computations inside loops?
- Is data fetched that isn't used?
- Are there missing indexes implied by new queries?

### Naming & Readability
- Are names descriptive and intention-revealing?
- Is the code self-explanatory or does it require comments to understand?
- Are functions/methods an appropriate length?
- Is the code consistently formatted?

### Error Handling
- Are errors handled at the appropriate level?
- Are exceptions swallowed silently?
- Is there error handling for things that can't fail (over-engineering)?
- Are error messages helpful for debugging?

### Test Coverage
- Are there tests for the new/changed behavior?
- Do existing tests still pass with the changes?
- Are edge cases covered?

## Step 3: Generate Report

Present findings in this format:

```
## Code Review Report

### Critical (must fix before merge)
- [file:line] Description of the issue and why it's critical

### Warning (should fix)
- [file:line] Description and recommended improvement

### Suggestion (nice to have)
- [file:line] Description of potential improvement

### Positive Observations
- Note any particularly well-written code, good patterns, or improvements

### Summary
Overall assessment in 2-3 sentences, followed by the top 3 priority items.
```

If there are no issues found in a category, omit that section. Be specific — always reference the exact file and line number. Be constructive — explain *why* something is an issue and suggest a concrete fix.
