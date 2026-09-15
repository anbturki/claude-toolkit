---
name: archive-docs
description: Move deprecated, superseded, or finished docs out of an active project and into the vault at /Users/aliturki/devspace/workspaces/vault, organized by project/company, then by category, under an archive/ tree. Preserves structure, stamps provenance (where it came from, when, why), updates the vault index, and leaves the source repo clean. Use when a doc is done, retired, or replaced and should leave the working tree without being lost.
user-invocable: true
allowed-tools: Read, Write, Edit, Grep, Glob, Bash(ls *), Bash(find *), Bash(git *), Bash(mkdir *), Bash(mv *), Bash(date *), Bash(rg *)
argument-hint: "[doc path, docs/ folder, or 'scan' to find archive candidates]"
---

# Archive Docs to Vault

Retired docs should leave the active tree but never be lost. This skill moves finished/deprecated/superseded docs into the vault, organized so they're findable later, with a provenance trail back to where they lived.

**Target vault:** `/Users/aliturki/devspace/workspaces/vault` (a git repo)

**Scope:** `$ARGUMENTS` - a specific doc, a `docs/` folder to sweep, or `scan` to list candidates without moving anything.

## Vault structure (follow it, don't invent)

The vault already has a convention. Read it before placing anything:

```
vault/
  companies/
    <company>/            # business entity docs (e.g. confera)
  projects/
    <project>/            # active reference docs kept long-term
      <category>/         #   e.g. compliance/
      archive/            # retired docs, organized by category
        research/
        legacy/
        decisions/
        billing/
        <category>/       # add a new category only when none fits
  README.md               # the index - keep it current
```

The rule: **`projects/<project>/archive/<category>/<doc>.md`**. Company-level material goes under `companies/<company>/`. Match an existing category before creating a new one (`ls vault/projects/<project>/archive`).

> The user described this as "project/docs/archived". The vault's actual on-disk convention is `project/archive/<category>` (no `docs/` level). Follow what's on disk - consistency with the existing tree beats the loose description. If the vault layout has changed since, re-read it and adapt.

## Step 1: Identify the source project

Infer the project name from the current working directory (the repo folder name), or ask if ambiguous. This becomes `<project>` in the vault path. Decide company vs project: entity/legal/finance material -> `companies/`; product/engineering docs -> `projects/`.

## Step 2: Find archive candidates (evidence-based - don't archive live docs)

A doc is a candidate only if there's evidence it's retired. Look for:

```bash
# Explicit retirement markers in frontmatter or body
rg -n -i 'status:\s*(done|complete|archived|deprecated|superseded|retired)' <scope>
rg -n -i 'deprecated|superseded by|no longer|obsolete|moved to|replaced by' <scope>

# Completed plans / phases (a plan whose work has shipped)
rg -n -i 'status:\s*shipped|all phases? complete|✅.*done' <scope>

# Stale by age - old dated docs that nothing links to anymore
find <scope> -name '*.md' -mtime +180
```

For each candidate, confirm it's actually finished:
- A plan/PRD whose feature has shipped -> archive.
- A doc explicitly marked deprecated/superseded -> archive (note what supersedes it).
- Research/decision records for work that's done -> archive (these are the most valuable to keep).
- **Do NOT archive:** anything still referenced by active docs or code, anything with open TODOs, current runbooks, anything you're unsure about. When in doubt, leave it and list it as "candidate, needs confirmation."

In `scan` mode, stop here and report the candidate list with the evidence line for each. Move nothing until the user confirms.

## Step 3: Check inbound references before moving

Moving a doc breaks links to it. Grep the source repo for references:

```bash
rg -n '<doc-filename>' . --glob '!<scope>/**'   # who links to this doc?
```

- If active docs/code link to it: either update those links to the vault path, or leave a one-line tombstone at the old path (`# Moved to vault: projects/<project>/archive/<category>/<doc>.md`). Prefer the tombstone for docs, link-update for code.
- If nothing links to it: clean move, no tombstone needed.

