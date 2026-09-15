---
name: elysia-service
description: Scaffold a service module with CRUD operations for an Elysia/Bun backend. Follows singleton object pattern with repository access and custom errors.
argument-hint: [entity-name]
allowed-tools: Read, Write, Edit, Grep, Glob, Bash(ls *)
---

# Create Service (Elysia/Bun)

Scaffold a new service module for `$ARGUMENTS`.

## Step 1: Learn Project Patterns

1. **Read CLAUDE.md** for service conventions
2. **Find existing services**:
   ```
   Glob("**/services/**/*.ts")
   Glob("**/*.service.ts")
   Glob("**/modules/**/services/**")
   ```
3. **Read 2-3 existing services** — learn:
   - How they import repositories
   - Error handling pattern (custom error classes)
   - Tenant/org isolation (if multi-tenant)
   - Export style (singleton object, class, etc.)

## Step 2: Scaffold Service

Match the project's directory structure for the service file.

```typescript
import { ${Entity}Repository, type ${Entity} } from "<db-import>";
import { ${Entity}NotFoundError } from "<errors-import>";
import type {
  Create${Entity}Input,
  Update${Entity}Input,
} from "<schemas-import>";

async function list(/* match existing params like orgId */): Promise<${Entity}[]> {
  return ${Entity}Repository.findAll(/* params */);
}

async function getById(id: string): Promise<${Entity}> {
  const entity = await ${Entity}Repository.findById(id);
  if (!entity) {
    throw new ${Entity}NotFoundError();
  }
  return entity;
}

async function create(
  input: Create${Entity}Input,
  /* match existing params like userId */
): Promise<${Entity}> {
  return ${Entity}Repository.create({
    ...input,
    // createdBy: userId, if applicable
  });
}

async function update(
  id: string,
  input: Update${Entity}Input,
): Promise<${Entity}> {
  const existing = await ${Entity}Repository.findById(id);
  if (!existing) {
    throw new ${Entity}NotFoundError();
  }
  const updated = await ${Entity}Repository.update(id, input);
  if (!updated) {
    throw new ${Entity}NotFoundError();
  }
  return updated;
}

async function remove(id: string): Promise<{ id: string }> {
  const existing = await ${Entity}Repository.findById(id);
  if (!existing) {
    throw new ${Entity}NotFoundError();
  }
  await ${Entity}Repository.remove(id);
  return { id };
}

export const ${Entity}Service = {
  list,
  getById,
  create,
  update,
  remove,
};
```

## Step 3: Create Supporting Files

### Barrel Exports
```typescript
// services/index.ts
export { ${Entity}Service } from "./${entity}.service";

// module/index.ts
export { ${Entity}Service } from "./services";
```

### Error Classes (if not using `create-error` skill)
```typescript
export class ${Entity}NotFoundError extends NotFoundError {
  constructor() {
    super("${entity}NotFound");
  }
}
```

## Step 4: After Creation

1. **Export** from the module index
2. **Add package export path** (monorepo): `"./${entity}": "./src/${entity}/index.ts"`
3. **Create error classes** if they don't exist
4. **Create repository** if it doesn't exist

## Rules

1. **Services = pure business logic** — no HTTP concepts (no request, response, status codes)
2. **Throw errors, don't return them** — use custom error classes
3. **Use repositories** — never access DB directly in services
4. **Singleton object export** — `export const Service = { ... }`, not classes (unless project uses classes)
5. **Tenant isolation** — if multi-tenant, always filter by orgId/tenantId
6. **Single responsibility** — one method per operation
