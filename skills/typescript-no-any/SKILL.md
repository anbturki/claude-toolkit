---
name: typescript-no-any
description: Ban `any`, untyped `object`, `as Type` assertion bypasses, and `@ts-ignore`. Use when auditing or refactoring TypeScript code for type safety.
user-invocable: true
allowed-tools: Read, Edit, Grep, Glob, Bash(npx tsc *), Bash(bunx tsc *), Bash(pnpm tsc *)
argument-hint: "[file path or scope]"
---

# No `any`

`any` opts out of TypeScript. It silently propagates through your code and turns type errors into runtime errors. Find every use, find the proper type instead.

## Scope

`$ARGUMENTS`

## What to ban

### Critical (must fix)

- Explicit `any` (`let x: any`, `function f(x: any)`)
- `any[]` and `Array<any>`
- `as any` and `as unknown as Foo` (the double-cast bypass)
- Untyped `object` (define the actual shape)
- `@ts-ignore` (use `@ts-expect-error` with a 1-line reason if truly unavoidable)
- Missing function return types on exported APIs
- Untyped function parameters

### Important (should fix)

- Implicit `any` from missing types (enable `noImplicitAny`)
- Overly broad types that could be narrowed
- Missing interface/type exports

## What to use instead

| Instead of | Use |
|---|---|
| `any` (unknown input) | `unknown` + type narrowing (see [[typescript-narrowing]]) |
| `any` (escape hatch in tests) | A real shape, even a minimal one |
| `as Foo` (assertion) | A type guard function that returns `value is Foo` |
| `object` | A real interface or `Record<string, unknown>` if the keys are dynamic |
| `Function` | A specific signature like `(x: number) => string` |
| `any[]` | The actual element type, e.g. `User[]` or `unknown[]` |

## Derive, don't duplicate

Types should be inferred from their source of truth instead of hand-written:

```typescript
// From ORM schemas (Drizzle, Prisma, etc.)
type User = typeof users.$inferSelect;        // Drizzle
type User = Prisma.UserGetPayload<{}>;        // Prisma

// From validation schemas (Zod, etc.)
type CreateInput = z.infer<typeof createSchema>;

// From API clients
type Response = Awaited<ReturnType<typeof apiCall>>;
```

Never manually duplicate a type that can be inferred - duplicates drift.

## Audit process

1. Run the project's typecheck (`tsc --noEmit`, `bun run typecheck`, etc.)
2. `grep -rn ": any\|as any\|@ts-ignore\|<any>" src/`
3. Fix each occurrence (replace with proper type, narrow `unknown`, or write a type guard)
4. Re-run typecheck

## See also

- [[typescript-narrowing]] - the `unknown` + type guard replacement pattern
- [[typescript-types]] - utility types (Pick/Omit/Partial/Record) for shaping replacements
- [[clean-code-srp]] - functions that take `any` are often doing too much
