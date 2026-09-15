---
name: deep-audit
description: The single comprehensive, evidence-based audit. Covers clean code, SOLID + design patterns, architecture boundaries, comment hygiene, code hygiene, doc-vs-code drift, source-truth (nothing fabricated, follows best practice), functional regression, memory accuracy, security, and cross-cutting concerns. Activated when the user asks to audit, double-check, validate, review thoroughly, or "make sure" work is correct. Every claim cites a primary source (file:line, command output, vendor doc URL); training-data recall is never a source. SOLID, design-pattern, and comment-noise checks are inline (self-contained) so nothing is skipped; composes the `clean-code` orchestrator for Dimension 1, `clean-architecture` for Dimension 1c, `verify-claims` for Dimension 3b, and `security-audit` for Dimension 6. Fixes must/should-fix findings inline and verifies with the project's static checks plus live integration test.
user-invocable: true
allowed-tools: Read, Write, Edit, Grep, Glob, Skill, Bash, WebSearch, WebFetch, TaskCreate, TaskUpdate, TaskList
argument-hint: "[scope - file, feature, changeset, or 'recent']"
---

# deep-audit, Runtime contract

Audit work is a structured, evidence-based pass through every dimension of recent changes. It is NOT a vibes-based review. Every finding cites a specific file + line or command output. Every fix is verified.

The goal is to catch what slipped through normal review: stale comments after a renumbering, dead code from a refactor, doc-vs-code drift, redundant patterns, layer violations, over-engineered abstractions, regressions invisible to static checks. These accumulate quietly and bite later.

---

## Core principles (override any specific check)

**1. Evidence-based, not assumed.** Every claim, every finding, every recommendation, every "I checked X", must be backed by a primary source verified in this turn:

- File path + line range (`Read` tool, `grep -n`).
- Command output quoted (e.g. linter results, test summary).
- Live test result with timestamp (e.g. `5/5 calls passed; latency p95 = 839 ms`).
- Doc URL or RFC for any external behavioral claim.

Pattern-matching from training memory, analogy from similar projects, or "this is how it usually works" is **not evidence**. If you can't verify in this turn, mark the assertion `[unverified]` and continue, OR go verify. The marginal cost of one `Read` or `grep` is small; the cost of a false-positive finding is large (it erodes trust in the audit).

#### Source hierarchy (what counts as a primary source)

For external claims (library behavior, protocol semantics, API shapes, vendor-specific defaults), rank sources in this order. Always prefer a higher tier when available:

1. **Installed package source.** The actual code running in the project's `.venv` / `node_modules` / `vendor/` / `Cargo.lock`-resolved cargo registry. The most authoritative source for behavior because it's the exact version in use. Always check this first when claiming "library X does Y." Python example: `services/<svc>/.venv/lib/python<ver>/site-packages/<pkg>/`. JS/TS example: `node_modules/<pkg>/dist/` or `node_modules/<pkg>/src/`. Adjust the path for your runtime.
2. **Official vendor documentation** at the vendor's own domain. Examples: docs.anthropic.com, docs.aws.amazon.com, cloud.google.com/docs, platform.openai.com/docs, vendor-specific docs sites. Fetched via `WebFetch` in this turn, not recalled.
3. **Vendor's official GitHub repository.** The README, CHANGELOG, source files, or pinned issues/discussions. Acceptable when the published doc is thin or stale relative to source.
4. **Vendor engineering blogs and changelogs.** Examples: Anthropic engineering, AWS architecture blog, Google research blog, Cloudflare blog. Acceptable when the post is clearly attributed to the vendor and dated recently enough to apply.
5. **Standards bodies.** IETF RFCs, W3C specs, ECMA, ISO. Direct fetch only; never paraphrase from memory.
6. **Reputable engineering articles** by named authors at known publications. Use only as supplementary context; never as the sole basis for a behavioral claim.

