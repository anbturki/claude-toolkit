# Claude Toolkit

47 production-tested skills and 9 specialized agents for AI coding agents.

## Install

```bash
# All skills + agents
npx skills add anbturki/claude-toolkit

# Pick specific ones
npx skills add anbturki/claude-toolkit --skill build --skill review --skill clean-code-srp

# List available
npx skills add anbturki/claude-toolkit --list
```

Or clone manually:

```bash
git clone https://github.com/anbturki/claude-toolkit.git /tmp/ct
cp -r /tmp/ct/skills/* .claude/skills/     # project skills
cp -r /tmp/ct/agents/* .claude/agents/     # project agents
cp -r /tmp/ct/skills/* ~/.claude/skills/   # global skills
cp -r /tmp/ct/agents/* ~/.claude/agents/   # global agents
rm -rf /tmp/ct
```

## Skills

### Development Workflow

| Skill | What it does |
|-------|-------------|
| [**research**](skills/research/SKILL.md) | Study the codebase, find patterns, get approval before writing code |
| [**build**](skills/build/SKILL.md) | Implementation mode with enforced standards and security |
| [**debug**](skills/debug/SKILL.md) | Evidence-first debugging and root cause analysis |
| [**review**](skills/review/SKILL.md) | Code audit - bugs, duplication, type safety, performance |
| [**validate**](skills/validate/SKILL.md) | Run typecheck, lint, format, tests (auto-detects tooling) |
| [**commit**](skills/commit/SKILL.md) | Stage and commit with clean messages following conventions |
| [**create-pr**](skills/create-pr/SKILL.md) | Create a GitHub PR with structured summary |
| [**ship**](skills/ship/SKILL.md) | Validate + commit + branch + PR in one flow |

### Architecture & Quality

