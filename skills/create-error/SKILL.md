---
name: create-error
description: Scaffold custom error classes for backend services. Follows project patterns for HTTP errors, domain errors, and error codes.
argument-hint: [error-name]
allowed-tools: Read, Write, Edit, Grep, Glob, Bash(ls *)
---

# Create Error Class

Scaffold a new error class for `$ARGUMENTS`.

## Step 1: Learn Project Patterns

1. **Read CLAUDE.md** for error handling conventions
2. **Find existing error classes**:
   ```
   Glob("**/errors/**/*.ts")
   Glob("**/exceptions/**/*.ts")
   Grep("extends.*Error")
   ```
3. **Read existing errors** — learn:
   - Base class hierarchy (AppError, HttpError, etc.)
   - Error code convention (camelCase, UPPER_SNAKE, kebab-case)
   - HTTP status mapping
   - Where error files live

## Step 2: Scaffold Error Class

Adapt to the project's error pattern:

### Pattern A: HTTP Base Classes
```typescript
// Common base classes the project may already have:
// NotFoundError (404), BadRequestError (400), ConflictError (409),
// UnauthorizedError (401), ForbiddenError (403)

// Not Found
export class ${Entity}NotFoundError extends NotFoundError {
  constructor() {
    super("${entity}NotFound");
  }
}

// Duplicate / Conflict
export class Duplicate${Entity}Error extends ConflictError {
  constructor() {
    super("duplicate${Entity}");
  }
}

// Business Rule Violation
export class Invalid${Entity}Error extends BadRequestError {
  constructor(message?: string) {
    super("invalid${Entity}");
    if (message) this.message = message;
  }
}

// Limit Exceeded
export class ${Entity}LimitExceededError extends BadRequestError {
  constructor(limit: number) {
    super("${entity}LimitExceeded");
    this.message = `Cannot exceed ${limit} ${entities}`;
  }
}
```

### Pattern B: NestJS Exceptions
```typescript
import { NotFoundException, ConflictException, BadRequestException } from "@nestjs/common";

export class ${Entity}NotFoundException extends NotFoundException {
  constructor(id: string) {
    super(`${Entity} with id ${id} not found`);
  }
}

export class Duplicate${Entity}Exception extends ConflictException {
  constructor() {
    super("${Entity} already exists");
  }
}
```

### Pattern C: Custom Error with Code
```typescript
export class AppError extends Error {
  constructor(
    public readonly code: string,
    public readonly statusCode: number,
    message: string,
  ) {
    super(message);
    this.name = this.constructor.name;
  }
}

export class ${Entity}NotFoundError extends AppError {
  constructor(id?: string) {
    super("${ENTITY}_NOT_FOUND", 404, id ? `${Entity} ${id} not found` : "${Entity} not found");
  }
}
```

## Error Code Conventions

Match the project's naming convention:

| Convention | Example |
|------------|---------|
| camelCase | `${entity}NotFound`, `duplicate${Entity}` |
| UPPER_SNAKE | `${ENTITY}_NOT_FOUND`, `DUPLICATE_${ENTITY}` |
| kebab-case | `${entity}-not-found` |

## Common Error Types

| Error | Base | Code | When |
|-------|------|------|------|
| NotFound | 404 | `${entity}NotFound` | Resource doesn't exist |
| Duplicate | 409 | `duplicate${Entity}` | Already exists (unique constraint) |
| Invalid | 400 | `invalid${Entity}` | Business rule violation |
| LimitExceeded | 400 | `${entity}LimitExceeded` | Quota or limit hit |
| Locked | 403 | `${entity}Locked` | Cannot modify in current state |
| Unauthorized | 401 | `unauthorized` | Auth required |

## Usage in Services

```typescript
// Check existence
const entity = await ${Entity}Repository.findById(id);
if (!entity) {
  throw new ${Entity}NotFoundError();
}

// Check uniqueness
const existing = await ${Entity}Repository.findByName(name);
if (existing) {
  throw new Duplicate${Entity}Error();
}
```

## Frontend Error Handling

Errors typically arrive as:
```json
{ "code": "${entity}NotFound", "message": "...", "status": 404 }
```

Handle in forms:
```typescript
try {
  await mutation.mutateAsync(data);
} catch (error) {
  const code = getErrorCode(error);  // match project's error utility
  if (code === "${entity}NotFound") {
    // specific handling
  } else {
    // generic handling
  }
}
```

## Rules

1. **Match existing error hierarchy** — use the project's base classes
2. **Match error code convention** — same casing as existing errors
3. **Throw errors, don't return them** — services throw, routes/middleware catch
4. **Error codes are stable** — they're part of the API contract, don't change them
5. **Add to the right file** — group by domain (domain.ts, integration.ts, etc.)
