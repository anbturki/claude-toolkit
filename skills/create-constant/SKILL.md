---
name: create-constant
description: Define constants, enums, and shared values — replace magic strings, numbers, and hardcoded values with typed, reusable definitions. Use when adding status types, error codes, config values, or any repeated literal.
argument-hint: [constant-name or domain]
allowed-tools: Read, Write, Edit, Grep, Glob, Bash(ls *)
---

# Create Constant / Enum

Define typed constants for `$ARGUMENTS`. Eliminate magic strings and numbers.

## Step 1: Learn Project Patterns

1. **Read CLAUDE.md** for constant/enum conventions
2. **Find existing constants**:
   ```
   Glob("**/constants/**/*.ts")
   Glob("**/enums/**/*.ts")
   Glob("**/types/**/*.ts")
   Glob("**/config/**/*.ts")
   Grep("as const")
   Grep("enum ")
   ```
3. **Read 2-3 existing constant files** — learn:
   - Style: `as const` objects vs `enum` vs union types
   - Naming: `UPPER_SNAKE` vs `PascalCase` vs `camelCase`
   - File location and grouping strategy
   - Whether types are co-exported with values

## Step 2: Choose the Right Pattern

### `as const` Object (preferred in most TS projects)

Best for: status values, roles, event names, error codes — anything that needs both a runtime value and a type.

```typescript
export const ${Name} = {
  ACTIVE: "active",
  INACTIVE: "inactive",
  PENDING: "pending",
} as const;

// Derive type from values — never define manually
export type ${Name} = (typeof ${Name})[keyof typeof ${Name}];
// → "active" | "inactive" | "pending"
```

### TypeScript `enum`

Use only if the project already uses `enum` style. Avoid mixing styles.

```typescript
export enum ${Name} {
  Active = "active",
  Inactive = "inactive",
  Pending = "pending",
}
```

### Union Type (no runtime value needed)

Best for: types used only in type checking, not at runtime.

```typescript
export type ${Name} = "active" | "inactive" | "pending";
```

### Numeric Constants

```typescript
export const ${NAME} = {
  MAX_RETRIES: 3,
  TIMEOUT_MS: 30_000,
  PAGE_SIZE: 50,
  MAX_FILE_SIZE_MB: 10,
  MAX_FILE_SIZE_BYTES: 10 * 1024 * 1024,
} as const;
```

### Map / Lookup Object

Best for: labels, display values, color mappings — anything mapping a key to a human-readable or UI value.

```typescript
export const ${Name}Labels: Record<${Name}, string> = {
  [${Name}.ACTIVE]: "Active",
  [${Name}.INACTIVE]: "Inactive",
  [${Name}.PENDING]: "Pending",
};

export const ${Name}Colors: Record<${Name}, string> = {
  [${Name}.ACTIVE]: "green",
  [${Name}.INACTIVE]: "gray",
  [${Name}.PENDING]: "yellow",
};
```

### Error Codes

```typescript
export const ${Domain}ErrorCodes = {
  NOT_FOUND: "${domain}NotFound",
  DUPLICATE: "duplicate${Domain}",
  INVALID: "invalid${Domain}",
  LIMIT_EXCEEDED: "${domain}LimitExceeded",
} as const;

export type ${Domain}ErrorCode = (typeof ${Domain}ErrorCodes)[keyof typeof ${Domain}ErrorCodes];
```

### Event Names

```typescript
export const ${Domain}Events = {
  CREATED: "${domain}.created",
  UPDATED: "${domain}.updated",
  DELETED: "${domain}.deleted",
} as const;

export type ${Domain}Event = (typeof ${Domain}Events)[keyof typeof ${Domain}Events];
```

### Config / Limits

```typescript
export const ${Domain}Config = {
  MAX_ITEMS: 100,
  DEFAULT_PAGE_SIZE: 20,
  NAME_MIN_LENGTH: 1,
  NAME_MAX_LENGTH: 255,
  ALLOWED_MIME_TYPES: ["image/png", "image/jpeg", "image/webp"] as const,
} as const;
```

## Step 3: Audit for Magic Values

After defining constants, find and replace hardcoded values:

```
Grep("\"active\"")        # Find magic strings
Grep("\"inactive\"")
Grep("=== 3")             # Find magic numbers
Grep("setTimeout.*[0-9]{4,}")  # Find hardcoded timeouts
Grep("\\.length > [0-9]") # Find hardcoded limits
```

Replace each with the constant:

```typescript
// BEFORE — magic string
if (user.status === "active") { ... }

// AFTER — typed constant
if (user.status === UserStatus.ACTIVE) { ... }

// BEFORE — magic number
if (retries > 3) throw new Error("...");

// AFTER — named constant
if (retries > RetryConfig.MAX_RETRIES) throw new Error("...");

// BEFORE — hardcoded timeout
setTimeout(fn, 86400000);

// AFTER — readable constant
const ONE_DAY_MS = 86_400_000;
setTimeout(fn, ONE_DAY_MS);
```

## Step 4: Wire to Validation Schemas

Constants should feed into Zod/validation schemas:

```typescript
import { ${Name} } from "../constants";

// Use constant values in schema
export const create${Entity}Schema = z.object({
  status: z.enum([${Name}.ACTIVE, ${Name}.INACTIVE, ${Name}.PENDING]),
  name: z.string().min(Config.NAME_MIN_LENGTH).max(Config.NAME_MAX_LENGTH),
});
```

## What Needs a Constant

| Type | Example | Constant? |
|------|---------|-----------|
| Status values | `"active"`, `"pending"` | Yes — `as const` object |
| Error codes | `"userNotFound"` | Yes — error codes object |
| Event names | `"user.created"` | Yes — events object |
| Timeouts/durations | `30000`, `86400000` | Yes — named constant |
| Retry counts/limits | `3`, `100`, `50` | Yes — config object |
| URLs/paths | `"/api/users"` | Yes — if used in 2+ places |
| HTTP methods | `"GET"`, `"POST"` | No — obvious in context |
| `0`, `1`, `-1` | array index, increment | No — universally understood |
| `true`, `false` | boolean flags | No — obvious |
| Empty string `""` | initialization | No — obvious |

## Rules

1. **Match project style** — `as const` vs `enum` vs union, follow what exists
2. **Co-export types** — always export the derived type alongside the value
3. **Group by domain** — `UserStatus`, `OrderStatus`, not a single `Statuses` mega-file
4. **Use the constant everywhere** — after defining, replace ALL occurrences of the literal
5. **Feed into schemas** — validation schemas should reference constants, not hardcode values
6. **Underscore separators for large numbers** — `86_400_000` not `86400000`
7. **No premature extraction** — only extract if used in 2+ places or if the meaning isn't obvious
