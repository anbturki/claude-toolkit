# Claude Toolkit

15 production-tested skills for AI coding agents.

## Install

```bash
# Install all skills
npx skills add anbturki/claude-toolkit

# Install specific skills
npx skills add anbturki/claude-toolkit --skill build --skill review --skill ship

# List available skills
npx skills add anbturki/claude-toolkit --list
```

Or manually:

```bash
git clone https://github.com/anbturki/claude-toolkit.git /tmp/ct
cp -r /tmp/ct/skills/* .claude/skills/   # project-level
cp -r /tmp/ct/skills/* ~/.claude/skills/  # global
rm -rf /tmp/ct
```

## Skills

| Skill | Description |
|-------|-------------|
| **research** | Pre-coding research. Study the codebase, find patterns, get approval before writing. |
| **build** | Implementation mode. Code standards, security, incremental delivery. |
| **debug** | Evidence-first debugging. Root cause analysis, no guessing. |
| **review** | Code review and audit. Bugs, duplication, type safety, performance. |
| **validate** | Run full checks — typecheck, lint, format, tests. |
| **ship** | Validate, commit, branch, open PR. |
| **system-design** | Design APIs, types, schemas, and interfaces before coding. |
| **refactor** | Clean code. SOLID, deduplication, complexity reduction. |
| **security-audit** | Audit for injection, auth bypasses, secret leaks, DoS. |
| **deep-research** | Evidence-based investigation with source validation. |
| **discover** | Product research. Structured docs with sources and options analysis. |
| **write-docs** | Generate specs, guides, architecture docs, decision records. |
| **explain** | Teaching mode. Explain code, verify comprehension. |
| **build-ui** | Frontend implementation. Component reuse, UI pattern compliance. |
| **plan-tasks** | Break work into atomic sub-tasks, track progress. |

## Compatibility

Works with any agent supporting the SKILL.md format — Claude Code, Codex CLI, Gemini CLI, Cursor, Windsurf, OpenCode, Cline.

## License

MIT