What is **not** a primary source:
- Training-data recall ("I think library X handles Y like Z", "the SDK probably raises W").
- Analogy from a similar library ("Framework A works like Framework B in this respect").
- Stack Overflow answers without a vendor link backing them.
- AI-generated tutorials, content farms, SEO-optimized blogspam.
- Memory entries from prior sessions: point-in-time observations and can be stale; re-verify against tier 1 or 2 before asserting.

When the claim is about *this codebase* (not external behavior), tier 1 is always `Read` / `grep` of the file in question. No exceptions.

**2. Honest pushback over agreement.** If the user, or your own earlier self, made a claim and your evidence contradicts it, say so directly:

- Don't silently align with the prior claim to be agreeable.
- Don't soft-pedal with "you're right, but..." when you actually disagree.
- Cite the contradiction with the source; propose the corrected view; let the user decide.

Phrasing template: *"Earlier claim: X. Audit evidence: <file:line> shows Y. Recommended correction: Z. OK to proceed on Z?"*

The goal is surfacing disagreement, not winning it.

---

## When to invoke

- User asks to "audit", "double-check", "validate", "review", or "make sure all works".
- After a multi-phase build session where a lot landed quickly.
- Before a release or before merging a long-lived branch.
- Whenever you suspect drift between docs and code (e.g. after a renaming/renumbering round).

Do NOT invoke for: small one-off bug fixes, single-commit work that's still hot in context, or tasks that haven't been built yet.

---

## The 7 audit dimensions

Cover every dimension. Skipping one is how regressions hide. Dimensions 1 / 1b / 1c are split by altitude: 1 = clean-code (lines/functions), 1b = SOLID + design patterns (classes/abstractions), 1c = architecture (layer boundaries). Each catches a different class of issue, so run all three. Dimension 3b is the orthogonal source-truth pass: 1-3 ask whether the code is *clean*; 3b asks whether it is *real* and *current* against the docs (nothing fabricated).

### 1. Code shape (the load-bearing dimension)

**Invoke the `clean-code` skill** via the Skill tool, passing the same audit scope. It orchestrates the focused rule skills in priority order and returns findings:

- [[clean-code-readability]] - inline complex conditions, ternary chains, clever one-liners, deep nesting
- [[clean-code-srp]] - single responsibility for functions/components/files; boolean flag parameters
- [[clean-code-dry]] - duplication. Especially after a new abstraction lands (context manager, factory, decorator), grep for the old pattern at every pre-existing call site
- [[clean-code-no-magic-values]] - bare literals that need named constants
- [[clean-code-complexity]] - cyclomatic <= 10, cognitive <= 15, long functions (>50 lines), deep nesting (>3 levels)
- [[clean-code-naming]] - intent-revealing names; consistency (`call_id` in one place, `callId` in another)
- [[clean-code-comments]] - comment noise (restatements, banner blocks, commented-out code, stale references, multi-paragraph essays)

Merge `clean-code`'s findings into the Dimension 1 section of your audit report. Then add the audit-only checks below (which `clean-code` does NOT cover):

- **Small, organized files.** ~300 lines is a soft ceiling; >500 is suspicious; >1000 is almost always a missed split. One concept per file. Tests next to (or mirroring) the thing they test. Imports grouped (stdlib / third-party / local).
- **Layer / architecture respect.** If the project documents a layer order (e.g. `api -> service -> domain` or `controller -> usecase -> entity`), grep for reverse imports: a lower layer must not import from a higher layer. Each violation is a finding. For a full boundary audit (Dependency Rule, dependency cycles, framework/IO leakage into the core, ports & adapters), delegate to [[clean-architecture]].
- **No over-engineering.** Premature abstractions (interfaces with one implementation, factories for one constructor, generic helpers with no second consumer, configurability never used). Three similar lines beat a premature abstraction. Add the abstraction at the THIRD use site, not the first.

Findings from this dimension still cite evidence (file:line, grep output) per Core Principle 1, even when delegated.

### 1b. Design (SOLID + design patterns)

Dimension 1 (`clean-code`) covers SRP at the function/file level. This dimension audits the remaining four SOLID principles and design-pattern use - the module/abstraction level. Run these checks directly; cite file:line evidence for each finding.

