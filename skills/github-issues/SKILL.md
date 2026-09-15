---
name: github-issues
description: Create, triage, and manage GitHub issues via the gh CLI - labels, milestones, assignees, search/list, close with reason. Use when asked to file a bug/task/feature as a GitHub issue, triage a backlog, or manage labels/milestones. Pair with github-projects to put the issue on a board, or github-wiki for narrative docs (an issue is never the right place for those).
disable-model-invocation: true
allowed-tools: Read, Grep, Glob, Bash(gh *)
---

# GitHub Issues

An issue is the unit of work - title, body, labels, assignees, comments, one
open/closed state. It lives in exactly one repo. This skill covers the
issue's own lifecycle; putting it on a board is `github-projects`, moving
narrative docs to a wiki is `github-wiki`.

## Create an issue

```bash
gh issue create --repo <owner>/<repo> \
  --title "<one descriptive sentence>" \
  --label <name> \
  --assignee <login|@me> \
  --milestone "<existing milestone title>" \
  --body "$(cat <<'EOF'
## Problem

<what's wrong or missing, with evidence>

## Proposed fix / shape

<what should happen instead>
EOF
)"
```

- `--milestone` takes the milestone's **name**, not a number - it must already
  exist (see Milestones below, there is no `gh milestone create`).
- `--project "<title>"` adds it to a board in the same call, but needs the
  `project` OAuth scope (`gh auth refresh -s project` if `gh issue create`
  warns about it) and only adds the item - it does not set any custom field
  (status, priority) on it. For anything beyond a bare add, use
  `github-projects`'s `item-edit` flow after creating the issue.
- Draft before creating: show the title and body to the user and wait for
  confirmation before running the command, unless they already gave you the
  exact text - this writes visible state into a repo other people watch.

## List and search

```bash
gh issue list --repo <owner>/<repo> --state open --label bug --assignee @me
gh issue list --repo <owner>/<repo> --search "is:open label:bug sort:created-asc"
gh issue list --repo <owner>/<repo> --json number,title,state,labels,projectItems --jq '.[]'
```

`--search` accepts full GitHub search syntax. There is no `--project` filter
flag on `gh issue list` - to filter by board membership use `gh project
item-list --query` (see `github-projects`) or pull `--json projectItems` and
filter with `jq`.

## Triage / edit

```bash
gh issue edit <number> --repo <owner>/<repo> \
  --add-label <name> --remove-label <name> \
  --add-assignee <login> \
  --milestone "<title>"
```

Accepts multiple issue numbers in one call: `gh issue edit 12 14 19 --add-label triaged`.

## Close / reopen / comment

```bash
gh issue close <number> --repo <owner>/<repo> --reason completed --comment "<why>"
gh issue close <number> --repo <owner>/<repo> --reason "not planned" --comment "<why>"
gh issue reopen <number> --repo <owner>/<repo> --comment "<why>"
gh issue comment <number> --repo <owner>/<repo> --body "<update>"
```

`--reason` is a closed enum: `completed | not planned | duplicate` (use
`--duplicate-of <number|url>` for the last one). Only close what's actually
done - state clearly in the comment whether the underlying work is verified,
not just claimed.

## Labels

```bash
gh label create <name> --color <hex6> --description "<what it means>" --repo <owner>/<repo>
gh label list --repo <owner>/<repo> --json name,color,description
gh label edit <name> --color <hex6> --repo <owner>/<repo>
gh label clone <owner>/<repo> --source-repo <owner>/<other-repo>   # copy a label set across repos
```

No scope requirement beyond the default token. Check `gh label list` before
creating a new label - reuse an existing close match rather than adding a
near-duplicate.

## Milestones

There is no `gh milestone` command. Reference an existing one by name in
`issue create/edit --milestone`; to create one, fall back to the REST API:

```bash
gh api repos/<owner>/<repo>/milestones -f title="<name>" -f state=open
gh api repos/<owner>/<repo>/milestones --jq '.[].title'
```

## Rules

- No Claude/AI/Anthropic reference anywhere in a title, body, or comment.
- No emoji.
- Never close or edit an issue you didn't create or weren't asked to
  triage, without confirming first.
- State evidence, not assumption, in the body - a failure mode with proof
  beats an opinion.

## What this is not

- **Not a board.** Status, priority, and sprint/iteration tracking are
  `github-projects`'s job - an issue alone only has open/closed plus labels.
- **Not for narrative docs.** Design write-ups, research notes,
  investigation logs belong in the repo's own docs or `github-wiki`, never
  as an issue body padded out to hold long-form content.
