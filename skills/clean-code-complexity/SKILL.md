---
name: clean-code-complexity
description: Reduce cyclomatic and cognitive complexity below safe thresholds. Use guard clauses, lookup objects, and extracted predicates. Use when a function has too many branches.
user-invocable: true
allowed-tools: Read, Edit, Grep, Glob
argument-hint: "[file path or scope]"
---

# Reduce Complexity

## Scope

`$ARGUMENTS`

## Thresholds

### Cyclomatic complexity

- **Maximum 10 per function** - if higher, extract branches into separate functions
- Each `if`, `else if`, `&&`, `||`, `?:`, `case`, `catch` adds 1

**Source:** McCabe (1976)

### Cognitive complexity

- **Maximum 15 per function** - nesting multiplies cognitive load
- Each nesting level adds a penalty (nested `if` inside `for` inside `if` = very high)

**Source:** SonarQube cognitive complexity

## How to reduce

### 1. Guard clauses

Handle edge cases early and return. Flattens the happy path.

```typescript
function process(order: Order) {
  if (!order) return;
  if (order.items.length === 0) return;
  if (order.status !== "pending") return;

  // happy path at top level
}
```

### 2. Lookup objects

Replace if/else chains with a `Record<key, handler>`.

```typescript
// BAD
if (type === "email") sendEmail(payload);
else if (type === "sms") sendSms(payload);
else if (type === "push") sendPush(payload);

// GOOD
const SENDERS = {
  email: sendEmail,
  sms: sendSms,
  push: sendPush,
} as const;

SENDERS[type](payload);
```

### 3. Extract predicate functions

Replace inline boolean logic with named functions.

```typescript
// BAD
if (user.age >= 18 && user.isVerified && !user.isBanned) { ... }

// GOOD
if (isEligibleUser(user)) { ... }

function isEligibleUser(user: User) {
  return user.age >= 18 && user.isVerified && !user.isBanned;
}
```

### 4. Extract branch bodies

Each `if` branch calls a named function.

### 5. Decompose conditionals

Name both the condition and the branches.

## See also

- [[clean-code-readability]] - guard clauses and deep-nesting rules
- [[clean-code-srp]] - high complexity is often an SRP violation
- [[clean-code-naming]] - extracted predicates and branches need good names