#### SOLID beyond SRP

- **Open/Closed (OCP).** Look for switch/if-else chains on a type field that grow with every new variant; classes/modules edited each time a new type is added. Fix: polymorphism - lookup object, strategy, discriminated union + handler map.
- **Liskov (LSP).** Look for subclasses that throw on inherited methods (`throw new Error("not supported")`); subclasses strengthening preconditions or weakening postconditions; callers branching on `if (x instanceof Subtype)`. Fix: favor composition, or replace inheritance with discriminated union + handler.
- **Interface Segregation (ISP).** Look for interfaces/abstract classes with many methods where implementers use only a few; consumers forced to depend on methods they never call. Fix: split into smaller, role-specific interfaces.
- **Dependency Inversion (DIP).** Look for high-level modules importing concrete low-level modules directly; modules instantiating their own dependencies (`new ConcreteDb()` inside a service). Fix: inject via constructor/parameter; depend on abstractions, not concretions. (Boundary-level DIP across layers is Dimension 1c.)

#### Design-pattern misapplications

| Pattern | Smell | Fix |
|---|---|---|
| Singleton | Hidden global state, untestable, hard to mock | Module function or explicit injection |
| Factory for one constructor | `XFactory.create()` that just calls `new X()` | Delete the factory, use `new X()` |
| Observer with one listener | Pub/sub for a single direct call | Replace with the direct call |
| Strategy with one strategy | Interface + one impl, no second use site | Inline the impl |
| Manager / Helper / Util class | Vague name = SRP violation, grab-bag of unrelated methods | Rename around the responsibility, or split |

#### Missing-pattern opportunities

| Smell | Pattern |
|---|---|
| Long if/else on a type field | Strategy / handler map / discriminated union |
| Many optional constructor params | Builder |
| Creating instances by runtime type | Factory |
| Cross-cutting concern wrapping every call (log, auth, timing) | Decorator / middleware |
| One-to-many event propagation | Observer / pub-sub |
| Wrapping incompatible interfaces | Adapter |
| Same algorithm, varying steps | Template Method |

Don't introduce abstractions (interfaces, factories, strategies) without a second use site in sight. Pattern findings must solve an actual problem in the code, not score textbook points. Don't recommend a pattern the code doesn't actually call for.

### 1c. Architecture (layer boundaries)

**Invoke the `clean-architecture` skill** via the Skill tool for the full boundary audit, OR run its core checks directly if invocation is unavailable: the Dependency Rule (inner layers never import outer), no dependency cycles between modules, no framework/IO leakage into the domain/use-case layers, and ports-and-adapters (inner defines the interface, outer implements). See [[clean-architecture]] for the full checklist (component cohesion/coupling, Humble Object, stability metrics). Calibrate to project size - architecture findings drop a severity level on small or short-lived code.

### 2. Code hygiene

- **Dead code:** unused imports (Python: `ruff check --select F401`; JS/TS: `eslint --no-eslintrc --rule '{"no-unused-vars":"error"}'`), unused functions, unreachable branches, never-called helpers, commented-out code blocks. Delete, don't comment out.
- **Stale comments / docstrings:** references to phase numbers, code paths, or behavior that no longer exists. Common after renaming/renumbering. Grep the old names.
- **Comment noise / flood (run this every audit).** Default to no comment; the only comment that survives is one short line on a non-obvious WHY (hidden constraint, vendor quirk, ordering requirement, deliberate non-handling). Find every comment by regex and delete on sight:
  ```bash
  rg -n --no-heading '(^|\s)//' <scope>                          # // line comments
  rg -n --no-heading '(^|\s)#(?!!)' <scope> -g '*.py' -g '*.sh'  # # comments (skip shebangs)
  rg -n --no-heading --multiline '/\*[\s\S]*?\*/' <scope>        # block / docblock
  rg -c '(^|\s)(//|#|/\*)' <scope> | sort -t: -k2 -nr | head     # densest files first
  ```
  Delete: restatements (`// increment counter`), section banners (`// ==== helpers ====`), step narration (`# 1) validate`), commented-out code, author/date/changelog stamps, task/PR/caller references (`// added for X flow`), TODOs with no issue link, docstrings on private helpers, multi-paragraph essays. Keep (never delete): `eslint-disable`/`# noqa`/`@ts-expect-error`/`# type: ignore` directives, license/SPDX headers, comments linking an issue or vendor doc. Full checklist: [[clean-code-comments]].
