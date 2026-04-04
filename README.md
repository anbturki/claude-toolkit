# Claude Toolkit

21 production-tested skills for AI coding agents.

## Install

```bash
# All skills
npx skills add anbturki/claude-toolkit

# Pick specific ones
npx skills add anbturki/claude-toolkit --skill build --skill review --skill ship

# List available
npx skills add anbturki/claude-toolkit --list
```

Or clone manually:

```bash
git clone https://github.com/anbturki/claude-toolkit.git /tmp/ct
cp -r /tmp/ct/skills/* .claude/skills/   # project
cp -r /tmp/ct/skills/* ~/.claude/skills/  # global
rm -rf /tmp/ct
```

## Skills

### Development Workflow

| Skill | What it does |
|-------|-------------|
| **research** | Study the codebase, find patterns, get approval before writing code |
| **build** | Implementation mode with enforced standards and security |
| **debug** | Evidence-first debugging and root cause analysis |
| **review** | Code audit — bugs, duplication, type safety, performance |
| **validate** | Run typecheck, lint, format, tests |
| **commit** | Stage and commit with clean messages following conventions |
| **create-pr** | Create a GitHub PR with structured summary |
| **ship** | Validate + commit + branch + PR in one flow |

### Architecture & Quality

| Skill | What it does |
|-------|-------------|
| **system-design** | Design APIs, types, schemas, and interfaces before coding |
| **refactor** | Clean code — SOLID, deduplication, complexity reduction |
| **security-audit** | Find injection, auth bypasses, secret leaks, DoS vectors |

### Planning & Research

| Skill | What it does |
|-------|-------------|
| **plan-feature** | Create PRD + phased implementation plan for a feature |
| **plan-tasks** | Break work into atomic sub-tasks, track progress |
| **deep-research** | Evidence-based investigation with source validation |
| **discover** | Product research with structured docs and options analysis |

### Documentation & Communication

| Skill | What it does |
|-------|-------------|
| **write-docs** | Generate specs, guides, architecture docs, decision records |
| **explain** | Teaching mode — explain code, verify comprehension |

### Specialized

| Skill | What it does |
|-------|-------------|
| **build-ui** | Frontend implementation with component reuse and UI pattern compliance |
| **manage-secrets** | Sync .env secrets via GitHub Variables |
| **observability** | Audit and improve logging — transports, coverage, request context, structured logs |
| **audit** | Exhaustive zero-assumption audit — reads every file, enforces CLAUDE.md, reuses skills/agents |

## Compatibility

Claude Code, Codex CLI, Gemini CLI, Cursor, Windsurf, OpenCode, Cline — any agent supporting SKILL.md.

## License

MIT
