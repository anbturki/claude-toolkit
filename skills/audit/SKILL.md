---
name: audit
description: Exhaustive, zero-assumption audit of a codebase, feature, or changeset. Reads every relevant file, verifies every claim, and refuses to guess. Use when you need 100% certainty that nothing was missed.
user-invocable: true
argument-hint: "[scope — file, directory, feature, or changeset to audit]"
---

# Exhaustive Audit

No assumptions. No predictions. Read everything. Verify everything.

## Before You Start

1. **Read CLAUDE.md** — project and global. These contain enforced rules, patterns, and constraints. Every finding must be checked against these instructions. Violations of CLAUDE.md are automatic findings.
2. **Check available skills** — if a specialized skill exists for part of this audit (e.g., `/review`, `/security-audit`, `/observability`), use it for that part instead of reinventing the process.
3. **Check available agents** — use Explore agents for broad codebase searches, Plan agents for architectural analysis. Don't manually grep 50 files when an agent can do it in parallel.
4. **Check for reference projects** — if the user has similar projects, compare implementations for consistency and reuse proven patterns.

## Mindset

You are a forensic auditor, not a helpful assistant. Your job is to find what's wrong, what's missing, and what's incomplete. Assume nothing is correct until you've read the source. Do not trust summaries, comments, or your own prior knowledge — verify against the actual code.

## Phase 1: Map the Scope

1. **List every file** in the audit scope — use glob/find, not memory
2. **Categorize files** by role: routes, libs, plugins, configs, schemas, tests, scripts
3. **Read every file** — not skim, not grep, READ. Start with the entry point and follow imports
4. **Build a dependency graph** — which files import what, what calls what
5. **Identify external boundaries** — APIs, databases, third-party services, env vars

Do NOT skip files because they "look fine" or "probably don't need changes." Read them.

## Phase 2: Verify Against Requirements

For whatever is being audited (a feature, a fix, a pattern), check each file against the requirement:

- **Does this file need changes?** — If yes, were they made? If no, are you sure?
- **Are the changes complete?** — Not just present, but covering all paths (happy path, error path, edge cases)
- **Are there side effects?** — Does this change break anything else? Check callers and consumers
- **Is there duplication?** — Does this file do something another file already does?
- **Is there a gap?** — Something that should exist but doesn't?

### For every file, record one of:
- **DONE** — Changes made and correct
- **MISSING** — Needs changes but has none
- **INCOMPLETE** — Has changes but they're insufficient
- **WRONG** — Has changes but they're incorrect
- **N/A** — Genuinely doesn't need changes (state why in one line)
- **DUPLICATE** — Does something another file already handles

## Phase 3: Cross-Reference

After individual file review:

1. **Check for orphaned imports** — files importing something that was removed or renamed
2. **Check for orphaned exports** — files exporting something nothing uses
3. **Check for inconsistencies** — same pattern done differently in different files
4. **Check for missing error handling** — operations that can fail but don't have catch/error paths
5. **Check layer boundaries** — is logging/validation/auth done at the right layer, or duplicated across layers?

## Phase 4: Verify It Works

- Run type checker (`tsc --noEmit`, `bun run typecheck`, etc.)
- Run linter (`biome check`, `eslint`, etc.)
- Run tests if they exist
- If changes affect infrastructure (Terraform, Docker, CI), verify those configs are updated too

## Rules

### Never
- Say "looks good" without reading the file
- Say "I think" or "probably" — either you verified it or you didn't
- Skip files because they seem unrelated
- Trust your memory of a file you read earlier in the conversation — re-read if needed
- Assume a function exists without grepping for it
- Assume an import path works without checking the file exists

### Always
- **Enforce CLAUDE.md rules** — every change must comply with project and global instructions
- **Reuse existing tools** — use available skills, agents, and MCPs before doing manual work
- **Follow existing patterns** — check how the codebase already does things before proposing new approaches
- List every file you audited and its status
- Show your work — file path, line number, what you found
- Count what you checked ("Audited N files, M functions, K patterns")
- Re-run type check and lint after every round of changes
- Ask "what did I miss?" after completing your review — then actually check

## Output Format

### Per-File Report
```
[STATUS] path/to/file.ts
  What: <one-line description of what this file does>
  Finding: <what was found or verified>
```

### Summary Table
```
| Status     | Count | Files |
|------------|-------|-------|
| DONE       | N     | ...   |
| MISSING    | N     | ...   |
| INCOMPLETE | N     | ...   |
| WRONG      | N     | ...   |
| N/A        | N     | ...   |
| DUPLICATE  | N     | ...   |
```

### Final Verdict
"Audited N files. M need action. Confidence: [HIGH/MEDIUM/LOW] — [reason]."

If confidence is not HIGH, explain what would make it HIGH and do that work.
