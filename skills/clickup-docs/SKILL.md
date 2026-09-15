---
name: clickup-docs
description: Create and maintain ClickUp Docs as a project wiki - nested pages, a Home landing page that links to every section, and the exact markdown-formatting rules that keep pages from rendering as mangled walls of text (no hard-wrapped list items, no fenced-code language tags, no toggle/collapsible blocks - none of these survive the Docs API's markdown import). Use when asked to create a ClickUp wiki, write ClickUp documentation, add a ClickUp doc page, or move research/design notes into ClickUp.
allowed-tools: Read
---

# ClickUp Docs as a wiki

ClickUp Docs is ClickUp's wiki feature - there is no separately-branded "Wiki" product, Docs *is*
it: nested pages, a Home page, full-text search across the workspace (Docs Hub), and
Doc-to-task Relationships. Use the `clickup_*` MCP Doc tools
(`clickup_create_document`, `clickup_create_document_page`, `clickup_update_document_page`,
`clickup_list_document_pages`, `clickup_get_document_pages`).

## Structure convention

One Doc per project, attached to that project's Folder, with a Home page that links to every
section - the pattern ClickUp's own help docs recommend for a scalable wiki
([Build a scalable knowledge base](https://clickup.com/p/how-to-create-knowledge-base-for-team-wiki-structure-guidelines)):

```
Doc: "<project> Wiki"
  Home            - index, one paragraph per section, links out
    Research      - investigation notes, each page states its own status/date
    Decisions     - the why behind non-obvious calls
    Guides        - onboarding/how-to content
```

Nest topic pages under `Research`/`Decisions` rather than making everything a Home sibling - a
reader drilling from Home into a section should see only that section's pages, not the whole
wiki flattened into one sidebar level.

## The formatting rules - confirmed empirically, not a style preference

ClickUp's Docs API has real, documented markdown-import limitations
([Docs import/export limitations](https://developer.clickup.com/docs/docsimportexportlimitations)),
plus one gotcha that is really an authoring habit, not a ClickUp limitation. All three below were
proven directly - a page was written, read back, and the damage inspected before writing this rule.

**1. Never hard-wrap a list item or paragraph across multiple lines.** Writing a long bullet as
several physical lines (relying on the reader's editor or a renderer to soft-wrap it back into one
line) breaks on import - ClickUp's markdown parser treats each internal line break inside a list
item as the start of a **new paragraph**, shattering one bullet into a bullet fragment plus loose
paragraphs underneath it. This is the single biggest cause of a ClickUp doc coming out looking
"mixed together" or badly structured. Fix: every list item and every paragraph is **one continuous
string with no embedded newline**, no matter how long the sentence gets. Let the ClickUp editor's
own soft-wrap handle line breaks visually - never insert a real one mid-item.

**2. Never put a language tag on a fenced code block.** ` ```ts `, ` ```bash `, ` ```toml ` and
similar all get corrupted on import - confirmed directly: a ` ```ts ` block came back as ` ```dpr `,
a ` ```toml ` block came back as ` ```plain `. The code *content* survives; the language label does
not, and what's displayed instead is a wrong, confusing label - worse than no label. Use a bare
` ``` ` fence with no tag. Syntax highlighting is lost either way (ClickUp's own docs confirm code
formatting is not preserved on import) - a bare fence at least doesn't display false information.

**3. Toggle/collapsible sections are not importable at all** - not a formatting mistake, a real
API gap ("Toggle List" is explicitly listed as unsupported). The closest available substitute:
ClickUp lets a reader manually collapse **any heading** in the live editor UI, hiding everything
until the next heading of the same or greater level. So structure content with clear, real
headings (`##`, `###`) rather than reaching for a toggle - the reader gets equivalent
collapse-on-demand behavior once viewing the page, even though it can't be pre-set collapsed via
the API.

**Tables are supported and safe to use**, but keep cells short and simple (no embedded lists or
multi-line content inside a cell) - ClickUp's own docs say tables "lose formatting" on import,
which in practice means the table structure survives but any fancy per-cell formatting won't.

## Visibility - a plan gate, not a bug

`clickup_create_document` with `visibility: "PRIVATE"` failed with `not_found_or_authorized` on a
Free-plan workspace; `visibility: "PUBLIC"` succeeded immediately with identical parameters
otherwise. This reproduced on both a Space-level and Folder-level parent, ruling out a
parent-type problem - it's a plan-tier gate on private Docs. Default to `PUBLIC` and confirm with
the user before treating a Doc as access-restricted, since it may need a plan upgrade to actually
enforce that.

## Page creation mechanics

`clickup_create_document` needs a `parent: {id, type}` where type is `"4"` (Space), `"5"` (Folder),
`"6"` (List), `"7"` (Everything), or `"12"` (Workspace). `create_page: true` gives a default blank
page with `name: null` - rename it to `Home` via `clickup_update_document_page` rather than
creating a second page and leaving an orphaned blank one. Nest subsequent pages with
`parent_page_id` on `clickup_create_document_page`.

`clickup_update_document_page`'s `content_edit_mode` defaults to `replace` (overwrites the whole
page) - use `append`/`prepend` for incremental edits that must preserve existing content exactly,
without needing to read the page first.

## Rules

- Migrating a file from disk: preserve the substance, not the original file's exact prose - apply
  the single-line-item rule while porting, don't carry over the source file's own line wrapping.
- State a migrated page's original status plainly, and correct it in the page itself if it's
  known to be stale (e.g. a "not started" research note whose feature has since shipped) - don't
  silently carry forward a false status just because it's what the source file said.
- No Claude/AI/Anthropic reference in page content.

## What this is not

- **Not for task/project/space management** - see `clickup-tasks` for hierarchy operations and
  the REST-fallback pattern for what the MCP server doesn't expose.
- **Not the default destination for documentation.** Generated docs (a build pipeline, a docs
  site, structured frontmatter another tool reads) stay in the source repo - this skill is for
  narrative content with no such pipeline behind it, the same rule `github-wiki` applies for the
  GitHub-hosted equivalent.