- **Type-annotation gaps:** `Any` (Python) or `any` (TS) that could be tighter, missing return-type annotations on public functions, type-suppression comments without a reason. For TS specifically, delegate to [[typescript-no-any]] and [[typescript-narrowing]].
- **Anti-patterns:** silent exception swallowing (`except: pass`, empty `catch {}`), god-objects, action-at-a-distance via globals, mutable default args, magic numbers without named constants.
- **Naming consistency:** `call_id` in one place, `callId` in another. Pick one and grep for the other.

### 3. Documentation

- **Doc-vs-code drift:** docs claim X exists, code has Y. Open every doc that describes shipped behavior, cross-reference against the code.
- **Stale references after renaming/renumbering:** grep for old names. Almost always something escaped the global edit.
- **Internal contradictions:** counts, statuses, version fields disagreeing across `README.md` / `CHANGELOG.md` / `docs/*.md`.
- **TBDs that should be filled:** benchmark numbers, dates, links.
- **Inaccurate status:** "scaffolded" but actually shipped; "shipped" but tests broken.
- **Retired docs cluttering the tree:** finished plans/PRDs whose work has shipped, deprecated/superseded docs, decision records for completed work. These don't belong in the active `docs/` tree. Flag them as a finding, and for the fix invoke [[archive-docs]] to move them into the vault (`projects/<project>/archive/<category>/`) with a provenance stamp rather than deleting them. Don't archive anything still referenced or with open TODOs.

### 3b. Source-truth (nothing fabricated, follows best practice)

Dimensions 1-3 ask whether the code is *clean*. This one asks whether it is *real*: does every technical claim the implementation rests on hold up against a primary source, and is the approach current best practice rather than fabricated or deprecated? This is where hallucinated APIs, nonexistent config fields, made-up flags, and citations that don't exist get caught - they pass clean-code, pass typecheck, and fail silently in production.

**Invoke the `verify-claims` skill** via the Skill tool, passing the same audit scope. It inventories every checkable claim (imported symbols, config keys, CLI flags, SDK methods, behavior assumptions, version requirements, "best practice" rationales, and any citations already written into comments/docs), validates each against the strongest available source, and tags it Verified / Outdated / Unverified / Fabricated. Merge its findings here, mapping: Fabricated (depended-on) -> must-fix; Outdated approach / bad citation -> should-fix; Unverified -> flag for a decision (never a pass).

If invocation is unavailable, run its core checks directly - validate against the source hierarchy in Core Principle 1 (installed package source/types first, then official version-matched docs, then `--help`/spec):

- **Existence:** grep the installed package's `.d.ts`/source for every imported symbol, config key, and option field the code uses. `rg -n '<symbol>' node_modules/<pkg>/**/*.d.ts` (TS) or the `.venv`/site-packages equivalent. Not in the types, not in the source, not in the version-matched docs -> Fabricated.
- **Flags/commands:** run `<cli> --help` and confirm every flag used actually exists in the installed version.
- **Behavior:** confirm any assumed behavior ("retries on 429", "this option deletes on null") is documented or implemented in the source - not assumed.
- **Best practice:** for "we did it this way because it's recommended" claims, confirm against *current* official guidance and check it's not deprecated; for deep or contested questions, compose [[deep-research]] rather than a shallow lookup.
- **Citations in the work:** WebFetch any URL/doc already cited in comments or docs and confirm it says what it's claimed to say. A citation that 404s or doesn't support its claim is a finding.