| Skill | What it does |
|-------|-------------|
| [**system-design**](skills/system-design/SKILL.md) | Design APIs, types, schemas, and interfaces before coding |
| [**clean-architecture**](skills/clean-architecture/SKILL.md) | Layer-level boundaries: Dependency Rule, dependency cycles, framework/IO leakage into the core, ports & adapters, component cohesion/coupling (Uncle Bob) |
| [**clean-code**](skills/clean-code/SKILL.md) | Orchestrator: run a full clean-code pass over a scope (composes the clean-code-* rule skills) |
| [**security-audit**](skills/security-audit/SKILL.md) | Find injection, auth bypasses, secret leaks, DoS vectors |
| [**deep-audit**](skills/deep-audit/SKILL.md) | The single comprehensive audit: clean code, SOLID + design patterns, architecture boundaries, comment hygiene, docs, memory, regression, security - across dimensions, with fix + verify cycle |
| [**full-audit**](skills/full-audit/SKILL.md) | deep-audit's parallel form: fans out one isolated `audit-runner` sub-agent per quality skill (security, architecture, every clean-code rule, TypeScript, React) so none is skipped. Report-only then offers fixes |
| [**verify-claims**](skills/verify-claims/SKILL.md) | Validate a built feature/implementation/doc against primary sources (official docs, the dependency's own types/source, specs, `--help`, web research): confirms nothing is fabricated (invented APIs, nonexistent fields/flags, bad citations) and that it follows current best practice. Tags each claim Verified / Outdated / Unverified / Fabricated with the source cited |

### Clean Code

Focused, single-rule skills - composable. The `clean-code` skill above orchestrates these in priority order.

| Skill | What it does |
|-------|-------------|
| [**clean-code-readability**](skills/clean-code-readability/SKILL.md) | No inline complex conditions, no ternary chains, no clever one-liners, no deep nesting |
| [**clean-code-srp**](skills/clean-code-srp/SKILL.md) | Single responsibility for functions, components, files, and layers. No boolean flag parameters |
| [**clean-code-size**](skills/clean-code-size/SKILL.md) | Canonical line-count thresholds for functions, files, components, and nesting. Single source of truth for size limits |
| [**clean-code-dry**](skills/clean-code-dry/SKILL.md) | Rule of Three. What to extract, what NOT to extract |
| [**clean-code-no-magic-values**](skills/clean-code-no-magic-values/SKILL.md) | Replace magic numbers with named constants, magic strings with const objects |
| [**clean-code-complexity**](skills/clean-code-complexity/SKILL.md) | Cyclomatic <= 10, cognitive <= 15. Guard clauses, lookup objects, extracted predicates |
| [**clean-code-naming**](skills/clean-code-naming/SKILL.md) | Intent-revealing names, verb phrases, question-form booleans, UPPER_SNAKE constants |
| [**clean-code-comments**](skills/clean-code-comments/SKILL.md) | Regex-sweep a codebase for comment noise - delete restatements, banners, dead-code, stale refs; keep only short WHY comments |

### TypeScript

| Skill | What it does |
|-------|-------------|
| [**typescript-no-any**](skills/typescript-no-any/SKILL.md) | Ban `any`, untyped `object`, `as Type` bypasses, `@ts-ignore` |
| [**typescript-types**](skills/typescript-types/SKILL.md) | Discriminated unions over boolean flags, `as const` over `enum`, utility types |
| [**typescript-narrowing**](skills/typescript-narrowing/SKILL.md) | `unknown` + type guards instead of `any`, schema validators for complex shapes |

### React

| Skill | What it does |
|-------|-------------|
| [**react-components**](skills/react-components/SKILL.md) | Reuse-first inventory, max 150 lines, separate data from presentation |
| [**react-jsx**](skills/react-jsx/SKILL.md) | Declarative scannable JSX - no inline logic, no ternary chains, no inline objects/handlers |
| [**react-hooks**](skills/react-hooks/SKILL.md) | Scaffold data hooks (TanStack Query, SWR, Apollo) for queries and mutations |
| [**react-performance**](skills/react-performance/SKILL.md) | Stable keys, useMemo/useCallback only when needed, never define components inside components |
| [**react-accessibility**](skills/react-accessibility/SKILL.md) | Semantic HTML, keyboard accessibility, labels, alt text, aria-label |

### Next.js

| Skill | What it does |
|-------|-------------|
| [**nextjs-route**](skills/nextjs-route/SKILL.md) | Scaffold Next.js API route (App Router or Pages Router) |
| [**nextjs-feature**](skills/nextjs-feature/SKILL.md) | Feature folder structure with proper Server / Client component split |
| [**nextjs-og-images**](skills/nextjs-og-images/SKILL.md) | Generate favicons, app icons, and OG/Twitter share images via `next/og` - embedding real brand fonts under the 500KB bundle limit via font subsetting, verified by rendering |
| [**nextjs-seo**](skills/nextjs-seo/SKILL.md) | Metadata API, viewport, `robots.ts`/`sitemap.ts`/`manifest.ts`, and JSON-LD structured data |

### Planning & Research

| Skill | What it does |
|-------|-------------|
| [**plan-feature**](skills/plan-feature/SKILL.md) | Create PRD + phased implementation plan for a feature |
| [**plan-tasks**](skills/plan-tasks/SKILL.md) | Break work into atomic sub-tasks, track progress |
| [**deep-research**](skills/deep-research/SKILL.md) | Evidence-based investigation - quick inline answer, single-file research doc, or multi-file product-discovery folder |

### Documentation & Communication

| Skill | What it does |
|-------|-------------|
| [**write-docs**](skills/write-docs/SKILL.md) | Generate specs, guides, architecture docs, decision records |
| [**archive-docs**](skills/archive-docs/SKILL.md) | Move deprecated/finished docs out of a project into the vault, organized by project/category under archive/, with provenance stamps and an updated index |
| [**explain**](skills/explain/SKILL.md) | Teaching mode - explain code, verify comprehension |

### GitHub Project Management

Issues, Projects (v2), and Wiki are complementary GitHub primitives, not
interchangeable ones - an issue is the task, a project is a board view over
issues, a wiki is a separate git repo for narrative docs with no API. Three
focused skills plus one migration skill that composes them.

| Skill | What it does |
|-------|-------------|
| [**github-issues**](skills/github-issues/SKILL.md) | Create, triage, and manage issues - labels, milestones, assignees, search, close with reason |
| [**github-projects**](skills/github-projects/SKILL.md) | Create/manage a Project (v2) board - custom fields, adding issues as items, setting status via the field/option-ID mechanics `gh` requires |
| [**github-wiki**](skills/github-wiki/SKILL.md) | Publish and maintain a repo's wiki via git against `<repo>.wiki.git` - only after confirming docs shouldn't stay in-repo (note: GitHub Wikis are unavailable for private repos owned by an organization on the Free plan) |
| [**github-migrate**](skills/github-migrate/SKILL.md) | One-time move of local tasks/requests/docs (any convention: notes/tasks/, a vault-style dir, TODO.md, docs/) onto the three skills above |

### ClickUp Project Management

An alternative to the GitHub trio above when tasks/docs need to span multiple
GitHub orgs/accounts under one roof, or when GitHub Wiki's Free-plan
restriction is a blocker. Uses ClickUp's official MCP server
(`mcp.clickup.com`) with a raw-REST-API fallback for the handful of
operations that server doesn't expose.

| Skill | What it does |
|-------|-------------|
| [**clickup-tasks**](skills/clickup-tasks/SKILL.md) | Task/List/Folder/Space management via the ClickUp MCP tools, hierarchy best practices, and the personal-API-token REST fallback for space create/delete/move (not exposed by the MCP server) |
| [**clickup-docs**](skills/clickup-docs/SKILL.md) | ClickUp Docs as a wiki - nested pages, Home landing page convention, and the markdown-formatting rules that avoid mangled imports (no hard-wrapped list items, no fenced-code language tags, no toggle blocks) |

### Scaffolding - Backend & Data

| Skill | What it does |
|-------|-------------|
| [**create-error**](skills/create-error/SKILL.md) | Scaffold custom error classes following project patterns |
| [**create-constant**](skills/create-constant/SKILL.md) | Define typed constants, enums, and config - replace magic strings and numbers |
| [**zod-schema**](skills/zod-schema/SKILL.md) | Scaffold Zod validation schemas with inferred TypeScript types |

### Specialized

| Skill | What it does |
|-------|-------------|
| [**manage-secrets**](skills/manage-secrets/SKILL.md) | Auto-detect and manage secrets across providers (GitHub, AWS, Vault, Doppler, K8s) |

### Stacks (opt-in)

These skills target specific frameworks/services and are **not installed by default**. Add them explicitly with `--skill stacks/<name>`.

| Skill | Stack |
|-------|-------|
| [**stacks/elysia-route**](skills/stacks/elysia-route/SKILL.md) | Elysia API route (Bun) |
| [**stacks/elysia-service**](skills/stacks/elysia-service/SKILL.md) | Elysia service module |
| [**stacks/drizzle-schema**](skills/stacks/drizzle-schema/SKILL.md) | Drizzle ORM schema with tables, relations, indexes |
| [**stacks/posthog-logging**](skills/stacks/posthog-logging/SKILL.md) | PostHog logging via OTLP - pino transport, structured logs, events |

## Agents

Specialized sub-agents that run in isolated contexts with constrained tools and focused roles.

| Agent | What it does |
|-------|-------------|
| [**researcher**](agents/researcher.md) | Deep research - studies codebase patterns and proven solutions before recommending |
| [**code-auditor**](agents/code-auditor.md) | Read-only audit for bugs, duplication, edge cases, type safety, and security |
| [**architecture-reviewer**](agents/architecture-reviewer.md) | Read-only structural review - separation of concerns, patterns, dependencies |
| [**planner**](agents/planner.md) | Creates implementation proposals with options, tradeoffs, and waits for approval |
| [**refactorer**](agents/refactorer.md) | Improves code quality - deduplication, simplification, cleanup - without changing behavior |
| [**pre-impl-checker**](agents/pre-impl-checker.md) | Verifies CLAUDE.md compliance and identifies reusable patterns before writing code |
| [**type-guardian**](agents/type-guardian.md) | TypeScript audit - finds and fixes `any` types, missing definitions, type duplication |
| [**test-runner**](agents/test-runner.md) | Runs tests, reports failures with context, checks coverage, flags untested paths |
| [**audit-runner**](agents/audit-runner.md) | Runs exactly one skill against a scope, evidence-cited, report-only. Spawned in parallel by `full-audit` so no skill is skipped |

## Compatibility

Claude Code, Codex CLI, Gemini CLI, Cursor, Windsurf, OpenCode, Cline - any agent supporting SKILL.md.

## License

MIT
