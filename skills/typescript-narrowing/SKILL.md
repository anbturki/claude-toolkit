---
name: typescript-narrowing
description: Replace `any` and assertions with `unknown` + type narrowing. Write type guards instead of `as Type`. Use when handling external input, API responses, or unknown data.
user-invocable: true
allowed-tools: Read, Edit, Grep, Glob
argument-hint: "[file path or scope]"
---

# Type Narrowing

When you don't know the shape of a value, use `unknown` + narrowing - not `any` and not `as Foo`.

## Scope

`$ARGUMENTS`

## Rules

### 1. `unknown` over `any` for external input

`unknown` forces you to narrow before use; `any` lets bugs through.

```typescript
// BAD
function parse(input: any) {
  return input.user.name; // crashes at runtime if shape is wrong
}

// GOOD
function parse(input: unknown) {
  if (!isUserPayload(input)) throw new Error("Invalid payload");
  return input.user.name; // narrowed - safe
}
```

### 2. Type guards over assertions

Replace `as Foo` with a function that returns `value is Foo`.

```typescript
// BAD - assertion lies to the compiler
const user = data as User;

// GOOD - type guard verifies at runtime AND narrows the type
function isUser(value: unknown): value is User {
  return (
    typeof value === "object" &&
    value !== null &&
    "id" in value &&
    "email" in value &&
    typeof (value as { id: unknown }).id === "string"
  );
}

if (isUser(data)) {
  // data is User here, verified
}
```

### 3. Use schema validators for complex shapes

Hand-written type guards get unwieldy for nested objects. Reach for a runtime validator (Zod, Valibot, ArkType) and infer the type from the schema.

```typescript
const userSchema = z.object({
  id: z.string(),
  email: z.string().email(),
});

type User = z.infer<typeof userSchema>;

const result = userSchema.safeParse(data);
if (result.success) {
  // result.data is User
}
```

### 4. Narrow with the typeof / in / discriminant ladder

```typescript
function format(value: string | number | { tag: "date"; iso: string }) {
  if (typeof value === "string") return value.toUpperCase();
  if (typeof value === "number") return value.toFixed(2);
  return value.iso; // narrowed to the object variant
}
```

### 5. Use the `satisfies` operator to keep narrowed literal types

```typescript
// Without satisfies - widens to Record<string, string>, loses literal keys
const config: Record<string, string> = { home: "/", about: "/about" };
config.home; // string (not "/")

// With satisfies - keeps literal keys, still type-checks against the constraint
const config = {
  home: "/",
  about: "/about",
} satisfies Record<string, string>;
config.home; // "/" (preserved)
```

## See also

- [[typescript-no-any]] - the rule this skill implements the alternative for
- [[typescript-types]] - discriminated unions are the easiest things to narrow
- [[clean-code-readability]] - guard clauses pair naturally with type narrowing
