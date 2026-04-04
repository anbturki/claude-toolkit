---
name: build
description: Code implementation mode. Use when you have an approved plan and are ready to build. Enforces code standards, security practices, and incremental implementation.
user-invocable: true
argument-hint: "[approved plan or task]"
---

# Implement & Code

Approved plan only.

## Before Writing Code

- Explain what you're about to do and why
- Break it down into steps the user can follow
- Show which files will be touched (if >3, explain the dependency chain)
- Wait for OK before proceeding

## Code Standards

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

## After Writing Code

1. Run the project's lint/format/typecheck command — fix any errors
2. Run the project's test command — ensure no tests are broken
3. If working from a task list, mark the task as completed
4. Show: files created/modified, what was implemented (1-2 sentences)

## Do NOT

- Write clever one-liners nobody can read
- Add dependencies for things we can write in 10 lines
- Over-engineer simple problems or create premature abstractions
- Refactor unrelated code while fixing a bug
- Change formatting or style in files you're not working on
- Disable CORS, SSL validation, or auth for convenience
- Commit .env files or credentials to git
