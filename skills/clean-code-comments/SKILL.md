---
name: clean-code-comments
description: Sweep a codebase (or a single file) for comment noise and strip it down. Finds every comment by regex, deletes restatements, banner blocks, commented-out code, author/date stamps, and stale references, and shortens multi-line essays to one concise WHY. Every surviving comment must explain a non-obvious WHY. Use when AI-generated or legacy code is flooded with comments and you want a focused cleanup pass.
user-invocable: true
allowed-tools: Read, Edit, Grep, Glob, Bash(grep *), Bash(rg *), Bash(find *), Bash(wc *)
argument-hint: "[file path, directory, or 'full' for the whole repo]"
---

# Comment Hygiene Sweep

Most comments rot. Code is the source of truth; comments drift the moment they stop being maintained. AI assistants in particular over-narrate - a comment on every line, section banners, restatements of the next statement. Default to no comment. Keep one only when removing it would confuse a future reader.

## Scope

`$ARGUMENTS` - a file, a directory, or `full` for the whole repository. If empty, default to the files changed in the working tree (`git diff --name-only` + untracked), since that's usually the AI-generated code you want to clean.

## The rule

**One short line per non-obvious WHY.** That is the entire rule. Everything below is how to find violations and apply it.

## Run the sweep

### Step 1: Find every comment by regex

Comments are language-specific. Run the patterns that match the files in scope. `rg` (ripgrep) preferred; fall back to `grep -rn` if `rg` is unavailable.

```bash
# Line comments: // (C/JS/TS/Go/Rust/Java/Swift) and # (Python/Ruby/Shell/YAML/TOML)
rg -n --no-heading '(^|\s)//' <scope>                      # // line comments
rg -n --no-heading '(^|\s)#(?!!)' <scope> -g '*.py' -g '*.rb' -g '*.sh'   # # comments (skip shebangs)

# Block comments and docblocks: /* ... */, /** ... */, JSX {/* ... */}
rg -n --no-heading --multiline '/\*[\s\S]*?\*/' <scope>

# Python/Ruby docstrings and block strings
rg -n --no-heading --multiline '"""[\s\S]*?"""' <scope> -g '*.py'

# Count comment density per file to find the worst offenders first
rg -c '(^|\s)(//|#|/\*)' <scope> | sort -t: -k2 -nr | head -20
```

Start with the densest files - those are where the noise concentrates.

### Step 2: Classify each hit

For every comment, decide delete / shorten / keep using the tables below. When in doubt, delete: git history preserves anything you remove.

### Step 3: Apply edits, then verify

After stripping comments, run the project's lint/format and typecheck (`biome check`, `eslint`, `tsc --noEmit`, `ruff`, etc.) to confirm you didn't remove a load-bearing directive (see "What NOT to do"). Comments carry no runtime behavior, so tests should be unaffected - if a test breaks, you deleted a directive comment; restore it.

A surviving comment must explain at least one of:
- A hidden constraint (upstream bug, vendor quirk, ordering requirement).
- A non-obvious invariant (why this branch can never fire; why this default is flipped).
- A workaround tied to a specific issue (link the issue).
- Behavior that would surprise the next reader (an empirically-tuned constant, a deliberate non-handling).

If the comment is longer than the code it annotates, cut it. Long-form belongs in `docs/`, not in source files.

## Delete on sight

