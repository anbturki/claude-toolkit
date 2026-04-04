---
name: nextjs-route
description: Scaffold a Next.js API route (App Router or Pages Router). Use when adding server-side API endpoints in a Next.js app.
argument-hint: [entity-name]
allowed-tools: Read, Write, Edit, Grep, Glob, Bash(ls *)
---

# Create Next.js API Route

Scaffold a new API route for `$ARGUMENTS`.

## Step 1: Detect Router Type

1. **Read CLAUDE.md** for API conventions
2. **Detect router**:
   - `app/api/` directory → App Router (route.ts)
   - `pages/api/` directory → Pages Router (handler functions)
3. **Find existing API routes** and read 2-3 to match patterns

## Step 2: Scaffold

### App Router (`app/api/${entities}/route.ts`)

```typescript
import { NextRequest, NextResponse } from "next/server";
import { ${Entity}Service } from "<service-import>";
import { create${Entity}Schema } from "<schema-import>";

// List
export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const query = {
    search: searchParams.get("search") ?? undefined,
    limit: Number(searchParams.get("limit") ?? 50),
    offset: Number(searchParams.get("offset") ?? 0),
  };

  const result = await ${Entity}Service.list(query);
  return NextResponse.json(result);
}

// Create
export async function POST(request: NextRequest) {
  const body = await request.json();
  const validated = create${Entity}Schema.parse(body);
  const entity = await ${Entity}Service.create(validated);
  return NextResponse.json(entity, { status: 201 });
}
```

### App Router with Dynamic Params (`app/api/${entities}/[id]/route.ts`)

```typescript
import { NextRequest, NextResponse } from "next/server";

type Params = { params: Promise<{ id: string }> };

// Get by ID
export async function GET(request: NextRequest, { params }: Params) {
  const { id } = await params;
  const entity = await ${Entity}Service.getById(id);
  return NextResponse.json(entity);
}

// Update
export async function PUT(request: NextRequest, { params }: Params) {
  const { id } = await params;
  const body = await request.json();
  const validated = update${Entity}Schema.parse(body);
  const entity = await ${Entity}Service.update(id, validated);
  return NextResponse.json(entity);
}

// Delete
export async function DELETE(request: NextRequest, { params }: Params) {
  const { id } = await params;
  await ${Entity}Service.remove(id);
  return NextResponse.json({ id });
}
```

### Pages Router (`pages/api/${entities}/index.ts`)

```typescript
import type { NextApiRequest, NextApiResponse } from "next";

export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  switch (req.method) {
    case "GET": {
      const result = await ${Entity}Service.list(req.query);
      return res.json(result);
    }
    case "POST": {
      const validated = create${Entity}Schema.parse(req.body);
      const entity = await ${Entity}Service.create(validated);
      return res.status(201).json(entity);
    }
    default:
      res.setHeader("Allow", ["GET", "POST"]);
      return res.status(405).end();
  }
}
```

### Server Actions (if the project uses them)

```typescript
"use server";

import { revalidatePath } from "next/cache";

export async function create${Entity}Action(formData: FormData) {
  const input = create${Entity}Schema.parse({
    name: formData.get("name"),
  });
  await ${Entity}Service.create(input);
  revalidatePath("/${entities}");
}
```

## Rules

1. **Match the project's router type** — don't mix App Router and Pages Router
2. **Validate input** — parse body with the project's validation library
3. **Proper status codes** — 201 for create, 200 for others
4. **Auth** — apply the same auth pattern as existing routes (middleware, session checks, etc.)
5. **Error handling** — match existing error response format
