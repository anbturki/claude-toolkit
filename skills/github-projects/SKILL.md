---
name: github-projects
description: Create and manage a GitHub Project (v2) board via the gh CLI - custom fields, adding issues as items, moving an item's status, cross-repo boards. Use when asked to set up a project board, add tasks/issues to a project, track sprint/status on a board, or move an item between columns. A project is a view over issues, not a replacement for them - pair with github-issues, which owns the actual task content.
disable-model-invocation: true
allowed-tools: Read, Grep, Glob, Bash(gh *)
---

# GitHub Projects (v2)

A Project is a board layered on top of issues (and PRs, and standalone
"draft issues") - it adds custom fields (Status, Priority, Iteration) that
don't exist on the issue itself, and the same issue can sit on more than one
board. **The project never stores the task - it references an issue.**
Create the issue with `github-issues` first; use this skill to put it on a
board and drive its status field.

## 0. Auth preflight

Every write here needs the `project` OAuth scope, separate from the
default token:

```bash
gh project list --owner <owner>   # fails with "missing required scopes" if project scope is absent
```

If it fails, tell the user to run `gh auth refresh -s project` themselves -
this is an interactive OAuth step, not something a session can do
non-interactively. `--owner` accepts a user login, an org login, or the
literal `@me` - identical semantics across every subcommand below.

## 1. Create a project

```bash
gh project create --owner <owner> --title "<repo/team> backlog" --format json
```

Returns the project's `number` and its GraphQL `id` (needed later for raw
`gh api graphql` calls) - capture both.

## 2. Add fields

```bash
gh project field-create <number> --owner <owner> --name "Status" \
  --data-type SINGLE_SELECT --single-select-options "Todo,In Progress,Done"
gh project field-create <number> --owner <owner> --name "Priority" \
  --data-type SINGLE_SELECT --single-select-options "Low,Medium,High"
```

`--data-type` is a closed set: `TEXT | SINGLE_SELECT | DATE | NUMBER`. There
is **no `ITERATION` type creatable via the CLI** - a Sprint/Iteration field
needs the raw GraphQL mutation (`createProjectV2Field` with
`ProjectV2IterationFieldConfiguration`); don't attempt it through
`field-create`, tell the user to add it from the project's web UI instead if
they need sprints, or reach for `gh api graphql` directly if scripting it
matters more than the extra complexity.

## 3. Add items

```bash
gh project item-add <number> --owner <owner> --url https://github.com/<owner>/<repo>/issues/<n>
```

`--url` is mandatory (no bare issue number, no `--repo`-relative shortcut).
Prefer real issues over `item-create` (draft issues): a draft has no labels,
no milestone, no assignees, and isn't visible from the repo's own issue
list - only use `item-create` for a placeholder that will become a real
issue later.

## 4. Set a field value on an item - the ID-resolution gotcha

`item-edit` addresses everything by **node ID**, never by name - there is no
`--field-name` or human-readable option flag:

```
gh project item-edit --id <item-id> --project-id <project-id> \
  --field-id <field-id> --single-select-option-id <option-id>
```

Resolve the field ID and its options' IDs first:

```bash
gh project field-list <number> --owner <owner> --format json \
  --jq '.fields[] | {id, name, options}'
```

Then set the value:

```bash
gh project item-edit --id <item-id> --project-id <project-id> \
  --field-id <status-field-id> --single-select-option-id <done-option-id>
```

If the CLI's flag surface doesn't cover a case (e.g. resolving everything in
one round trip), the equivalent raw mutation is:

```bash
gh api graphql -f query='
  mutation {
    updateProjectV2ItemFieldValue(input: {
      projectId: "<project-id>", itemId: "<item-id>",
      fieldId: "<field-id>", value: { singleSelectOptionId: "<option-id>" }
    }) { projectV2Item { id } }
  }'
```

Only one field can be set per `item-edit` invocation - no batch update.

## 5. List / query items

```bash
gh project item-list <number> --owner <owner> --format json \
  --jq '.items[] | {title, status: .fieldValues[]}'
gh project item-list <number> --owner <owner> --query "status:\"In Progress\""
```

`--format json` plus `-q/--jq` is the scripting surface for reads across
every subcommand - use it instead of scraping table output.

## 6. Cross-repo boards

A project is account-scoped (user- or org-owned), not repo-scoped - items
from unrelated repos can sit on the same board as long as you have access
to each (`item-add --url` takes any issue/PR URL). To surface a project on a
specific repo's sidebar:

```bash
gh project link <number> --owner <owner> --repo <owner>/<repo>
gh project unlink <number> --owner <owner> --repo <owner>/<repo>
```

## Rules

- Never re-derive a field/option ID you already fetched this session -
  cache it in the conversation instead of re-running `field-list`.
- A project is disposable relative to its issues: closing or deleting a
  project loses the board view, not the underlying issues or their
  history. Deleting the *issues* is the destructive action - confirm before
  ever suggesting that.
- No Claude/AI/Anthropic reference in any project title, field name, or
  draft-issue body.

## What this is not

- **Not the system of record.** The issue is. Never describe "add it to the
  project" as if that alone created the task - `github-issues` creates the
  task, this skill only puts it on a board.
- **Not for documentation.** A project's fields hold status metadata, not
  narrative content - long-form material goes in the repo or `github-wiki`.