## Step 4: Place each doc in the vault

For each confirmed doc:

1. Pick the category. Match an existing `archive/<category>/`; create a new one only when nothing fits, and keep the name a single lowercase domain word (`research`, `decisions`, `legacy`, `billing`, `architecture`, `features`).
2. Preserve any meaningful sub-structure (a doc that lived in `docs/billing/v2/` keeps `billing/v2/` under the category if it aids findability; don't flatten a coherent set into one dir).
3. Create the destination and move:

```bash
VAULT=/Users/aliturki/devspace/workspaces/vault
DEST="$VAULT/projects/<project>/archive/<category>"
mkdir -p "$DEST"
git -C "$(git rev-parse --show-toplevel)" mv <doc> "$DEST/" 2>/dev/null || mv <doc> "$DEST/"
```

Use `git mv` in the source repo when the doc is tracked (records the removal cleanly); plain `mv` otherwise. Cross-repo moves do not carry git history - the provenance stamp in Step 5 is what preserves the trail.

## Step 5: Stamp provenance

Prepend (or merge into existing frontmatter) an archive block at the top of each moved doc so its origin is never lost:

```markdown
---
archived: true
archived_on: <YYYY-MM-DD>        # from `date +%F`
archived_from: <original repo-relative path>
source_repo: <repo name or remote>
reason: <shipped | deprecated | superseded | finished>
superseded_by: <path or url, if applicable>
---
```

Get the date live: `date +%F`. Do not hardcode it. Keep the reason to one word plus an optional clause - no essay.

## Step 6: Update the vault index

The vault README and any per-project index must reflect what's now stored:

1. Add/extend the project's line in `vault/README.md` structure block if it's a new project or new category.
2. If the project has many archived docs, maintain `vault/projects/<project>/archive/README.md` as a short table: doc | category | archived_on | reason.
3. Keep entries one line each. The index is a finding aid, not a summary.

## Step 7: Commit the vault (and the source)

Two repos change. Commit both:

```bash
# Vault
git -C "$VAULT" add -A
git -C "$VAULT" commit -m "archive: <project> - <N> docs (<categories>)"

# Source repo (the removals / tombstones)
git -C "$(git rev-parse --show-toplevel)" add -A
git -C "$(git rev-parse --show-toplevel)" commit -m "docs: archive retired docs to vault"
```

Commit messages: factual, no AI references, no em-dash. Push only if the user asks.

## Output summary

```
## Archived to vault

Project: <project>  ->  vault/projects/<project>/archive/

Moved:
- <category>/<doc>.md   (reason: <...>, from: <old path>)
- ...

Left in place (needs confirmation):
- <doc> - <why uncertain>

References handled:
- <doc>: tombstone left at <old path>  |  links updated in <file>

Index: vault/README.md updated; archive/README.md table updated
Commits: vault <sha-or-pending>, source <sha-or-pending>
```

## Rules

- **Evidence before moving.** A doc is archived only with a retirement signal (Step 2) or explicit user say-so. Never sweep a whole `docs/` folder blindly.
- **Follow the vault's on-disk layout**, not a remembered one. `ls` it first.
- **Never lose provenance.** Every moved doc gets the archive frontmatter and an index entry.
- **Don't break links silently.** Tombstone or update every inbound reference.
- **One category per doc.** Don't duplicate a doc across categories.
- **Don't edit the doc's content** beyond the provenance frontmatter. Archiving preserves; it doesn't rewrite.
- **Two-repo commit.** The source repo's removal and the vault's addition are both recorded.

## See also

- [[write-docs]] - for producing docs; this skill retires them
- [[deep-audit]] - its Documentation dimension flags doc-vs-code drift and stale docs that are archive candidates
- [[rename-project]] - related housekeeping: keeps Claude's project state aligned when a working dir moves
