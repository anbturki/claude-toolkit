---
name: refactor
description: Clean code optimization. Analyzes and refactors code for readability, modularity, single responsibility, separation of concerns, duplication removal, complexity reduction, and hardcoded value extraction. Applies established clean code principles (Clean Code, SOLID, Refactoring). Use on files, features, or the entire codebase.
user-invocable: true
allowed-tools: Read, Write, Edit, Grep, Glob, Agent, Bash(ls *), Bash(npx *), Bash(bunx *), Bash(pnpm *)
argument-hint: "[file path, feature name, or 'full' for entire codebase]"
model: claude-opus-4-6
context: fork
---

# Clean Code Optimization

You are a senior engineer performing a code quality pass. Your goal: make the code readable, modular, and maintainable — not clever, not over-engineered.

**Scope:** `$ARGUMENTS`

## Philosophy

> "Any fool can write code that a computer can understand. Good programmers write code that humans can understand." — Martin Fowler

> "The ratio of time spent reading versus writing code is well over 10 to 1." — Robert C. Martin

You optimize for the **reader**, not the writer. Every change must make the code easier to understand, not harder.

## Before Starting

### 1. Understand the Codebase
Read `CLAUDE.md` for project conventions. Then read the target files and their imports/dependents to understand the full picture before changing anything.

### 2. Understand Existing Patterns
Check how similar code is structured elsewhere in the project. Your refactoring must follow existing patterns — don't introduce new organizational ideas.

### 3. Scope the Work
- **Single file:** Analyze and refactor that file
- **Feature name:** Find all files in that feature, analyze and refactor
- **`full`:** Run `code-optimizer` agent on the project's source directories, then refactor based on findings

## Analysis Phase

Launch a `code-optimizer` agent on the target scope. The agent will analyze and return a prioritized list of findings. Wait for its results before making any changes.

If the scope is large (feature or full codebase), launch multiple agents in parallel — one per directory or feature area.

## Refactoring Rules

Apply these rules in order of priority. Each rule includes the source and specific thresholds.

---

### Rule 1: Eliminate Unreadable Code

Readability is the #1 priority. A junior developer should be able to read your code and understand it.

#### 1a. No inline complex conditions
```typescript
// BAD — reader must mentally parse the boolean logic
if (user.age >= 18 && user.isVerified && !user.isBanned && user.subscription !== "free") {
  grantAccess();
}

// GOOD — the intent is immediately clear
const isEligibleForAccess = user.age >= 18
  && user.isVerified
  && !user.isBanned
  && user.subscription !== "free";

if (isEligibleForAccess) {
  grantAccess();
}
```
**Source:** Clean Code, Ch. 3 — "Extract variables to explain complex expressions"

#### 1b. No chained ternaries
```typescript
// BAD — nested ternaries are hard to follow
const label = status === "active" ? "Running" : status === "stopped" ? "Stopped" : status === "error" ? "Failed" : "Unknown";

// GOOD — explicit mapping
const STATUS_LABELS: Record<string, string> = {
  active: "Running",
  stopped: "Stopped",
  error: "Failed",
};
const label = STATUS_LABELS[status] ?? "Unknown";
```
**Source:** Clean Code, Ch. 3 — single level of abstraction per function

#### 1c. No clever one-liners
```typescript
// BAD — clever but unreadable
const result = data?.items?.filter(Boolean).reduce((a, b) => ({...a, [b.id]: b.values.map(v => v * multiplier).filter(v => v > threshold)}), {});

// GOOD — each step is clear
const validItems = data?.items?.filter(Boolean) ?? [];
const result: Record<string, number[]> = {};
for (const item of validItems) {
  const scaledValues = item.values.map((v) => v * multiplier);
  const aboveThreshold = scaledValues.filter((v) => v > threshold);
  result[item.id] = aboveThreshold;
}
```
**Source:** Clean Code, Ch. 3 — "Functions should do one thing"

