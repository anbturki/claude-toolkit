---
name: elysia-route
description: Scaffold an Elysia API route with CRUD endpoints, validation, and auth. Use when adding API endpoints to an Elysia backend.
argument-hint: [entity-name]
allowed-tools: Read, Write, Edit, Grep, Glob, Bash(ls *)
---

# Create Elysia Route

Scaffold a new Elysia route file for `$ARGUMENTS`.

## Step 1: Learn Project Patterns

1. **Read CLAUDE.md** for API conventions
2. **Find existing routes**:
   ```
   Glob("**/routes/**/*.ts")
   Glob("**/*.routes.ts")
   ```
3. **Read 2-3 existing route files** — learn:
   - Import style for services and schemas
   - Auth plugin usage
   - Middleware/guard patterns
   - Response format
   - How routes are registered in the app

## Step 2: Scaffold Route

**File location:** Match existing route file locations (e.g., `src/routes/${entity}.routes.ts`)

```typescript
import { ${Entity}Service } from "<service-import>"; // match project imports
import {
  create${Entity}Schema,
  list${Entity}sSchema,
  update${Entity}Schema,
} from "<schema-import>"; // match project imports
import { Elysia } from "elysia";

export const ${entity}Routes = new Elysia({
  name: "${entities}",
  prefix: "/${entities}",
  tags: ["${Entity}s"],
})
  // Apply auth/middleware matching existing routes
  .use(authPlugin)
  // List
  .get(
    "/",
    async ({ query }) => {
      return ${Entity}Service.list(query);
    },
    {
      query: list${Entity}sSchema,
      detail: { description: "List all ${entities}" },
    },
  )
  // Create
  .post(
    "/",
    async ({ body }) => {
      return ${Entity}Service.create(body);
    },
    {
      body: create${Entity}Schema,
      detail: { description: "Create a new ${entity}" },
    },
  )
  // Get by ID
  .get(
    "/:id",
    async ({ params }) => {
      return ${Entity}Service.getById(params.id);
    },
    {
      detail: { description: "Get ${entity} by ID" },
    },
  )
  // Update
  .put(
    "/:id",
    async ({ params, body }) => {
      return ${Entity}Service.update(params.id, body);
    },
    {
      body: update${Entity}Schema,
      detail: { description: "Update a ${entity}" },
    },
  )
  // Delete
  .delete(
    "/:id",
    async ({ params }) => {
      return ${Entity}Service.remove(params.id);
    },
    {
      detail: { description: "Delete a ${entity}" },
    },
  );
```

## Patterns

### File Upload (use Elysia's `t`, not Zod)
```typescript
import { Elysia, t } from "elysia";

.post("/import", handler, {
  body: t.Object({
    file: t.File(),
    tagId: t.Optional(t.String()),
  }),
})
```

### Nested Routes
```typescript
.get("/:id/items", async ({ params, query }) => {
  return ${Entity}Service.listItems(params.id, query);
}, { query: paginationSchema })

.post("/:id/items", async ({ params, body }) => {
  return ${Entity}Service.addItem(params.id, body);
}, { body: itemSchema })
```

### Action Endpoints
```typescript
.post("/:id/start", async ({ params }) => {
  return ${Entity}Service.start(params.id);
})
```

## Step 3: Register Route

Add to the route index (match existing registration pattern):
```typescript
import { ${entity}Routes } from "./${entity}.routes";

export const api = new Elysia({ prefix: "/api" })
  .use(${entity}Routes);
```

## Rules

1. **Method chaining** — required for Eden Treaty type inference
2. **Zod schemas for body/query** — pass to Elysia options, never call `.parse()` manually
3. **`t.File()` for uploads** — Elysia's type system, not Zod
4. **Use PUT, not PATCH** — unless the project specifically uses PATCH
5. **Services handle business logic** — routes only parse request and call services
6. **Return data directly** — Eden Treaty wraps in `{ data, error }` automatically
7. **Match existing auth/middleware** — apply same plugins as other routes
