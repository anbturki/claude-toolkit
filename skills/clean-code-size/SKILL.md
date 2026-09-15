---
name: clean-code-size
description: Keep units small. Enforces canonical size limits for functions, source files, React components, and modules. Single source of truth for line-count thresholds across the toolkit. Use when reviewing for bloat or auditing file/function/component sizes.
user-invocable: true
allowed-tools: Read, Edit, Grep, Glob, Bash(find *), Bash(wc *), Bash(sort *)
argument-hint: "[file path or scope]"
---

# Size Limits

Small units are easier to read, test, name, and reuse. When a unit grows past its soft limit it usually means it took on more than one responsibility. Enforce thresholds at review time, not after the next ten changes.

## Scope

`$ARGUMENTS`

## Canonical thresholds

| Unit | Soft (target) | Hard (audit immediately) | Notes |
|---|---|---|---|
| **Function / method** | <= 20 lines | <= 50 lines | Clean Code Ch.3 standard. Past 50, the function is doing too much. |
| **Function parameters** | <= 3 | <= 4 | Past 3, group related params into an object. |
| **React component file** | <= 150 lines | <= 200 lines | Includes hooks + JSX. Past 150 -> extract sub-components. |
| **Non-component source file** | <= 300 lines | <= 500 lines | One concept per file. Past 300 -> consider splitting; past 500 -> almost always a missed split. |
| **Any file** | n/a | > 1000 lines | Hard fail. Split now. |
| **JSX return block** | <= 10 lines | <= 20 lines | Past 10 -> extract sub-components. |
| **Nesting depth** | <= 3 levels | <= 4 levels | Past 3 -> guard clauses, extracted predicates, early returns. |

These thresholds are deliberate ceilings, not aspirations. Code commonly lives below them; that's good.

## How to measure

```bash
# Find the longest files in a directory
find <scope> -name '*.ts' -o -name '*.tsx' -o -name '*.py' -o -name '*.go' | xargs wc -l | sort -nr | head -20

# Per language (TS/TSX):
find src -name '*.ts' -o -name '*.tsx' | xargs wc -l | sort -nr | head -20
```

For function size, use the project's complexity tool when available (ESLint `max-lines-per-function`, Python `radon cc`, Go `gocyclo`). Otherwise visual inspection.

## What to do when you find an oversized unit

### Oversized function (> 50 lines)
1. Find the natural break points - usually after a guard-clause block, before a transformation, or before a return.
2. Extract each block into a named helper function. The helper name describes the WHY.
3. The top-level function should now read like a high-level summary of the steps.

### Oversized file (> 500 lines)
1. Group related functions / types / constants into separate concerns.
2. Look for "and" in the file's purpose ("user types and user API and user formatters") - that's a 3-file split waiting to happen.
3. Extract each concern into its own file. Tests follow the file they test.

### Oversized component (> 200 lines)
1. Pull data-fetching out into a hook ([[react-hooks]]).
2. Pull stateless presentation blocks out into sub-components.
3. Pull derived data / computations above the return into named consts.
4. The remaining component should be a wiring shell.

### Too many parameters (> 4)
1. Group related params into an options object: `fn({ user, project, role })` instead of `fn(user, project, role, ...)`.
2. If the params come from a single domain object, pass the object itself.

### Nesting > 3 deep
1. Invert with guard clauses (`if (!user) return;`).
2. Extract the inner block into a named predicate or helper.
3. Replace nested switch/if-else trees with lookup objects.

## What does NOT need extracting

- A 60-line function that is one cohesive sequence with no natural break points - extracting just to hit a number creates fragmented call chains that are harder to follow.
- A 350-line file where the contents are tightly cohesive (e.g. a single complex schema, a generated file, a long-but-flat config object).
- A 200-line component that is mostly a tall flat JSX tree with no logic - splitting it into 10 wrappers is worse than leaving it.

The thresholds exist to flag suspicion, not to force a refactor every time. Use judgment: if the unit is over the limit AND has multiple responsibilities OR is hard to read - split it. If it's just long but cohesive - leave it and add a comment about why.

## Inline complexity (cross-reference)

Bloat is not just lines. Complex inline expressions hurt readability even at low line counts:

- Inline complex conditions, chained ternaries, clever one-liners -> [[clean-code-readability]]
- Inline object literals, inline event handlers, inline component definitions in JSX -> [[react-jsx]] and [[react-performance]]

A 30-line function with a 12-condition inline ternary is worse than a 60-line function with five guard clauses.

## See also

- [[clean-code-srp]] - oversized units almost always violate single responsibility; fix SRP and size usually resolves itself
- [[clean-code-complexity]] - cyclomatic/cognitive thresholds; size and complexity are correlated but distinct
- [[clean-code-readability]] - line count doesn't capture inline complexity; fix readability separately
- [[react-components]] - React-specific component structure rules that build on these size limits
- [[react-hooks]] - extract data-fetching out of oversized components into hooks
- [[clean-code]] - the orchestrator that invokes this skill among others

**Source:** Robert C. Martin, *Clean Code*, Ch. 3 (Functions) and Ch. 5 (Formatting).
