---
name: clean-code
description: Run a clean-code pass over the target scope. Orchestrates the focused clean-code-* rule skills (readability, srp, size, dry, no-magic-values, complexity, naming, comments) in priority order. Use when you want one entry point for a full clean-code review and cleanup.
user-invocable: true
allowed-tools: Read, Write, Edit, Grep, Glob, Agent, Skill, Bash(ls *), Bash(npx *), Bash(bunx *), Bash(pnpm *)
argument-hint: "[file path, feature name, or 'full' for entire codebase]"
model: claude-opus-4-6
context: fork
---

# Clean Code Pass

You are a senior engineer performing a code quality pass. Goal: make the code readable, modular, and maintainable - not clever, not over-engineered.

**Scope:** `$ARGUMENTS`

This skill is a **thin orchestrator**. Each rule lives in its own focused skill - this one walks them in priority order.

## Zero code smells

The bar is **no code smells left in scope.** Every code smell you find is a finding that must be fixed (or, if a fix is genuinely out of scope or risky, flagged explicitly in the output - never silently left). The priority table below maps the common smells to the skill that fixes each. But the catalog isn't exhaustive: if you spot a smell that no single rule names - feature envy, a data clump, a long parameter list, primitive obsession, shotgun surgery, a leaky abstraction, dead code, speculative generality - treat it as a finding too, attribute it to the closest rule skill, and fix it. Don't pass over a smell just because there's no row for it.

## Before starting

### 1. Understand the codebase

Read `CLAUDE.md` for project conventions. Read the target files and their imports/dependents to understand the full picture before changing anything.

### 2. Understand existing patterns

Check how similar code is structured elsewhere in the project. Your refactoring must follow existing patterns - don't introduce new organizational ideas.

### 3. Scope the work

- **Single file:** analyze and refactor that file
- **Feature name:** find all files in that feature, analyze and refactor
- **`full`:** run analysis agents across the project's source directories, then refactor based on findings

## Analysis phase

Launch a `code-optimizer` (or `code-auditor`) agent on the target scope. The agent returns a prioritized list of findings. Wait for results before changing anything.

If the scope is large, launch multiple agents in parallel - one per directory or feature area.

## Apply rules in this priority order

For each finding, **invoke the relevant focused skill via the Skill tool**, passing the file path or scope. The rule skill returns specific guidance for its rule; apply that guidance, then move to the next priority.

| Priority | Concern | Skill to invoke |
|---|---|---|
| 1 | Hard-to-read code (inline conditions, ternary chains, deep nesting) | `clean-code-readability` |
| 2 | Functions / components / files doing too much | `clean-code-srp` |
| 3 | Oversized functions / files / components (line-count thresholds) | `clean-code-size` |
| 4 | Duplication of logic, components, or validators | `clean-code-dry` |
| 5 | Magic numbers and strings | `clean-code-no-magic-values` |
| 6 | High cyclomatic / cognitive complexity | `clean-code-complexity` |
| 7 | TypeScript safety holes (`any`, assertions) | `typescript-no-any` |
| 8 | Bad naming | `clean-code-naming` |
| 9 | Comment noise (restatements, dead-code comments, banner blocks, stale refs) | `clean-code-comments` |

Address higher priorities first - fixing readability often dissolves complexity findings, and applying SRP often dissolves duplication findings.

## Implementation phase (one file at a time)

For each file in scope:

1. Read the file
2. Apply all planned changes for that file
3. Verify the file still works (no broken imports, no missing references)
4. Move to the next file

## Verification

After all changes:

1. Run the project's lint/format command (e.g., `biome check --write`, `eslint --fix`, `prettier --write`)
2. Run the project's typecheck command (e.g., `tsc --noEmit`)
3. Run the project's test suite if available
4. Report: files changed, what was improved, any issues found

## What NOT to do

- Don't refactor code you weren't asked to touch (unless it's in scope)
- Don't change formatting or style in files you're not refactoring
- Don't add comments to explain bad code - fix the code
- Don't create abstractions for single-use code
- Don't rename things that are already clear
- Don't move code between files unless it clearly violates SRP
- Don't add error handling for impossible scenarios
- Don't add logging, metrics, or telemetry unless asked
- Don't change public APIs without flagging it
- Don't break existing tests

## Output Summary

```
## Clean Code Pass Complete

### Changes Made
- [file]: [what changed - 1 sentence]

### Extracted
- [new file]: [purpose - 1 sentence]

### Skills Applied
- clean-code-readability: N findings
- clean-code-srp: N findings
- clean-code-dry: N findings
- clean-code-no-magic-values: N findings
- clean-code-complexity: N findings
- typescript-no-any: N findings
- clean-code-naming: N findings
- clean-code-comments: N findings

### Verification
- Lint: pass/fail
- TypeScript: pass/fail
- Tests: pass/fail (if applicable)
```
