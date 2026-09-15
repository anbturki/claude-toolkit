---
name: clean-code-readability
description: Make code readable. Eliminate inline complex conditions, chained ternaries, clever one-liners, and deep nesting. Use when reviewing or refactoring hard-to-read code.
user-invocable: true
allowed-tools: Read, Edit, Grep, Glob
argument-hint: "[file path or scope]"
---

# Readability

Readability is the #1 priority. A junior developer should be able to read the code and understand it.

> "Any fool can write code that a computer can understand. Good programmers write code that humans can understand." - Martin Fowler

## Scope

`$ARGUMENTS`

## Rules

### 1. No inline complex conditions

Extract boolean expressions into named variables so the intent is immediately clear.

```typescript
// BAD - reader must mentally parse the boolean logic
if (user.age >= 18 && user.isVerified && !user.isBanned && user.subscription !== "free") {
  grantAccess();
}

// GOOD - the intent is immediately clear
const isEligibleForAccess = user.age >= 18
  && user.isVerified
  && !user.isBanned
  && user.subscription !== "free";

if (isEligibleForAccess) {
  grantAccess();
}
```

**Source:** Clean Code, Ch. 3 - "Extract variables to explain complex expressions"

### 2. No chained ternaries

Replace nested ternaries with explicit mappings.

```typescript
// BAD - nested ternaries are hard to follow
const label = status === "active" ? "Running" : status === "stopped" ? "Stopped" : status === "error" ? "Failed" : "Unknown";

// GOOD - explicit mapping
const STATUS_LABELS: Record<string, string> = {
  active: "Running",
  stopped: "Stopped",
  error: "Failed",
};
const label = STATUS_LABELS[status] ?? "Unknown";
```

**Source:** Clean Code, Ch. 3 - single level of abstraction per function

### 3. No clever one-liners

Each step in a data transformation should be readable on its own line.

```typescript
// BAD - clever but unreadable
const result = data?.items?.filter(Boolean).reduce((a, b) => ({...a, [b.id]: b.values.map(v => v * multiplier).filter(v => v > threshold)}), {});

// GOOD - each step is clear
const validItems = data?.items?.filter(Boolean) ?? [];
const result: Record<string, number[]> = {};
for (const item of validItems) {
  const scaledValues = item.values.map((v) => v * multiplier);
  const aboveThreshold = scaledValues.filter((v) => v > threshold);
  result[item.id] = aboveThreshold;
}
```

**Source:** Clean Code, Ch. 3 - "Functions should do one thing"

### 4. No deep nesting - use guard clauses

Handle edge cases early and return. Keep the happy path flat.

```typescript
// BAD - 4 levels deep
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

// GOOD - guard clauses, flat structure
function processOrder(order: Order) {
  if (!order) return;
  if (order.items.length === 0) return;
  if (order.status !== "pending") return;
  if (order.total <= 0) return;

  // actual logic at the top level
}
```

**Threshold:** Maximum 3 levels of nesting.
**Source:** Refactoring - "Replace Nested Conditional with Guard Clauses"

## See also

- [[clean-code-complexity]] - cyclomatic and cognitive complexity thresholds
- [[clean-code-naming]] - naming the variables you extract
- [[clean-code-srp]] - single responsibility (often the underlying cause)