Treat any silently-ignored config key or "No data"/empty result as a fabrication/mismatch first, infrastructure issue second. Calibrate to scope - a tiny script with no external deps has little to verify here; a feature wiring up an SDK/API has a lot.

### 4. Functional

- **All static checks pass:** the project's static-check command (`make check`, `npm test`, `cargo check`, `bun check`, `go vet ./...`, etc.) green. Don't trust earlier green if files have changed since.
- **Tests pass:** full suite, not just the file you touched. Coverage didn't drop.
- **Live integration:** run the actual happy-path. CI green is necessary but not sufficient.
- **Regression check:** if the project has a baseline benchmark, re-run and compare. Document drift even if within tolerance.
- **Edge cases the project has historically tested:** concurrency isolation, restart resilience, auth failures, etc. Re-run the ones that apply.

### 5. Memory / cross-session state

- **Memory entries accurate:** every `project_*.md`, `feedback_*.md`, `reference_*.md` file under the project's memory dir. Update or delete stale ones.
- **Memory references resolve:** if memory says "see `docs/X.md`", that file must exist.
- **MEMORY.md index matches the files in the same dir.**

### 6. Security / safety

**Invoke the `security-audit` skill** via the Skill tool, passing the same audit scope. It runs the full vulnerability checklist (auth/authz, secrets, input validation, SQL/command injection, network, DoS, frontend XSS, deps) and returns findings categorized by severity (CRITICAL/HIGH/MEDIUM/LOW).

Merge `security-audit`'s findings into the Dimension 6 section, mapping its severity to audit severity (CRITICAL/HIGH -> must-fix; MEDIUM -> should-fix; LOW -> nice-to-have). Then confirm the minimum sweep below:

- **No secrets committed:** grep for known prefixes. Common patterns: `sk-`, `gsk_`, `dg_`, `xoxb-`, `ghp_`, `AKIA[0-9A-Z]{16}`, RSA/EC private-key headers. Adjust for the providers actually in use.
- **Secret-redaction working:** if the project has a secret-redaction layer, write a test that proves it's still active.
- **No accidentally-disabled defenses:** auth, CORS, SSL verification, rate limits.

### 7. Cross-cutting

- **Git history clean:** no untracked work-in-progress files. No `.env` in the staged set. No `*.swp` / editor temp files.
- **CI green on the latest commit.**
- **No mystery TODOs/FIXMEs that should already be done.**

---

## Method

### Track the whole audit as a task list (do this first)

Before any checking, open a tracked task list with `TaskCreate` so the user can see exactly what will be audited, what's in progress, and what's done. The audit is not a black box - every dimension and every fix is a visible, status-tracked item.

Create one task per dimension up front, plus the scoping, fix, and verify steps:

```
- Bound the scope                                    [in_progress -> completed]
- Dim 1  - Code shape (clean-code)                   [pending]
- Dim 1b - Design (SOLID + patterns)                 [pending]
- Dim 1c - Architecture (clean-architecture)         [pending]
- Dim 2  - Code hygiene                              [pending]
- Dim 3  - Documentation                             [pending]
- Dim 3b - Source-truth (verify-claims)              [pending]
- Dim 4  - Functional (tests + live)                 [pending]
- Dim 5  - Memory / cross-session                    [pending]
- Dim 6  - Security (security-audit)                 [pending]
- Dim 7  - Cross-cutting                             [pending]
- Fix must-fix + should-fix                          [pending]
- Verify nothing regressed                           [pending]
```

Then drive the list as you work:

- Mark a dimension `in_progress` with `TaskUpdate` when you start it, `completed` when its evidence is collected and findings recorded.
- **Every finding becomes its own task.** When a dimension surfaces a must-fix/should-fix, add a task for it via `TaskCreate` (e.g. "Fix [must-fix] services/api/x.ts:42 - SQL injection"), so the fix phase has a concrete, status-tracked checklist. nice-to-have findings can be one rolled-up task.
- Skip a dimension only with reason; if you skip one (e.g. no React in scope), mark its task `completed` with a note saying it was N/A, not silently dropped.
- The task list is the audit's source of truth for "what will happen / what happened." `TaskList` at the end must show every dimension done and every must/should-fix finding either fixed or explicitly deferred by the user.

