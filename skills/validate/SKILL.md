---
name: validate
description: Validation Runner — runs the full validation suite (typecheck, lint, format, tests) and reports results. Use after implementation to verify everything is green.
user-invocable: true
allowed-tools: Bash, Read, TaskList
---

# Validate

Run the full validation suite and report clear results. No code changes — just check and report.

## Validation Steps

Run these commands in order and report results:

### 1. TypeScript Type Check
```bash
bun run typecheck
```

### 2. Lint & Format (Biome)
```bash
bun run check
```

### 3. Tests
```bash
bun test
```

## Output Format

```
Validation Results
==================

Typecheck:  PASS / FAIL (N errors)
Lint:       PASS / FAIL (N issues)
Format:     PASS / FAIL (N files need formatting)
Tests:      PASS / FAIL (X passed, Y failed)

Overall:    ALL GREEN / N ISSUES FOUND
```

If any step fails, show the exact error output so the developer can fix it.

If ALL GREEN, check the task list and say: "All checks pass. Next steps: [suggest next action based on task list]."