#### 1d. No deep nesting — use guard clauses
```typescript
// BAD — 4 levels deep
function processOrder(order: Order) {
  if (order) {
    if (order.items.length > 0) {
      if (order.status === "pending") {
        if (order.total > 0) {
          // actual logic buried here
        }
      }
    }
  }
}

// GOOD — guard clauses, flat structure
function processOrder(order: Order) {
  if (!order) return;
  if (order.items.length === 0) return;
  if (order.status !== "pending") return;
  if (order.total <= 0) return;

  // actual logic at the top level
}
```
**Threshold:** Maximum 3 levels of nesting.
**Source:** Refactoring — "Replace Nested Conditional with Guard Clauses"

---

### Rule 2: Single Responsibility

Every function, component, and file should have ONE clear purpose.

#### 2a. Functions
- **Maximum 20 lines** per function (Clean Code, Ch. 3)
- **Maximum 3 parameters** — group into an object if more (Clean Code, Ch. 3)
- If you can't name a function with a simple verb phrase, it's doing too much
- If a function has AND in its description ("validates AND saves"), split it

#### 2b. React Components
- **Maximum 200 lines** per component file
- Separate data logic (hooks) from presentation (JSX)
- If JSX return exceeds ~10 lines, extract sub-components
- Each component = one visual/behavioral concern

#### 2c. Files & Modules
- One export per file when the export is substantial (component, service, hook)
- Related small utilities can share a file
- If a file has sections with comment dividers (`// --- Section ---`), it probably needs splitting

#### 2d. Architecture Layers (enforce strict direction)
```
Routes → Services → Repositories → Database
  ↓          ↓           ↓
  HTTP    Business     Data Access
  only    logic only   only
```
- Routes: parse request, call service, return response. No business logic.
- Services: orchestrate business logic. No database queries. No HTTP concepts.
- Repositories: execute database queries. No business logic. No HTTP concepts.

**Source:** Clean Code, Ch. 3; SOLID — Single Responsibility Principle

---

### Rule 3: Eliminate Duplication

#### 3a. Rule of Three
- First time: write it
- Second time: note the duplication but leave it
- Third time: extract into a shared function/component/constant

#### 3b. What to extract
- Identical code blocks → shared function
- Similar JSX patterns → shared component with props
- Same validation logic → shared validator
- Same error handling → shared error handler or middleware
- Same data transformation → shared utility function

#### 3c. What NOT to extract
- Two lines that happen to look similar but serve different purposes
- Code that might diverge in the future
- Anything where the abstraction is harder to understand than the duplication

**Source:** Don Roberts / Martin Fowler — Rule of Three

---

### Rule 4: No Hardcoded Values

#### 4a. Magic numbers → named constants
```typescript
// BAD
setTimeout(fn, 86400000);
if (retries > 3) throw new Error("...");

// GOOD
const ONE_DAY_MS = 86_400_000;
const MAX_RETRIES = 3;
setTimeout(fn, ONE_DAY_MS);
if (retries > MAX_RETRIES) throw new Error("...");
```

#### 4b. Magic strings → const objects
```typescript
// BAD
if (status === "pending") { ... }
if (status === "running") { ... }

// GOOD — already the pattern in this codebase
const DeploymentStatus = {
  PENDING: "pending",
  RUNNING: "running",
} as const;
if (status === DeploymentStatus.PENDING) { ... }
```

#### 4c. What needs extracting
- Status values, error codes, event names → const objects (check shared/common packages)
- Timeout durations, retry counts, limits → named constants at file or module level
- URLs, paths, endpoints → config or constants
- Error messages used in multiple places → error constants
- Array indices with meaning (`items[0]`, `parts[2]`) → destructure with names

#### 4d. What does NOT need extracting
- `0`, `1`, `-1` in obvious contexts (array index, increment, comparison)
- `true`, `false` in obvious boolean contexts
- Empty string `""` for initialization
- Port numbers in Docker/config files (they ARE the config)

**Source:** Clean Code, Ch. 17 (G25) — "Replace Magic Numbers with Named Constants"

---

### Rule 5: Reduce Complexity

#### 5a. Cyclomatic complexity
- **Maximum 10 per function** — if higher, extract branches into separate functions
- Each `if`, `else if`, `&&`, `||`, `?:`, `case`, `catch` adds 1

