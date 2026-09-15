---
name: typescript-types
description: Shape types correctly. Discriminated unions over boolean flags, `as const` over `enum`, utility types (Pick/Omit/Partial/Record) to derive focused types. Use when designing or refactoring type definitions.
user-invocable: true
allowed-tools: Read, Edit, Grep, Glob
argument-hint: "[file path or scope]"
---

# Type Shape

Pick the right shape for the data, not the first one that compiles.

## Scope

`$ARGUMENTS`

## Rules

### 1. Discriminated unions over boolean flags

If a flag changes the shape of an object, model the variants directly.

```typescript
// BAD - what does `data` look like when isError is true?
interface Response {
  isError: boolean;
  data?: User;
  error?: string;
}

// GOOD - discriminated union, exhaustive
type Response =
  | { status: "success"; data: User }
  | { status: "error"; error: string };
```

The narrowed shape removes whole classes of impossible states (e.g., `isError: true` with `data` set).

### 2. `as const` objects over `enum`

`enum` has runtime cost, awkward bidirectional mapping, and doesn't play well with `import type`. `as const` objects are simpler and tree-shake.

```typescript
// AVOID
enum Status {
  Pending = "pending",
  Running = "running",
}

// PREFER
const Status = {
  PENDING: "pending",
  RUNNING: "running",
} as const;

type Status = typeof Status[keyof typeof Status];
```

### 3. Utility types for derived shapes

Don't hand-write subset types - derive them.

| Helper | Use case |
|---|---|
| `Pick<T, K>` | Take only specific fields |
| `Omit<T, K>` | Drop specific fields |
| `Partial<T>` | All fields optional (update payloads) |
| `Required<T>` | All fields required |
| `Readonly<T>` | Frozen shape |
| `Record<K, V>` | Map types with known keys |
| `Awaited<T>` | Unwrap a Promise |
| `ReturnType<F>` | Function's return type |
| `Parameters<F>` | Function's parameter tuple |

```typescript
type User = { id: string; name: string; email: string; password: string };

type PublicUser = Omit<User, "password">;
type UserUpdate = Partial<Pick<User, "name" | "email">>;
type UsersById = Record<string, User>;
```

### 4. Prefer `interface` for object shapes, `type` for unions/intersections/aliases

Both work, but the split keeps intent visible. Use whichever the codebase already uses.

### 5. Avoid `Function` and `Object`

Use specific signatures (`(x: number) => string`) and specific shapes (`Record<string, unknown>` if truly dynamic).

## See also

- [[typescript-no-any]] - what to use instead of `any` (often a proper type from this skill)
- [[typescript-narrowing]] - how to consume discriminated unions safely
- [[clean-code-no-magic-values]] - `as const` objects double as the magic-string fix
