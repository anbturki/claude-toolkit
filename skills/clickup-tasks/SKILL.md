---
name: clickup-tasks
description: Create and manage ClickUp tasks, lists, folders, and spaces via ClickUp's official MCP server (mcp.clickup.com) - workspace hierarchy best practices, and the REST-API-with-personal-token fallback for the operations the MCP server doesn't expose (space create, space/folder delete, moving a folder between spaces). Use when asked to set up ClickUp for a project, create a ClickUp space/folder/list, file a ClickUp task, or organize a ClickUp workspace's structure.
allowed-tools: Read, Bash(curl *)
---

# ClickUp tasks, lists, folders, spaces

ClickUp's hierarchy is `Workspace > Space > Folder > List > Task` (subtasks nest under a task).
Prefer the `clickup_*` MCP tools for everything they cover - they're already authenticated via the
connected OAuth session, no token to manage. This skill exists for the hierarchy-level operations
the MCP server does not expose at all, confirmed empirically, not guessed.

## Hierarchy best practice - decide this before creating anything

**Create a new Space only when a group needs its own statuses, permissions, or way of
working** - not just "a different project." ClickUp's own guidance: "the fewer Spaces, the
better" ([Hierarchy best practices](https://help.clickup.com/hc/en-us/articles/20480724378135-Hierarchy-best-practices)).
A Folder is "a grouping of related Lists (such as a client, a program, or a product line)" - that's
what a project is, in the common case of one owner working across several unrelated codebases with
the same workflow. Default shape:

```
Space: "Projects"
  Folder: <project-name>
    List: Tasks
  Folder: <another-project-name>
    List: Tasks
```

Promote a Folder to its own Space later if it genuinely needs separate permissions (a client or
collaborator who shouldn't see the other projects) - don't pre-split on day one.

## MCP tools - the default path

- `clickup_get_workspace_hierarchy` - see what exists before creating anything duplicate.
- `clickup_create_folder` (needs `space_id`), `clickup_create_list_in_folder` (needs `folder_id`).
- `clickup_create_task`, `clickup_update_task`, `clickup_filter_tasks`, `clickup_get_task`.
- `clickup_resolve_assignees` to turn an email/username/`"me"` into a user ID first.
- `clickup_get_custom_fields` before setting one on a task - custom field values are set by field
  ID, not name (same node-ID-not-name pattern as GitHub Projects v2's `item-edit`).

## The gap: what the MCP server does NOT expose

Confirmed directly, not assumed - calling `clickup_get_operators` for `models: ["space"]` returns
"No enabled operators match the request. Enabled operators: none." There is no `space.create`,
`space.delete`, `folder.delete`, `folder.move`, or `document.delete` operator on this server, even
though the underlying ClickUp REST API supports space creation
([Create Space](https://developer.clickup.com/reference/createspace)) and folder/document deletion.
This is the MCP server's own choice of exposed surface, not a ClickUp platform limitation - the
fallback below is real API, not a workaround.

**There is also no "move a Folder to a different Space" operation anywhere** (not in the MCP tools,
not in the REST API) - a Folder is created inside one Space and stays there. To relocate one:
create the new structure under the target Space, recreate its Lists/Docs/content, then delete the
old Folder (which cascades to its Lists and any Doc attached to it - confirmed: deleting a Folder
that had a Doc attached left the Doc itself unreachable afterward, no separate doc-delete needed).

## REST fallback - exact pattern

Needs a **personal API token** (`pk_...`), not the MCP server's OAuth token (that's internal to the
MCP connection and not reachable from outside it). Generate one at ClickUp -> avatar/workspace icon
-> Settings -> Apps -> API Token -> Generate. This is different from "ClickUp API Settings -> Create
an App", which registers a public OAuth integration (client ID/secret) - not what a one-off
REST call needs.

Store it as a local file or secret, never inline in a command or committed to a repo:

```
curl -s -X POST "https://api.clickup.com/api/v2/team/<workspace_id>/space" \
  -H "Authorization: $(cat ~/.keys/.CLICKUP_API_KEY)" \
  -H "Content-Type: application/json" \
  -d '{"name": "Projects", "multiple_assignees": true, "features": {}}'
```

`team_id` in this endpoint is the Workspace ID (ClickUp renamed Teams to Workspaces years ago but
kept the URL param name). Get it from `clickup_get_workspace_hierarchy`'s root `id`, or from any
ClickUp URL (`app.clickup.com/<workspace_id>/...`).

Deleting a folder (cascades to its Lists and attached Doc):

```
curl -s -X DELETE "https://api.clickup.com/api/v2/folder/<folder_id>" \
  -H "Authorization: $(cat ~/.keys/.CLICKUP_API_KEY)"
```

## GitHub integration, if this project also uses `github-issues`

ClickUp's native GitHub integration connects at the **Space level**
([GitHub integration](https://help.clickup.com/hc/en-us/articles/6305771568791-GitHub-integration)) -
one Space can link multiple repos, so a single "Projects" Space can pull in GitHub activity from
several unrelated repos/orgs without needing a Space per repo. A task/branch/commit/PR is linked by
including the ClickUp task ID in the branch name, commit message, or PR title/description; magic
words like `#taskID[closed]` in a commit message can auto-close the task. This is additive to
`github-issues`, not a replacement - keep code-adjacent issues on GitHub, let this integration
surface that activity inside ClickUp rather than duplicating tasks in both places.

## Rules

- Check `clickup_get_workspace_hierarchy` before creating a Space/Folder/List that might already
  exist under a different name.
- Never hardcode a personal API token inline in a command that gets logged/echoed - read it from a
  file or env var at call time.
- Confirm the target Space/Folder with the user before a REST-fallback delete - it is not
  reversible the way most `clickup_*` MCP writes are (ClickUp's trash/recovery windows are shorter
  and less discoverable for folders than for tasks).

## What this is not

- **Not for Docs/wiki content or formatting** - see `clickup-docs` for page structure and the
  markdown-import gotchas.
- **Not a substitute for checking `clickup_get_operators` first** - the exact set of enabled
  operators can change; re-check before assuming a gap still exists rather than reaching for the
  REST fallback out of habit.