### Step 1: Bound the scope

Ask the user, or infer from the conversation, what counts as "recent work":

- A specific phase or feature (most common).
- A commit range (`<sha>..HEAD`).
- A file set.
- "Everything since this branch diverged from main."

If the scope is ambiguous, ask. **Do not audit everything in the repo by default**: that's noise.

### Step 2: Collect evidence per dimension

Use the right tool for each check. Every check produces a citation that lands in the finding (`file:line`, command output snippet, etc.):

| Check | Tool |
|---|---|
| File size | language-agnostic: `wc -l <src dirs>/**` and sort. Examples: `find src -name '*.py' \| xargs wc -l \| sort -nr \| head` (Python); `find src -name '*.ts' -o -name '*.tsx' \| xargs wc -l \| sort -nr \| head` (TS) |
| Function complexity | Python: `radon cc src/ -s -a`. JS/TS: `npx complexity-report` or `eslint --rule complexity`. Go: `gocyclo`. If not installed: visual inspection |
| Duplication | `grep -rn '<old pattern>'` after a new abstraction lands; visual diff between similar files |
| Dead imports | The project's linter for unused-imports rule (Python: `ruff check --select F401 .`; JS/TS: `eslint`; Go: `goimports -l`) |
| Layer violations | `grep -rn 'from <higher-layer>' src/<lower-layer>/` (or import-equivalent for the language) |
| Stale references | `grep -rn '<old-name>' .` (after renumbering, target the old prefix) |
| Doc-vs-code drift | Read each doc; cross-reference to the source it describes |
| Test green | full test runner; not just the changed file's tests |
| Live integration | the project's existing live-test script |
| Regression | the project's bench harness, or run the live test 5x and compare |
| Memory accuracy | Read `~/.claude/projects/<project>/memory/MEMORY.md` and each linked file |
| Secrets | `grep -rn -E '<provider prefixes you use>'`; rotate the regex to your stack |

If a dimension has no automated check, do it manually by Reading the relevant files. **Do not skip a dimension because the automated check doesn't exist**: that's how the most expensive bugs hide.

### Step 3: Produce the findings list

Format each finding with **mandatory** evidence + recommendation:

```
N. [SEVERITY] <file:line>
   Evidence: <quoted line / command output / grep result>  (required)
   Finding: <one-line description>
   Recommendation: <what to do>
```

Without the Evidence line, it is not a finding; it is a guess. Every entry must show its work.

Severity:

- **must-fix:** bug, regression, security issue, broken behavior. Fix before continuing.
- **should-fix:** code-quality issue, drift, stale doc. Fix during this audit cycle.
- **nice-to-have:** cosmetic, refactor opportunity. Note but defer unless the user asks.

If there are zero findings in a dimension, say so explicitly **and cite what you ran to confirm**. Example: "Code shape: zero findings. Verified by `wc -l src/**/*.py | sort -nr | head` (max 134 lines, well under 500); `grep -rn 'from <project>.api' src/<project>/domain/` (no matches)." Silence is not the same as "checked, all clean."

### Step 4: Fix the must-fix and should-fix

Don't accumulate findings into a separate ticket queue. Audit + fix in the same cycle.

For each fix:

1. State what you're changing and why (one sentence).
2. Make the change.
3. Mark that finding's task `completed` via `TaskUpdate` (or `in_progress` while mid-fix).
4. After all fixes for a dimension are in, re-run that dimension's check.

A finding the user chooses to defer stays as a task with a note ("deferred by user") rather than being deleted - the list must reflect reality.

### Step 5: Verify nothing regressed

After fixes:

1. The project's static-check command (e.g. `make check`, `npm test`, `cargo check`): green.
2. Full test suite: all tests pass; coverage held.
3. Live integration test: still green.
4. If the project has a benchmark, re-run; compare to baseline.

