---
name: verify-claims
description: Validate an already-built feature, implementation, workflow, or doc against primary sources - official docs, the dependency's own source/types, specs/RFCs, the CLI's --help, and current web research. Confirms two things: (1) nothing is fabricated (no invented APIs, nonexistent config fields, made-up flags, hallucinated behavior, or citations that don't exist), and (2) what was built matches current best practice, not a deprecated or made-up approach. Every claim is checked against a primary source and tagged Verified / Outdated / Unverified / Fabricated, with the source cited inline. Use when you want to be sure that what was implemented is real and correct against the docs - "validate this feature", "is this fabricated", "fact-check the implementation", "confirm this follows best practices", "verify against the docs".
user-invocable: true
allowed-tools: Read, Grep, Glob, WebSearch, WebFetch, Skill, Bash(git *), Bash(rg *), Bash(ls *), Bash(find *), Bash(cat *), Bash(npm *), Bash(bun *), Bash(bunx *), Bash(npx *), Bash(pnpm *), Bash(node *), Bash(python *), Bash(pip *)
argument-hint: "[feature name, file/dir, doc path, changeset, or 'recent' for the last change]"
model: claude-opus-4-8
context: fork
---

# Verify Claims Against Primary Sources

Take something that was already built - a feature, an implementation, a workflow, a decision doc - and prove it is **real** and **current** against authoritative sources. This is the anti-fabrication audit: it does not check whether the code is *clean* (that's `clean-code`/`deep-audit`) or whether it *runs* (that's `validate`). It checks whether every technical claim the work rests on actually holds up against the docs, the source, and the spec - and that none of it was invented.

Two questions, asked of every claim:

1. **Is it real?** Does this API / field / flag / method / option actually exist in the version we use, and behave the way the code assumes? Or was it hallucinated - plausible-sounding but absent from the docs and the types?
2. **Is it current best practice?** Is this the approach the official source recommends *today*, or a deprecated, superseded, or folklore pattern?

**Scope:** `$ARGUMENTS` - a feature, a file/directory, a doc, a changeset, or `recent` (the working-tree changes / last commit). If empty, default to `git diff` plus untracked files.

## The source hierarchy (what counts as a primary source)

Validate against the strongest source available, in this order. A claim "checked" against a weaker source when a stronger one exists is not fully verified.

1. **The dependency's own installed source / type definitions** - `node_modules/<pkg>`, `.d.ts` files, the package's published types. This is ground truth for "does this API exist in the version we actually have."
2. **Official documentation for the installed version** - the vendor's docs site, pinned to our version. Prefer the `context7` MCP (`resolve-library-id` then `query-docs`) or a project doc-search MCP when available; otherwise WebFetch the official docs URL.
3. **The CLI's own `--help` / `--version`** - for command/flag claims, run it. The binary is authoritative over any blog.
4. **Specs / RFCs / standards** - for protocol, format, or language-level claims.
5. **Reputable vendor engineering guides** - for "best practice" claims, corroborated by at least one of the above.
6. **Community sources (blogs, SO, forums)** - lowest tier. Useful for leads, never sufficient alone, and never for an existence claim.

Training-data recall is **not** a source. "I'm fairly sure this API exists" is exactly the failure mode this skill exists to catch. If you didn't read it this turn, it isn't verified.

## Step 1: Inventory the checkable claims

Read the scope and extract every assertion that can be checked against a source. Don't validate vibes - validate specifics. Pull from the code, its comments, and any docs/PRD/decision records that justify it.

Claim types to harvest:

| Type | Examples |
|---|---|
| **Existence** | An imported function, a config key, a CLI flag, an SDK method, an env var the lib reads, a decorator, an option object field |
| **Behavior** | "this call retries on 429", "this flag enables X", "passing null deletes the row", "this hook runs on mount only" |
| **Version / compat** | "needs Node >= 20", "available since v5", "the v2 API", peer-dep ranges |
| **Best practice** | "we use approach X because it's recommended", "this is the idiomatic pattern", any decision doc rationale |
| **Citation** | Any URL, doc reference, or "per the docs" claim already written into comments/docs - verify the cited source says what it's claimed to say |

Record the installed versions first - everything is judged against them:

```bash
cat package.json | rg '"(dependencies|devDependencies)"' -A 60 2>/dev/null
ls node_modules/.bin 2>/dev/null
rg -n 'version' package.json | head
python -c "import importlib.metadata as m; [print(d.metadata['Name'], d.version) for d in m.distributions()]" 2>/dev/null | rg -i '<pkg>'
```

## Step 2: Validate each claim against its source

For each claim, go to the strongest available source (Step's hierarchy) and confirm it - actually read/fetch/run, don't recall.

- **Existence claims:** grep the installed package's types/source for the identifier, or read the official API reference for the installed version.

  ```bash
  rg -n '<symbol>' node_modules/<pkg>/**/*.d.ts node_modules/<pkg>/dist 2>/dev/null
  <cli> --help 2>&1 | rg -- '--<flag>'
  ```

  If the symbol/flag/field is not in the types, not in the source, and not in the official docs for our version -> it is **Fabricated**, full stop.

- **Behavior claims:** find the documented behavior in the official docs or the source implementation. Confirm the code's assumption matches. A behavior the docs don't state and the source doesn't implement is **Unverified** at best, **Fabricated** if the code depends on it being true.

- **Version/compat claims:** compare the installed version against the doc's "available since" / requirements. A feature used below its introduction version is a real bug.

- **Best-practice claims:** confirm against *current* official guidance, and check it's not deprecated. Search the web for whether the approach is still recommended or has been superseded. Corroborate with at least two sources for any "this is the best way" claim. For deep or contested best-practice questions, compose the `deep-research` skill rather than doing a shallow search here.

- **Citations already in the work:** fetch the cited URL/doc and confirm it actually says what the comment/doc claims. A citation that doesn't support its claim is as bad as a fabrication.

## Step 3: Detect fabrication specifically

The highest-value output of this skill. Fabrications hide because they're plausible and often fail *silently* - empty results, ignored fields, no error. Hunt for:

- An API / method / field / flag that appears nowhere in the installed types, source, or official docs.
- A config key the schema/loader silently ignores (it doesn't error - it just does nothing). Cross-check against the library's config schema or CRD/struct.
- A behavior the code relies on that no source documents and the source doesn't implement.
- A "best practice" justified by a source that, when fetched, says no such thing - or a citation/URL that doesn't exist or 404s.
- Plausible-looking names that pattern-match a *different* library's API (cross-contaminated from a similar SDK).
- Numbers/limits/defaults stated as fact with no source (timeouts, rate limits, retry counts, SAN defaults, env-var names).

Per the project rule: treat any "No data" / empty payload / silently-ignored field as a *possible validation failure first*, infrastructure failure second.

## Step 4: Tag and rate every claim

Each claim gets one status and a confidence:

| Status | Meaning | Severity |
|---|---|---|
| **Verified** | Confirmed against a primary source (cited) | - |
| **Outdated** | Real, but deprecated / superseded by a current best practice | should-fix |
| **Unverified** | No primary source found either way; can't confirm or deny | flag - never report as a pass |
| **Fabricated** | Contradicted by a primary source: nonexistent identifier, ignored field, unsupported behavior, or a citation that doesn't hold | must-fix |

Confidence: **high** (read it in the source/types this turn), **medium** (official docs, version-matched), **low** (single community source). Low confidence on an existence claim is itself a finding - go find ground truth.

Severity calibration: a fabrication the code *depends on* (a called method that doesn't exist, a flag that's silently ignored) is must-fix because it's silently broken. A fabrication in a comment/doc (a wrong citation) is should-fix - it misleads readers. An outdated-but-working approach is should-fix or nice-to-have depending on the cost of the deprecation.

## Step 5: Report (evidence-cited, then offer fixes)

Report first. Do not edit until the user picks what to fix.

```
## Verify Claims - <scope>

Sources of truth used: <installed types | official docs @version | --help | spec | web>
Versions checked against: <pkg@x.y.z, ...>

### Fabricated (must-fix) - N
1. <file:line>  -  <the claim, quoted from code/doc>
   Checked against: <node_modules/<pkg>/x.d.ts | https://docs... | `cli --help`>
   Result: <symbol/field/flag absent | docs say otherwise | citation 404s>
   Impact: <silently broken | misleads | runtime error>
   Fix: <use the real API <name> | remove the field | correct the citation>

### Outdated (should-fix) - N
1. <file:line>  -  <approach used>
   Current guidance: <what the official source recommends now> (<url>)
   Fix: <migrate to ...>

### Unverified (needs a decision) - N
1. <file:line>  -  <claim>  -  no primary source found. <what I searched>. Suggest: <confirm with vendor / add a test / leave with a [verify] note>

### Verified - N
- <claim> -> confirmed in <source> (<url / file:line>)
```

End with totals and the honest bottom line: "X of Y claims verified against primary sources; Z fabricated; W unverified." If everything checks out, say so plainly and name the sources that prove it.

Then ask which to fix (must-fix only / must + should / specific items). On approval, fix in scope, and re-verify each fix against its source - a fix is not done until it too is confirmed against the doc/types.

## Rules

- **Read the source this turn or it isn't verified.** No claim passes on memory. Cite the file:line / URL / command you actually checked.
- **Existence beats eloquence.** A confident, well-written claim with no source is exactly what gets caught here. Plausibility is not evidence.
- **Version-match every check.** Validate against the *installed* version, not "latest" and not whatever the top blog post used.
- **Silent ignores are findings.** A config key that does nothing, a field the schema drops, a "No data" panel - assume fabrication/mismatch first.
- **Unverified is not verified.** Never round an unconfirmed claim up to a pass. Flag it.
- **Report before fixing.** No edits until the user picks findings.
- **Don't re-audit clean-code or security here** - this skill is solely about truth against sources. Hand those to `deep-audit` / `security-audit`.

## See also

- [[deep-research]] - the forward-looking version: research best practices *before* building; compose it here for deep best-practice questions
- [[deep-audit]] - quality/correctness/regression audit; this skill is the source-truth lane and `deep-audit`'s evidence discipline applied as a standalone pass
- [[evidence-based]] - the per-turn rule that every claim needs a primary source; this skill enforces it retroactively on shipped work
- [[write-docs]] / [[archive-docs]] - produce and retire docs; verify-claims checks that the docs' claims hold

**Source:** the project's own standing rule - "Validate Against Documentation - Never Guess": validate API/command/config names against primary documentation before trusting them; source-cite every non-trivial technical claim; treat "No data"/empty payloads as validation failures first.
