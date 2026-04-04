---
name: ship
description: Validate, commit, push, and create a PR in one flow. Use when changes are ready to ship.
user-invocable: true
argument-hint: "[optional: base branch or PR description]"
---

# Ship: Validate, Commit & PR

Run checks, create branch, group commits logically, and open a PR.

## Steps

1. **Validate** — Execute the `/validate` skill. Stop if anything fails.
2. **Branch** — Create a descriptive branch from the working changes (e.g., `fix/billing-audit`, `feat/deploy-ux`). Use the diff to pick a name that fits all changes.
3. **Commit** — Group changes into small, logical commits by domain/file similarity. Use `git add <specific files>` per commit. Write clear commit messages: what changed and why. No AI references.
4. **Push & PR** — Push branch, create PR via `gh pr create`. Keep PR title short (<70 chars), body is a brief summary (bulleted list of what changed). No test plans. No AI references.

## Rules

- Never mention Claude, AI, Anthropic, or "generated" in commits or PR
- No `Co-Authored-By` lines
- No emoji in commits or PR
- Group by logical change, not by file type
- Specific `git add` per commit — never `git add -A` or `git add .`
- If `/validate` fails, fix first, don't ship broken code
