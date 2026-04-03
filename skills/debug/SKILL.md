---
name: debug
description: Testing and debugging mode. Use when something breaks or when writing/reviewing tests. Enforces evidence-first debugging, root cause analysis, and comprehensive test coverage.
user-invocable: true
argument-hint: "[error, bug description, or test scope]"
---

# Test & Debug

Evidence first, fix second.

## When Debugging

- Read the full error message and stack trace first
- Search the codebase for related patterns (grep, find, glob)
- Check if this error has been encountered before in the project
- Reproduce the issue before attempting a fix
- Trace the data flow end-to-end before modifying it
- Check tests and git history to understand intended behavior
- Explain the root cause to the user, not just the fix
- Verify the fix doesn't break adjacent functionality

### Do NOT

- Guess at fixes without reading the error
- Apply shotgun fixes (changing multiple things hoping one works)
- Suppress errors or add try/catch as a "fix"
- Delete code you don't understand
- Skip testing after the fix

## Testing Standards

- Write tests BEFORE implementation when possible (TDD)
- Every bug fix must include a regression test
- Run existing tests before and after changes
- Never disable or skip failing tests without explaining why
- Test edge cases, error handling paths, and failure scenarios — not just the happy path
- Use the testing patterns already in the codebase
- Test at the right level (unit vs integration vs e2e)
- Mock external services, not internal logic
- Ensure tests are independent, isolated, deterministic, readable, and maintainable

### Do NOT

- Write tests that only pass for hardcoded values
- Skip tests because "it's a small change"
- Write tests that depend on execution order
- Test implementation details instead of behavior
