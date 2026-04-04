---
name: zod-schema
description: Scaffold Zod validation schemas with inferred TypeScript types. Use when adding input validation for API endpoints or forms.
argument-hint: [schema-name]
allowed-tools: Read, Write, Edit, Grep, Glob, Bash(ls *)
---

# Create Zod Schema

Scaffold validation schemas for `$ARGUMENTS`.

## Step 1: Learn Project Patterns

1. **Read CLAUDE.md** for validation conventions
2. **Find existing schemas**:
   ```
   Glob("**/schemas/**/*.ts")
   Glob("**/validators/**/*.ts")
   ```
3. **Read 2-3 existing schemas** — learn:
   - Import style (`import { z } from "zod"` vs `import * as z`)
   - How common schemas are reused (pagination, etc.)
   - Type export patterns
   - Where schemas live in the project

## Step 2: Scaffold Schemas

**File location:** Match existing schema file locations.

```typescript
import { z } from "zod";

// Reuse common schemas if they exist in the project
// import { paginationSchema } from "./common";

// List/filter schema (for GET query params)
export const list${Name}sSchema = z.object({
  search: z.string().optional(),
  limit: z.coerce.number().min(1).max(100).optional().default(50),
  offset: z.coerce.number().min(0).optional().default(0),
  // Add filter fields
});

// Create schema (for POST body)
export const create${Name}Schema = z.object({
  name: z.string().min(1, "Name is required").max(255),
  // Add required fields with validation
});

// Update schema (for PUT body)
export const update${Name}Schema = z.object({
  name: z.string().min(1).max(255).optional(),
  // Optional fields for partial updates
});

// Export inferred types — NEVER define these manually
export type List${Name}sInput = z.infer<typeof list${Name}sSchema>;
export type Create${Name}Input = z.infer<typeof create${Name}Schema>;
export type Update${Name}Input = z.infer<typeof update${Name}Schema>;
```

## Common Patterns

### Query Parameters (coerce from strings)
```typescript
z.coerce.number()           // "50" → 50
z.coerce.boolean()          // "true" → true
z.string().optional()       // already a string from URL
```

### Enums
```typescript
export const statusSchema = z.enum(["active", "inactive", "pending"]);
type Status = z.infer<typeof statusSchema>;  // "active" | "inactive" | "pending"
```

### Arrays
```typescript
z.array(z.string()).min(1).max(10)          // 1-10 strings
z.array(z.string().uuid()).optional()       // optional UUID array
```

### Refinements
```typescript
z.string().regex(/^#[0-9A-Fa-f]{6}$/, "Invalid hex color")
z.string().email("Invalid email")
z.string().url("Invalid URL")
z.string().min(8, "Password must be at least 8 characters")
```

### Nested Objects
```typescript
export const create${Name}Schema = z.object({
  name: z.string().min(1),
  address: z.object({
    street: z.string(),
    city: z.string(),
    zip: z.string(),
  }).optional(),
  tags: z.array(z.string()).default([]),
});
```

### Extending Schemas
```typescript
// Update = Create with all fields optional
export const update${Name}Schema = create${Name}Schema.partial();

// List = pagination + filters
export const list${Name}sSchema = paginationSchema.extend({
  status: statusSchema.optional(),
  search: z.string().optional(),
});
```

### Discriminated Unions
```typescript
const eventSchema = z.discriminatedUnion("type", [
  z.object({ type: z.literal("click"), x: z.number(), y: z.number() }),
  z.object({ type: z.literal("scroll"), offset: z.number() }),
]);
```

## Step 3: After Creation

1. **Export** from schemas index file
2. **Export types** — always co-export inferred types
3. **Wire to routes** — pass schemas to route validation
4. **Wire to forms** — use with `@hookform/resolvers/zod` for React Hook Form

## Rules

1. **Always export inferred types** — `z.infer<typeof schema>`, never manual interfaces
2. **Coerce query params** — URL values are strings, use `z.coerce.number()` etc.
3. **Reuse common schemas** — pagination, sorting, filtering if they exist
4. **Meaningful validation messages** — `.min(1, "Name is required")`
5. **`.default()` for optional with defaults** — `z.number().optional().default(50)`
6. **`.partial()` for update schemas** — derive from create schema when possible
