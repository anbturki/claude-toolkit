---
name: validate
description: Validation Runner — runs the full validation suite (typecheck, lint, format, tests) and reports results. Use after implementation to verify everything is green.
user-invocable: true
allowed-tools: Bash, Read, TaskList
---

# Validate

Run the full validation suite and report clear results. No code changes — just check and report.

## Step 0: Detect Project Commands

Before running anything, detect the correct commands:
1. Read `CLAUDE.md` for validation commands
2. Check `package.json` scripts: `typecheck`, `lint`, `check`, `test`, `format`
3. Check `Makefile` for relevant targets
4. Common patterns by runtime:
   - **Bun**: `bun run typecheck`, `bun run check`, `bun test`
   - **Node/npm**: `npm run typecheck`, `npm run lint`, `npm test`
   - **pnpm**: `pnpm typecheck`, `pnpm lint`, `pnpm test`
   - **Deno**: `deno check`, `deno lint`, `deno test`

## Validation Steps

Run these in order using the detected commands:

### 1. TypeScript Type Check
```bash
# e.g., bun run typecheck, npx tsc --noEmit, pnpm typecheck
```

### 2. Lint & Format
```bash
# e.g., bun run check, npm run lint, npx biome check, npx eslint .
```

### 3. Tests
```bash
# e.g., bun test, npm test, pnpm test, npx vitest
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