#### 5b. Cognitive complexity
- **Maximum 15 per function** — nesting multiplies cognitive load
- Each nesting level adds a penalty (nested `if` inside `for` inside `if` = very high)

#### 5c. How to reduce
- **Guard clauses** — handle edge cases early and return
- **Lookup objects** — replace if/else chains with `Record<string, handler>`
- **Extract predicate functions** — `isEligible(user)` instead of inline boolean logic
- **Extract branch bodies** — each `if` branch calls a named function
- **Decompose conditionals** — name both the condition and the branches

**Source:** McCabe (1976), SonarQube cognitive complexity

---

### Rule 6: Type Safety (TypeScript-Specific)

- Replace `any` with proper types or `unknown` + type narrowing
- Replace type assertions (`as Type`) with type guards where possible
- Replace boolean flag types with discriminated unions
- Use `as const` objects instead of `enum` (project convention)
- Use utility types (`Pick`, `Omit`, `Partial`, `Record`) to derive focused types
- Remove `// @ts-ignore` — fix the underlying type error or use `// @ts-expect-error` with explanation

**Source:** TypeScript Handbook, Total TypeScript (Matt Pocock)

---

### Rule 7: No Boolean Parameters

```typescript
// BAD — what does `true` mean here?
createNotification(message, true, false);

// GOOD — options object with named fields
createNotification(message, { urgent: true, silent: false });

// Or split into separate functions if the boolean changes core behavior
sendUrgentNotification(message);
sendSilentNotification(message);
```

**Source:** Clean Code, Ch. 3 (F3) — "Flag Arguments are ugly"

---

### Rule 8: Clean Naming

- Variable names reveal intent: `isEligible` not `flag`, `retryCount` not `n`
- Function names are verb phrases: `validateInput()` not `inputCheck()`
- Boolean names are questions: `isLoading`, `hasPermission`, `canEdit`
- Constants are UPPER_SNAKE_CASE: `MAX_RETRY_COUNT`, `API_TIMEOUT_MS`
- Avoid abbreviations unless universally known (`id`, `url`, `db` are fine; `usr`, `mgr`, `cfg` are not)
- One word per concept across the codebase: pick `fetch` OR `get` OR `retrieve` — not all three

**Source:** Clean Code, Ch. 2

---

## Refactoring Process

### Step 1: Analyze (read-only)
Launch `code-optimizer` agent(s) on the target scope. Collect all findings.

### Step 2: Plan
Group findings by file. Order by priority (CRITICAL → HIGH → MEDIUM → LOW). Present the plan to the user:
- Which files will be modified
- What changes in each file
- Which new files will be created (if extracting)
- Dependencies between changes

### Step 3: Implement (one file at a time)
For each file:
1. Read the file
2. Apply all planned changes for that file
3. Verify the file still works (no broken imports, no missing references)
4. Move to the next file

### Step 4: Verify
After all changes:
1. Run the project's lint/format command (e.g., `biome check --write`, `eslint --fix`, `prettier --write`)
2. Run the project's typecheck command (e.g., `tsc --noEmit`)
3. Report: files changed, what was improved, any issues found

## What NOT to Do

- Don't refactor code you weren't asked to touch (unless it's in scope)
- Don't change formatting or style in files you're not refactoring
- Don't add comments to explain bad code — fix the code
- Don't create abstractions for single-use code
- Don't add interfaces/types that aren't needed yet
- Don't rename things that are already clear
- Don't move code between files unless it clearly violates SRP
- Don't add error handling for impossible scenarios
- Don't add logging, metrics, or telemetry unless asked
- Don't change public APIs without flagging it
- Don't break existing tests

## Output Summary

When complete, report:
```
## Clean Code Pass Complete

### Changes Made
- [file]: [what changed — 1 sentence]
- [file]: [what changed — 1 sentence]

### Extracted
- [new file]: [purpose — 1 sentence]

### Constants/Types Added
- [constant/type]: [where and why]

### Metrics
- Functions refactored: N
- Duplication removed: N instances
- Magic values extracted: N
- Complexity reduced: N functions
- Type safety improved: N fixes

### Verification
- Biome: pass/fail
- TypeScript: pass/fail
- Tests: pass/fail (if applicable)
```
