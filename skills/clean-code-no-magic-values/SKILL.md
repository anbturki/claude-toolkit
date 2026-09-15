---
name: clean-code-no-magic-values
description: Replace magic numbers and strings with named constants and const objects. Knows what does and does not need extracting. Use when reviewing code with bare literals.
user-invocable: true
allowed-tools: Read, Edit, Grep, Glob
argument-hint: "[file path or scope]"
---

# No Magic Values

A magic value is a literal whose meaning isn't visible at the call site. Names make code searchable, refactorable, and self-documenting.

## Scope

`$ARGUMENTS`

## Magic numbers -> named constants

```typescript
// BAD
setTimeout(fn, 86400000);
if (retries > 3) throw new Error("...");

// GOOD
const ONE_DAY_MS = 86_400_000;
const MAX_RETRIES = 3;
setTimeout(fn, ONE_DAY_MS);
if (retries > MAX_RETRIES) throw new Error("...");
```

## Magic strings -> const objects

```typescript
// BAD
if (status === "pending") { ... }
if (status === "running") { ... }

// GOOD
const DeploymentStatus = {
  PENDING: "pending",
  RUNNING: "running",
} as const;

if (status === DeploymentStatus.PENDING) { ... }
```

## What needs extracting

- Status values, error codes, event names -> const objects (check shared/common packages first)
- Timeout durations, retry counts, limits -> named constants at file or module level
- URLs, paths, endpoints -> config or constants
- Error messages used in multiple places -> error constants
- Array indices with meaning (`items[0]`, `parts[2]`) -> destructure with names

## What does NOT need extracting

- `0`, `1`, `-1` in obvious contexts (array index, increment, comparison)
- `true`, `false` in obvious boolean contexts
- Empty string `""` for initialization
- Port numbers in Docker/config files (they ARE the config)
- Test fixtures and example data

## See also

- [[create-constant]] - the scaffolder that does this extraction (patterns, file location, type derivation)
- [[clean-code-naming]] - UPPER_SNAKE_CASE for constants
- [[clean-code-dry]] - duplicated literals = a constant waiting to be born

**Source:** Clean Code, Ch. 17 (G25) - "Replace Magic Numbers with Named Constants"
