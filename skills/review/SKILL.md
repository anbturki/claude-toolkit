---
name: review
description: Code review and quality audit mode. Use to review existing code, audit a system, or do a sweep before release. Checks for bugs, duplication, architecture issues, type safety, and performance.
user-invocable: true
argument-hint: "[file, feature, or scope to review]"
---

# Review & Audit

Be thorough. Take your time.

## Check For

- Errors, bugs, and problematic areas
- Code duplication — consolidate similar functions
- Missing elements, inconsistencies, and misalignments
- Unhandled edge cases and race conditions
- Hacky approaches that need fixing
- Overall correctness and quality

## Code Standards (Enforce These)

- Implement only what's immediately needed — solve the immediate problem, nothing more
- Follow existing patterns exactly — check how similar features are built and do the same
- Reuse existing code (DRY) — identify reusable utilities before writing anything new
- Each component/file should have ONE clear, focused purpose (Single Responsibility)
- Keep functions small and single-purpose
- Favor readability over cleverness — write code a junior can maintain
- Add comments for non-obvious logic; use meaningful variable and function names
- No `any` types, no untyped `object`, no TypeScript assertion bypasses
- Minimize DOM nesting — keep structure flat, no redundant wrapper divs
- Keep component files under 200 lines when possible; separate logic from presentation
- Use composition over inheritance
- Use straightforward REST endpoints — no fancy patterns
- Use parameterized queries (never string concatenation for SQL)
- Validate all user input — assume it's hostile
- Never log or expose secrets, tokens, or credentials
- Never hardcode API keys, passwords, or secrets

## Flag as Violations

- Clever one-liners nobody can read
- Dependencies added for things we can write in 10 lines
- Over-engineered simple problems or premature abstractions
- Unrelated code refactored while fixing a bug
- Formatting or style changes in files not being worked on
- Disabled CORS, SSL validation, or auth for convenience
- `.env` files or credentials committed to git

## Architecture & Performance

- Check separation of concerns and single responsibility adherence
- Flag unnecessary abstractions
- Review repository/database query efficiency
- Check for memory leaks and unnecessary re-renders
- Identify client-to-server conversion opportunities
- Identify DOM simplification opportunities

## Type Safety & Code Quality

- Verify all interfaces are properly typed (no `any`, no untyped `object`)
- Check for proper error boundaries
- Validate API response handling
- Review state management patterns
- Run type checks, linting, and format verification

## Output Format

For each issue found:
```
[SEVERITY] file:line — Description
  Current: <what the code does>
  Better: <what it should do>
  Why: <brief explanation>
```

Severity levels:
- **BUG** — Will cause incorrect behavior
- **WARN** — Could cause problems under certain conditions
- **IMPROVE** — Works but could be better
- **STYLE** — Minor style inconsistency (only flag if pattern violation)

### Summary

"Reviewed N files. Found: X bugs, Y warnings, Z improvements."

Only flag important issues. No nitpicks. No praise. Be direct.

Base all recommendations on established practices — do NOT over-engineer, predict future needs, or propose creative/novel solutions.