If anything regressed, that's a higher-priority finding than the one you were fixing. Stop, fix the regression, re-run.

### Step 6: Commit + push

One commit per audit cycle (or one per dimension if the changes are big and unrelated). Commit message format:

```
audit: <summary of what was found and fixed>

- <file>:<line> <one-line per finding fixed>
- ...

Verified: <static-check command> green; <N>/<N> tests pass; <live integration result, e.g. "live gate p95 839ms within 5% of baseline" or "5/5 happy-path scenarios pass">.
```

Push. Verify CI green.

---

## Quality bar

- **Every finding cites evidence:** `file:line`, command output snippet, or grep result. "Looks duplicated" without a line reference is not a finding.
- **Every claim is fact-checked in this turn:** no recall, no analogy, no "I think this works like X". If you cannot verify, mark `[unverified]` explicitly. See core principle 1.
- **Push back when evidence contradicts a prior claim:** the user's, your own, or a memory entry's. Cite the contradiction; let the user decide. See core principle 2.
- **Severity is honest.** If you tag everything must-fix, the tag is meaningless. If you tag everything nice-to-have, you're avoiding work.
- **Zero findings is allowed** if you actually checked every dimension and they were clean, and you cite what you ran to confirm.
- **No false positives.** A finding that the user reads and dismisses is worse than a missing one; it erodes trust in the audit.
- **Verify after fix.** Static green is necessary; live green is sufficient. If the project's live test takes >5 minutes, set up a Monitor and continue work; don't skip it.

---

## Common pitfalls

1. **"It looks fine":** not an audit step. Run the check.
2. **Asserting from training memory:** "I think library X handles Y like Z"; verify by reading the source or fetching docs in this turn. Specifically: claims about library behavior require a tier-1 (installed source) or tier-2 (official vendor docs) check; claims paraphrased from "what the SDK probably does" are assertions, not findings. See the Source hierarchy in Core Principle 1.
3. **Agreeing with a stale claim:** "the user said this works, so it works"; re-verify, the claim may be obsolete.
4. **Auditing only what you wrote:** old code drifts too. If you renamed phase IDs, audit ALL phase docs, not just the new ones.
5. **Trusting CI from before the fix:** re-run after every change.
6. **Reporting a finding without fixing it:** at must-fix / should-fix severity, you should fix it now.
7. **Spawning sub-agents that have no project context:** they re-read everything from cold and miss what's obvious to you. Spawn them only for genuinely independent searches.
8. **Skipping the live test:** static checks miss serialization bugs, ordering bugs, deadlock bugs.
9. **Editing memory without updating MEMORY.md:** both files must move together.
10. **Saying "no findings" when you only checked one dimension:** qualify per dimension.
11. **Calling a long file "fine because it works":** file size is a finding even when the code is correct; it indicates a missed split that will hurt the next reader.

---

## Example invocations

**User says:** "audit what we just shipped"

**You do:**
1. Identify scope: "recent work since the last commit on main" or "since this branch diverged".
2. Run all 7 dimensions; cite evidence per dimension (the command you ran or the file you read).
3. Report findings as a numbered list with severity, evidence, finding, recommendation.
4. Fix must/should-fix inline.
5. Re-verify with the project's static-check command + live test.
6. Commit + push as `audit: <summary>`.
7. Report back: number of findings by severity, what was fixed, what (if anything) was deferred.

**User says:** "make sure all works 100%"

**Same as above.** Treat as audit. The "100%" is rhetorical, not literal: focus on the dimensions, not on perfection.

---

## What this skill is NOT

- A code-review skill for unsubmitted work-in-progress (use a normal review).
- A test-writing skill (use a dedicated testing skill or the project's test-suite phase).
- A refactoring skill (audit finds smells; refactoring is a separate, scoped task).
- A documentation-writing skill (audit finds drift; writing fresh docs is separate).

When in doubt about scope, ask the user before expanding.