| Smell | What it looks like | Why it's noise |
|---|---|---|
| **Restatement** | `// increment counter` above `counter++` | Identifier already says it |
| **Banner / section block** | `// ===== USER METHODS =====` | Use a class, file, or namespace split instead |
| **Section header in a small function** | `// Step 1: validate`, `// Step 2: persist` | If the function needs internal headers, it's too long - extract |
| **Commented-out code** | `// const old = ...` left next to the new code | Git remembers. Delete it |
| **TODO with no owner or issue** | `// TODO: fix later` | Either file the issue and link, or delete |
| **Author / date / changelog stamps** | `// Author: X, 2021-04-12, modified by Y` | Git blame already has this |
| **Reference to current task / PR / caller** | `// added for the X flow`, `// used by Y` | Rots the moment the caller changes; belongs in the PR description |
| **Multi-paragraph essays inside code** | 20-line comment explaining the next 20 lines | Extract a named function or put the explanation in docs |
| **Re-stating the type** | `// returns a User` above `function getUser(): User` | Type signature already says it |
| **JSDoc / docstring on private helpers** | `/** Format the date. */ formatDate(...)` | Internal helpers don't need ceremony; the name carries the contract |

## Keep when the WHY is non-obvious

```ts
// Stripe webhooks deliver events out of order; sort by created_at before applying.
events.sort((a, b) => a.created_at - b.created_at);

// 4096 is the Bedrock per-message ceiling, not a project choice - see provider docs.
const MAX_TOKENS = 4096;

// Empty catch is intentional: the lookup is best-effort; failure means cache miss, not error.
try { return cache.get(key); } catch { return null; }
```

Each one survives because the reader cannot infer it from the code alone.

## Docstrings and JSDoc

- **Public API surface (exported functions, library entry points):** a short docstring is appropriate. One line of purpose + one line per non-obvious param/return contract. Skip parameter lines that are obvious from the type.
- **Internal / private helpers:** no docstring by default. The name is the contract.
- **Generated docs (TypeDoc, Sphinx, etc.):** if the project publishes docs from comments, treat exported symbols as the surface area; everything else stays bare.

## Stale-reference scan

When auditing existing comments, grep for stale anchors:

```bash
# References to phase IDs, old names, deleted files
rg -n 'TODO|FIXME|XXX|HACK' <scope>
rg -n '(deprecated|legacy|old|todo:)' <scope> -i
rg -n '<old-symbol-name>' <scope>   # after a rename
```

For each hit:
1. Is the referenced thing still real? If no -> delete the comment.
2. Is the referenced thing still relevant? If no -> delete.
3. Is the WHY still non-obvious? If no -> delete.

## File-flood symptoms

If a file has comments on every block, it is almost always one of:
- Functions doing too much - the writer added section headers to navigate their own code. Fix: extract per [[clean-code-srp]] and [[clean-code-size]].
- Names that don't reveal intent - the writer added narration to compensate. Fix: rename per [[clean-code-naming]].
- Code that is hard to read - the writer explained the trick instead of removing the trick. Fix: simplify per [[clean-code-readability]].

The comment fix is the symptom. The structural fix is the cause.

## What NOT to do

- Don't delete comments that link to issues, vendor docs, or specific tickets - those carry context git blame can't reconstruct.
- Don't strip license headers, SPDX identifiers, or copyright notices.
- Don't remove `// eslint-disable-next-line <rule>` or `# noqa: <code>` directives - those are load-bearing.
- Don't delete `# type: ignore[<code>]` or `// @ts-expect-error` with their justification on the same line.
- Don't rewrite a working comment just to make it shorter - shorten only if you genuinely improve it.

## Output (when used as a check)

```
Comment findings:
- <file:line> [delete] restatement of the next line
- <file:line> [delete] commented-out code (3 lines)
- <file:line> [delete] stale reference to <removed symbol>
- <file:line> [shorten] multi-paragraph essay -> one-line WHY
- <file:line> [keep] explains Stripe ordering quirk (non-obvious)
```

## See also

- [[clean-code-srp]] - section-header comments often mean the function is doing too much
- [[clean-code-size]] - file-flood comments often mean the file is too big
- [[clean-code-readability]] - comments explaining clever code are a symptom; fix the code instead
- [[clean-code-naming]] - comments restating intent often mean the name was wrong
- [[clean-code]] - the orchestrator that invokes this skill among others

**Source:** Robert C. Martin, *Clean Code*, Ch. 4 (Comments).
