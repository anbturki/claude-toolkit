---
name: full-audit
description: The parallel form of deep-audit. Fans out one isolated sub-agent per quality skill - security, architecture (layers + boundaries), every clean-code rule, TypeScript safety, React - so no skill is skipped by in-context composition. Each sub-agent takes its time and cites evidence. Report-only by default, then offers a consolidated fix phase. Use when you want the audit's per-skill isolation run concurrently; for the single-agent comprehensive version use `deep-audit` (which also covers SOLID, design patterns, docs, memory, and regression). SOLID + design-pattern coverage here comes from the clean-architecture and clean-code-srp lanes.
user-invocable: true
allowed-tools: Read, Glob, Grep, Bash, Agent, Skill
argument-hint: "[file path, feature name, changeset, or 'full' for the repo]"
model: claude-opus-4-8
context: fork
---

# Full Audit

Run the entire quality toolkit against a scope by spawning **one isolated sub-agent per skill**. This is not a vibes review and it is not a single agent trying to remember to invoke ten skills - it is a fleet of workers, each running exactly one skill with full attention.

**Scope:** `$ARGUMENTS`

## Why fan out to sub-agents

When one agent is asked to invoke many skills in sequence, it routinely skips some - the later skills get token-starved or quietly dropped. Giving each skill its **own** `audit-runner` sub-agent guarantees:

- Every skill actually runs (the worker's only job is to invoke that one skill).
- Each gets fresh, unrushed context - no competition for attention.
- Findings are isolated and auditable before anything is merged.

Do not collapse this into a single agent invoking skills inline. The fan-out is the point.

## Step 1: Bound the scope

Infer from `$ARGUMENTS` or the conversation. If ambiguous, ask. Default to the working-tree changes (`git diff --name-only` plus untracked) when no scope is given - that's usually the code that needs the sweep. Do not audit the whole repo unless asked; that's noise.

## Step 2: Detect which skills apply

Don't spawn React workers for a backend, or TypeScript workers for a Python repo. Detect first:

```bash
# Languages / frameworks present in scope
rg -l --no-heading '\.(ts|tsx)$' <scope> 2>/dev/null | head -1   # TypeScript present?
rg -l --no-heading 'from .react.|import React' <scope> 2>/dev/null | head -1   # React present?
find <scope> -name '*.py' | head -1   # Python present?
```

## Step 3: Build the worker list

Always-on lanes (any language):

| Lane | Skill |
|---|---|
| Security | `security-audit` |
| Architecture - layers + boundaries | `clean-architecture` |
| Readability | `clean-code-readability` |
| Single responsibility | `clean-code-srp` |
| Size limits | `clean-code-size` |
| Duplication | `clean-code-dry` |
| Magic values | `clean-code-no-magic-values` |
| Complexity | `clean-code-complexity` |
| Naming | `clean-code-naming` |
| Comment noise | `clean-code-comments` |

TypeScript lanes (only if `.ts`/`.tsx` in scope):

| Lane | Skill |
|---|---|
| No `any` | `typescript-no-any` |
| Narrowing | `typescript-narrowing` |
| Type shape | `typescript-types` |

React lanes (only if React in scope):

| Lane | Skill |
|---|---|
| Components | `react-components` |
| JSX | `react-jsx` |
| Performance | `react-performance` |
| Accessibility | `react-accessibility` |

Skip a lane only when the detection in Step 2 shows it doesn't apply, and **say so in the report** ("React lanes skipped: no React files in scope"). Silent skips read as coverage that didn't happen.

> Note: this deliberately runs the clean-code *rules* directly rather than the `clean-code` orchestrator. Running each rule as a separate worker is what guarantees it fires. SOLID + design-pattern coverage comes from `clean-architecture` (boundary-level DIP) and `clean-code-srp` (responsibility); the full SOLID/pattern checklist lives inline in `deep-audit` for the single-agent path. `deep-audit` is not spawned here - `full-audit` is its fan-out form.

## Step 4: Spawn one sub-agent per skill (in parallel)

For each skill in the worker list, launch an `audit-runner` sub-agent via the Agent tool. Send them in a single batch so they run concurrently.

Each sub-agent's prompt must contain:

```
SKILL: <the one skill>
SCOPE: <the bounded scope from Step 1>
MODE: report-only

Invoke the <skill> skill via the Skill tool against SCOPE. Take your time -
do not rush. Every finding cites file:line or quoted command output; mark
anything you cannot verify [unverified]. Do not edit any file. Return findings
in the required format.
```

Use `subagent_type: audit-runner`. Wait for all workers to return before consolidating.

If the scope is large, the workers will each scan their slice - that's expected and fine. Let them take the time.

## Step 5: Consolidate

Merge all worker reports into one findings list, grouped by lane, then sorted by severity within each lane. Deduplicate: the same `file:line` flagged by two lanes (e.g. a long function caught by both `clean-code-size` and `clean-code-complexity`) collapses to one entry that notes both lanes.

```
## Full Audit - <scope>

Workers run: <N> (<list>)
Lanes skipped: <lane: reason>

### must-fix (<count>)
1. <lane(s)> | <file:line>
   Evidence: <quoted>
   Finding: <one line>
   Fix: <what to do>

### should-fix (<count>)
...

### nice-to-have (<count>)
...

### Clean lanes
- <skill>: zero findings (verified by <command>)
```

Report total counts by severity. Be honest about severity - if everything is must-fix, the tag is meaningless.

## Step 6: Offer the fix phase

Do NOT fix yet. Present the consolidated findings and ask the user which to fix:

- "Fix all must-fix and should-fix?"
- "Just must-fix?"
- "Pick specific findings?"

When approved, fix in scope-disjoint batches to avoid conflicts (one lane's files at a time), then verify with the project's typecheck + lint + tests, and confirm green. Re-run any worker whose lane you touched if you want a clean re-check.

## Rules

- **Fan out - never inline.** One `audit-runner` per skill. This is the whole design.
- **Detect before spawning.** No React workers on a backend.
- **Report-only first.** No edits until the user picks findings (Step 6).
- **Evidence or it's not a finding.** Workers enforce this; the consolidation must not add un-cited claims.
- **Name what you skipped.** Skipped lanes and capped scopes go in the report, not silence.
- **Don't rush.** A complete slow audit beats a fast partial one.

## See also

- [[deep-audit]] - the single comprehensive audit (also covers SOLID, design patterns, docs, memory, regression); `full-audit` is its parallel fan-out form
- [[clean-architecture]] - layer boundaries; spawned here as one worker
- [[security-audit]] - the vulnerability checklist; spawned here as one worker
- [[clean-code]] - the in-context orchestrator; `full-audit` runs its rule skills as separate workers instead
