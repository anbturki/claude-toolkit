# Claude Toolkit

Production-tested skills for Claude Code, Codex CLI, and compatible AI coding agents.

Structured workflows that enforce code quality, evidence-based decisions, and consistent engineering standards across the entire development lifecycle.

## Skills

### Development Workflow

| Skill | Command | What it does |
|-------|---------|-------------|
| **research** | `/research` | Pre-coding research. Study codebase, find patterns, get approval before writing code. |
| **build** | `/build` | Implementation mode. Write code with enforced standards, security, and incremental delivery. |
| **debug** | `/debug` | Evidence-first debugging. Root cause analysis, no guessing. |
| **review** | `/review` | Code review and audit. Finds bugs, duplication, type issues, and performance problems. |
| **validate** | `/validate` | Run full validation suite — typecheck, lint, format, tests. |
| **ship** | `/ship` | Validate, commit, create branch, open PR. |

### Architecture & Quality

| Skill | Command | What it does |
|-------|---------|-------------|
| **system-design** | `/system-design` | Design module APIs, types, schemas, and interfaces before writing code. |
| **refactor** | `/refactor` | Clean code optimization. SOLID, duplication removal, complexity reduction. |
| **security-audit** | `/security-audit` | Audit for vulnerabilities, injection, auth bypasses, secret leaks. |

### Research & Documentation

| Skill | Command | What it does |
|-------|---------|-------------|
| **deep-research** | `/deep-research` | Evidence-based investigation with source validation and confidence ratings. |
| **discover** | `/discover` | Product research. Structured docs with sources, options analysis, implementation plans. |
| **write-docs** | `/write-docs` | Generate structured technical documentation — specs, guides, ADRs. |
| **explain** | `/explain` | Teaching mode. Explain code, verify comprehension, enforce documentation. |

### Specialized

| Skill | Command | What it does |
|-------|---------|-------------|
| **build-ui** | `/build-ui` | Frontend implementation. Component reuse, readability, UI pattern compliance. |
| **plan-tasks** | `/plan-tasks` | Break work into atomic sub-tasks, track progress across sessions. |

## Install

### Per-project (recommended)

```bash
# Copy the skills you need
git clone https://github.com/anbturki/claude-toolkit.git /tmp/claude-toolkit

# Pick what you need
cp -r /tmp/claude-toolkit/skills/build .claude/skills/
cp -r /tmp/claude-toolkit/skills/review .claude/skills/
cp -r /tmp/claude-toolkit/skills/ship .claude/skills/

# Or copy everything
cp -r /tmp/claude-toolkit/skills/* .claude/skills/

rm -rf /tmp/claude-toolkit
```

### Global (all projects)

```bash
git clone https://github.com/anbturki/claude-toolkit.git /tmp/claude-toolkit
cp -r /tmp/claude-toolkit/skills/* ~/.claude/skills/
rm -rf /tmp/claude-toolkit
```

## Usage

Skills are auto-discovered from `.claude/skills/` (project) or `~/.claude/skills/` (global).

```
/research add authentication to the API
/build the auth module based on the approved plan
/debug login returns 401 for valid credentials
/review src/auth/
/ship
```

## Compatibility

Works with any agent supporting the SKILL.md format:

- Claude Code
- Codex CLI
- Gemini CLI
- Cursor
- Windsurf
- OpenCode
- Cline

## License

MIT
