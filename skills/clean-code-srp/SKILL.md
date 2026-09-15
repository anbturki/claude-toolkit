---
name: clean-code-srp
description: Enforce single responsibility - one purpose per function, component, file, and architectural layer. Also bans boolean flag parameters. Use when a unit is doing too much.
user-invocable: true
allowed-tools: Read, Edit, Grep, Glob
argument-hint: "[file path or scope]"
---

# Single Responsibility

Every function, component, and file should have ONE clear purpose.

## Scope

`$ARGUMENTS`

## Rules

### 1. Functions

- **Maximum 20 lines** per function (Clean Code, Ch. 3)
- **Maximum 3 parameters** - group into an object if more (Clean Code, Ch. 3)
- If you can't name a function with a simple verb phrase, it's doing too much
- If a function has AND in its description ("validates AND saves"), split it

### 2. React Components

- **Maximum 200 lines** per component file
- Separate data logic (hooks) from presentation (JSX)
- If JSX return exceeds ~10 lines, extract sub-components
- Each component = one visual/behavioral concern

### 3. Files & Modules

- One export per file when the export is substantial (component, service, hook)
- Related small utilities can share a file
- If a file has sections with comment dividers (`// --- Section ---`), it probably needs splitting

### 4. Architecture Layers

Enforce strict direction:

```
Routes -> Services -> Repositories -> Database
  v          v             v
  HTTP    Business      Data Access
  only    logic only    only
```

- **Routes:** parse request, call service, return response. No business logic.
- **Services:** orchestrate business logic. No database queries. No HTTP concepts.
- **Repositories:** execute database queries. No business logic. No HTTP concepts.

### 5. No boolean parameters

A boolean parameter is two functions sharing a body. Split them.

```typescript
// BAD - what does `true` mean here?
createNotification(message, true, false);

// GOOD - options object with named fields
createNotification(message, { urgent: true, silent: false });

// Better - split into separate functions if the boolean changes core behavior
sendUrgentNotification(message);
sendSilentNotification(message);
```

**Source:** Clean Code, Ch. 3 (F3) - "Flag Arguments are ugly"

## See also

- [[clean-code-naming]] - if a function is hard to name, it's doing too much
- [[clean-code-complexity]] - high complexity often signals an SRP violation
- [[clean-code-dry]] - duplication often hides an extractable single responsibility

**Source:** Clean Code, Ch. 3; SOLID - Single Responsibility Principle
