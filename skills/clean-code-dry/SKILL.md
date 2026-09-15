---
name: clean-code-dry
description: Eliminate duplication using the Rule of Three. Knows what to extract and what NOT to extract. Use when refactoring repeated code, components, or logic.
user-invocable: true
allowed-tools: Read, Edit, Grep, Glob
argument-hint: "[file path or scope]"
---

# Don't Repeat Yourself

## Scope

`$ARGUMENTS`

## Rule of Three

- First time: write it
- Second time: note the duplication but leave it
- Third time: extract into a shared function/component/constant

**Source:** Don Roberts / Martin Fowler - Rule of Three

## What to extract

- Identical code blocks -> shared function
- Similar JSX patterns -> shared component with props
- Same validation logic -> shared validator
- Same error handling -> shared error handler or middleware
- Same data transformation -> shared utility function

## What NOT to extract

- Two lines that happen to look similar but serve different purposes
- Code that might diverge in the future
- Anything where the abstraction is harder to understand than the duplication
- Single-use helpers (premature abstraction)

## Why "what NOT" matters

Premature abstraction couples unrelated things together. If two code blocks look identical today but represent different domain concepts ("compute tax" vs. "compute discount"), extracting them creates a fragile shared function that will distort the moment either side needs to change. Three similar lines is better than a premature abstraction.

## See also

- [[clean-code-srp]] - duplication often hides an extractable single responsibility
- [[clean-code-no-magic-values]] - duplicated literals are a special case (extract to constants)
- [[clean-code-naming]] - what you name the extraction matters as much as the extraction itself
