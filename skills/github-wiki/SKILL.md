---
name: github-wiki
description: Publish and maintain a GitHub repo's wiki - long-form docs, research notes, design write-ups - via git against the repo's separate <repo>.wiki.git (there is no REST/GraphQL API for wiki content). Use when asked to create a GitHub wiki, move docs/research to the wiki, or add/update a wiki page. Not the default destination for documentation - see section 0 before using this.
disable-model-invocation: true
allowed-tools: Read, Grep, Glob, Bash(git *), Bash(gh *)
---

# GitHub Wiki

A wiki is a **separate git repository** (`<owner>/<repo>.wiki.git`), rendered
by Gollum. There is no REST or GraphQL API for its content at all - the
only access path is cloning that repo like any other and pushing plain
markdown files. `gh` has no subcommand that touches wiki content.

## 0. When NOT to use this - check first, every time

Default to keeping documentation **in the repo itself**. A wiki is a real
downgrade (no API, no frontmatter-as-data, weak search - GitHub only
indexes wiki content in code search past 500+ stars with public editing
disabled, irrelevant for a private repo), not a strict upgrade. Keep docs
in-repo whenever any of these hold:

- A build step or generator already turns the docs into something (a
  static-site generator, a docs-serving MCP server, a custom script) - the
  wiki would be a second, divergent copy.
- Anything queries the docs programmatically - search index, CI check,
  structure validation.
- The docs carry structured frontmatter another tool depends on (`kind`,
  `area`, or similar fields used to filter/organize pages).

A wiki is a reasonable destination only for plain narrative material with
**none** of the above - research notes, meeting notes, a glossary, design
write-ups meant to be read and edited straight on github.com. Confirm this
with the user rather than assuming it.

## 1. Enable the wiki

```bash
gh repo edit <owner>/<repo> --enable-wiki
```

The `.wiki.git` repo doesn't exist until the wiki has its first page - a
bare `--enable-wiki` isn't enough to clone yet if no page has ever been
created (via the web UI or the initial push below).

## 2. Clone it

```bash
git clone https://github.com/<owner>/<repo>.wiki.git
```

If this 404s right after enabling, create the first page from the web UI
once (`https://github.com/<owner>/<repo>/wiki/_new`), then clone.

## 3. Page naming

- `Home.md` is the landing page - always create or update it as the index
  into whatever else gets added.
- Every other page is `<Page-Name>.md` (spaces become hyphens in the URL
  automatically; name the file with hyphens to match).
- Link between pages with plain relative markdown links (`[Setup](Setup)`),
  not full URLs - keeps the wiki portable if the repo is ever renamed.

## 4. Add or update a page

Plain git - no `gh` command exists for this:

```bash
cd <repo>.wiki
# write/edit the .md file
git add <Page-Name>.md
git commit -m "<what changed and why>"
git push
```

If migrating existing docs in: strip any frontmatter the source files carry
before copying content over (see the caveat below on why it won't survive
as usable data) - fold anything load-bearing (status, author, date) into
the page's own prose instead.

## 5. Frontmatter caveat

Gollum (the wiki's rendering engine) does support YAML frontmatter for
cosmetic per-page settings, but this is a **rendering-time Gollum feature,
not a GitHub API contract** - it's not exposed through any REST/GraphQL
endpoint, not indexed or queryable, and not a stable public interface.
Treat it as "the file supports it if you write it," never as a metadata
layer anything automated can read back.

## Rules

- No Claude/AI/Anthropic reference in any page content or commit message.
- Delete source files elsewhere only after confirming the wiki push
  actually succeeded (`git log` in the wiki clone, or view the page on
  github.com) - never delete-then-push.
- Never force-push into a wiki clone; treat it like any other shared repo.

## What this is not

- **Not for tasks or requests.** Those need `status`/`labels` living on a
  real issue - `github-issues`, never a wiki page pretending to be one.
- **Not a docs-generation target.** If the project's docs come from a build
  pipeline (skills, a CMS, structured source files), this skill never
  reimplements that pipeline - it's only for content with no such pipeline
  behind it in the first place.
