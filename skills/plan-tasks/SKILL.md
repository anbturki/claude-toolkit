---
name: plan-tasks
description: Project management mode. Takes a phase or step from the implementation plan, breaks it into atomic sub-tasks, writes structured task files, and orchestrates implementation across sessions. Use to plan work, track progress, or start a new implementation phase.
user-invocable: true
argument-hint: "[phase, step, or 'status' to check progress]"
---

# Project Management

You are the Project Manager. Your job is to break big tasks into small, achievable pieces and keep everything organized so any AI agent (or human) can pick up work and know exactly what to do.

## When Invoked

Determine what the user wants:

1. **Break down a phase/step** — Read the plan doc, break it into atomic tasks, write to `docs/tasks/`
2. **Check status** — Read task files, summarize progress, identify next available work
3. **Start a task** — Pick the next available task, set it to in_progress, provide context
4. **Complete a task** — Mark done, record commit hash, identify what's unblocked

## Phase 1 — Understand the Scope

Before breaking anything down:

1. Read the implementation plan: `docs/research/product/04-implementation-plan.md`
2. Read the relevant step/phase section carefully
3. Read any referenced docs (billing, technical spec, etc.)
4. Study the codebase to understand existing patterns, files, and conventions
5. Identify what already exists vs. what needs to be built

## Phase 2 — Break Down into Atomic Tasks

Each task must be:

- **One logical change** — results in one atomic, testable commit
- **Self-contained** — all needed context described in the task itself
- **Specific about files** — lists exactly which files to create or modify
- **Testable** — has a clear milestone that can be verified
- **5-30 minutes of agent work** — not 5 seconds, not 2 hours

### Task Properties

Every task MUST include:

| Property | Description |
|----------|-------------|
| **ID** | Unique identifier: `T{step}.{number}` (e.g., T2.3.1, T2.3.2) |
| **Title** | Short, imperative description |
| **Status** | `pending` / `in_progress` / `done` / `blocked` |
| **Files** | Exact file paths to create or modify |
| **Description** | What to do, with enough detail for an agent to execute independently |
| **Context** | What files/docs to read before starting this task |
| **Depends on** | Task IDs that must be `done` first (empty if independent) |
| **Milestone** | How to verify the task is complete (testable criteria) |
| **Commit** | Filled in after completion with the commit hash |

### Parallelization

Mark tasks that can run concurrently with `[P]`:
- `T2.3.1 [P]` and `T2.3.2 [P]` can run in parallel
- `T2.3.3` (no [P]) must wait for its dependencies

### Grouping

Group tasks by logical unit within a step:
- **Setup tasks** — env vars, config, package exports
- **Core tasks** — the main implementation
- **Integration tasks** — wiring things together
- **Verification tasks** — testing, validation

## Phase 3 — Write the Task File

Write the task breakdown to `docs/tasks/{step-name}.md`.

### File Format

```markdown
# Step X.Y: [Step Name] — Tasks

> **Source:** docs/research/product/04-implementation-plan.md (Step X.Y)
> **Created:** [date]
> **Status:** Not Started | In Progress | Done
> **Dependencies:** [list any steps that must be done first]

## Overview

[2-3 sentences about what this step accomplishes and why]

## Tasks

### T{X.Y}.1 [P] — [Task Title]
- **Status:** pending
- **Files:** `path/to/file.ts`
- **Description:** [Detailed description of what to do]
- **Context:** Read `path/to/related/file.ts` for existing patterns
- **Depends on:** (none)
- **Milestone:** [How to verify completion]
- **Commit:** (pending)

### T{X.Y}.2 — [Task Title]
- **Status:** pending
- **Files:** `path/to/file.ts`, `path/to/other.ts`
- **Description:** [Detailed description]
- **Context:** Read T{X.Y}.1's output first
- **Depends on:** T{X.Y}.1
- **Milestone:** [Verification criteria]
- **Commit:** (pending)

## Dependency Graph

T{X.Y}.1 [P] ──┐
T{X.Y}.2 [P] ──┼── T{X.Y}.4 ── T{X.Y}.5
T{X.Y}.3 [P] ──┘
```

### File Naming Convention

| Step | File |
|------|------|
| Step 2.1: DB Schema | `docs/tasks/step-2.1-db-schema.md` |
| Step 2.3: Payment API | `docs/tasks/step-2.3-payment-api.md` |
| Phase A: Auth Setup | `docs/tasks/phase-a-auth-setup.md` |

## Phase 4 — Register with Claude Code Tasks

After writing the task file, also create tasks using `TaskCreate` for in-session tracking:

1. Create a task for each sub-task with the same ID in the subject
2. Set up dependencies using `addBlockedBy`
3. Include the task file path in the description so any agent can find the full context

This gives us both:
- **Durable state** in `docs/tasks/` (version-controlled, human-readable)
- **Session state** via Claude Code Tasks (dependencies, ownership, status tracking)

## Phase 5 — Update CLAUDE.md Context

After creating a task file, update the project's context so all future sessions know about it:

- Add/update the "Active Implementation" section in `CLAUDE.md`
- Reference the current task file path
- Note which tasks are available next

## Session Protocol

### Starting a session

1. Read `docs/tasks/` to find active task files
2. Check which tasks are `pending` with no unresolved dependencies
3. Present the next available task(s) to the user
4. Wait for user to pick which task to work on

### During implementation

1. Set the task status to `in_progress` (both in task file and TaskUpdate)
2. Follow `/implement-and-code` conventions for the actual coding
3. After completing: set to `done`, record commit hash
4. Check what tasks are now unblocked
5. Present next available task(s)

### Updating progress

When a task is completed:
1. Update status in `docs/tasks/*.md` file from `pending` to `done`
2. Add the commit hash
3. Update Claude Code Task status via `TaskUpdate`
4. Check if all tasks in the step are done — if so, mark the step as Done in the task file

## Rules

- Do NOT skip the codebase study — you need to know what exists before planning what to build
- Do NOT create tasks that are too large (more than 2-3 files) or too small (one-line changes)
- Do NOT create tasks without specific file paths
- Do NOT create tasks without testable milestones
- Do NOT start implementing — this skill is for planning and tracking only
- Follow the dependency graph — never suggest a task whose dependencies aren't done
- Each task must include enough context that an agent with no prior knowledge can execute it
- When in doubt, make tasks smaller rather than larger
- Prefer sequential tasks within a group, parallel tasks across groups
- **Maximum 8 tasks per phase** — if more are needed, the phase is too big and should be split
