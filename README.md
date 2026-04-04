# Claude Toolkit

30 production-tested skills and 8 specialized agents for AI coding agents.

## Install

```bash
# All skills + agents
npx skills add anbturki/claude-toolkit

# Pick specific ones
npx skills add anbturki/claude-toolkit --skill build --skill review --skill elysia-route

# List available
npx skills add anbturki/claude-toolkit --list
```

Or clone manually:

```bash
git clone https://github.com/anbturki/claude-toolkit.git /tmp/ct
cp -r /tmp/ct/skills/* .claude/skills/   # project skills
cp -r /tmp/ct/agents/* .claude/agents/   # project agents
cp -r /tmp/ct/skills/* ~/.claude/skills/  # global skills
cp -r /tmp/ct/agents/* ~/.claude/agents/  # global agents
rm -rf /tmp/ct
```

## Skills

### Development Workflow

| Skill | What it does |
|-------|-------------|
| [**research**](skills/research/SKILL.md) | Study the codebase, find patterns, get approval before writing code |
| [**build**](skills/build/SKILL.md) | Implementation mode with enforced standards and security |
| [**debug**](skills/debug/SKILL.md) | Evidence-first debugging and root cause analysis |
| [**review**](skills/review/SKILL.md) | Code audit — bugs, duplication, type safety, performance |
| [**validate**](skills/validate/SKILL.md) | Run typecheck, lint, format, tests (auto-detects tooling) |
| [**commit**](skills/commit/SKILL.md) | Stage and commit with clean messages following conventions |
| [**create-pr**](skills/create-pr/SKILL.md) | Create a GitHub PR with structured summary |
| [**ship**](skills/ship/SKILL.md) | Validate + commit + branch + PR in one flow |

### Architecture & Quality

| Skill | What it does |
|-------|-------------|
| [**system-design**](skills/system-design/SKILL.md) | Design APIs, types, schemas, and interfaces before coding |
| [**refactor**](skills/refactor/SKILL.md) | Clean code — SOLID, deduplication, complexity reduction |
| [**security-audit**](skills/security-audit/SKILL.md) | Find injection, auth bypasses, secret leaks, DoS vectors |

### Planning & Research

| Skill | What it does |
|-------|-------------|
| [**plan-feature**](skills/plan-feature/SKILL.md) | Create PRD + phased implementation plan for a feature |
| [**plan-tasks**](skills/plan-tasks/SKILL.md) | Break work into atomic sub-tasks, track progress |
| [**deep-research**](skills/deep-research/SKILL.md) | Evidence-based investigation with source validation |
| [**discover**](skills/discover/SKILL.md) | Product research with structured docs and options analysis |

### Documentation & Communication

| Skill | What it does |
|-------|-------------|
| [**write-docs**](skills/write-docs/SKILL.md) | Generate specs, guides, architecture docs, decision records |
| [**explain**](skills/explain/SKILL.md) | Teaching mode — explain code, verify comprehension |

### Scaffolding — Backend

| Skill | What it does |
|-------|-------------|
| [**elysia-route**](skills/elysia-route/SKILL.md) | Scaffold Elysia API route with CRUD, validation, and auth |
| [**elysia-service**](skills/elysia-service/SKILL.md) | Scaffold service module for Elysia/Bun backend |
| [**nextjs-route**](skills/nextjs-route/SKILL.md) | Scaffold Next.js API route (App Router or Pages Router) |
| [**create-error**](skills/create-error/SKILL.md) | Scaffold custom error classes following project patterns |

### Scaffolding — Frontend

| Skill | What it does |
|-------|-------------|
| [**build-ui**](skills/build-ui/SKILL.md) | Frontend implementation with component reuse and UI pattern compliance |
| [**nextjs-feature**](skills/nextjs-feature/SKILL.md) | Scaffold Next.js feature with components, hooks, and server/client split |
| [**react-hook**](skills/react-hook/SKILL.md) | Scaffold React hooks for queries and mutations (TanStack Query, SWR, Apollo) |

### Scaffolding — Data & Validation

| Skill | What it does |
|-------|-------------|
| [**drizzle-schema**](skills/drizzle-schema/SKILL.md) | Scaffold Drizzle ORM schema with tables, relations, and indexes |
| [**zod-schema**](skills/zod-schema/SKILL.md) | Scaffold Zod validation schemas with inferred TypeScript types |
| [**create-constant**](skills/create-constant/SKILL.md) | Define typed constants, enums, and config — replace magic strings and numbers |

### Specialized

| Skill | What it does |
|-------|-------------|
| [**manage-secrets**](skills/manage-secrets/SKILL.md) | Auto-detect and manage secrets across providers (GitHub, AWS, Vault, Doppler, K8s, etc.) |
| [**posthog-logging**](skills/posthog-logging/SKILL.md) | PostHog logging via OTLP — pino transport setup, structured logs, events, user linking, troubleshooting |
| [**audit**](skills/audit/SKILL.md) | Exhaustive zero-assumption audit — reads every file, enforces CLAUDE.md, reuses skills/agents |

## Agents

Specialized sub-agents that run in isolated contexts with constrained tools and focused roles.

| Agent | What it does |
|-------|-------------|
| [**researcher**](agents/researcher.md) | Deep research — studies codebase patterns and proven solutions before recommending |
| [**code-auditor**](agents/code-auditor.md) | Read-only audit for bugs, duplication, edge cases, type safety, and security |
| [**architecture-reviewer**](agents/architecture-reviewer.md) | Read-only structural review — separation of concerns, patterns, dependencies |
| [**planner**](agents/planner.md) | Creates implementation proposals with options, tradeoffs, and waits for approval |
| [**refactorer**](agents/refactorer.md) | Improves code quality — deduplication, simplification, cleanup — without changing behavior |
| [**pre-impl-checker**](agents/pre-impl-checker.md) | Verifies CLAUDE.md compliance and identifies reusable patterns before writing code |
| [**type-guardian**](agents/type-guardian.md) | TypeScript audit — finds and fixes `any` types, missing definitions, type duplication |
| [**test-runner**](agents/test-runner.md) | Runs tests, reports failures with context, checks coverage, flags untested paths |

## Compatibility

Claude Code, Codex CLI, Gemini CLI, Cursor, Windsurf, OpenCode, Cline — any agent supporting SKILL.md.

## License

MIT
