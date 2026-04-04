---
name: nextjs-feature
description: Scaffold a Next.js frontend feature with components, hooks, server/client split, and data fetching. Use when adding new UI features to a Next.js app.
argument-hint: [feature-name]
allowed-tools: Read, Write, Edit, Grep, Glob, Bash(ls *)
---

# Create Next.js Feature

Scaffold a new frontend feature for `$ARGUMENTS`.

## Step 1: Learn Project Patterns

1. **Read CLAUDE.md** for frontend conventions
2. **Detect data fetching approach**: TanStack Query, SWR, Server Components, server actions
3. **Find existing features**:
   ```
   Glob("**/features/*/")
   Glob("**/modules/*/")
   Glob("**/app/**/page.tsx")
   ```
4. **Read 2-3 existing features** — learn:
   - Directory structure
   - Server vs client component split
   - How data is fetched (RSC, hooks, server actions)
   - How mutations work
   - How types are derived
   - UI component library in use (shadcn, MUI, Chakra, etc.)

## Step 2: Create Feature Structure

```
features/${feature}/
├── index.ts                    # Barrel exports
├── components/
│   ├── ${entity}-list.tsx      # List component
│   ├── ${entity}-card.tsx      # Card/item component
│   └── create-${entity}-dialog.tsx  # Create dialog (if needed)
├── hooks/
│   ├── use-queries.ts          # Query hooks (TanStack Query/SWR)
│   └── use-mutations.ts        # Mutation hooks
└── types.ts                    # Feature-specific types (optional)
```

## Step 3: Create Files

### Query Hooks
```typescript
"use client";

import { useQuery } from "@tanstack/react-query";  // or SWR
import { list${Entity}s, get${Entity} } from "<api-client>";

export function use${Entity}s(params?: ListParams) {
  return useQuery({
    queryKey: ["${entities}", params],
    queryFn: () => list${Entity}s(params),
  });
}

export function use${Entity}(id: string) {
  return useQuery({
    queryKey: ["${entities}", id],
    queryFn: () => get${Entity}(id),
    enabled: !!id,
  });
}
```

### Mutation Hooks
```typescript
"use client";

import { useMutation, useQueryClient } from "@tanstack/react-query";
import { useRouter } from "next/navigation";
import { create${Entity}, update${Entity}, delete${Entity} } from "<api-client>";

export function useCreate${Entity}() {
  const router = useRouter();
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (input: Create${Entity}Input) => create${Entity}(input),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["${entities}"] });
      router.refresh();  // if using RSC
    },
  });
}

export function useUpdate${Entity}() {
  const router = useRouter();
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ id, data }: { id: string; data: Update${Entity}Input }) =>
      update${Entity}(id, data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["${entities}"] });
      router.refresh();
    },
  });
}

export function useDelete${Entity}() {
  const router = useRouter();
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (id: string) => delete${Entity}(id),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["${entities}"] });
      router.refresh();
    },
  });
}
```

### List Component
```typescript
import type { ${Entity}Item } from "../types";

interface ${Entity}ListProps {
  ${entities}: ${Entity}Item[];
}

export function ${Entity}List({ ${entities} }: ${Entity}ListProps) {
  if (${entities}.length === 0) {
    return <EmptyState message="No ${entities} found" />;
  }

  return (
    <div className="space-y-4">
      {${entities}.map((item) => (
        <${Entity}Card key={item.id} ${entity}={item} />
      ))}
    </div>
  );
}
```

### Page (App Router)
```typescript
// app/${entities}/page.tsx (Server Component)
import { ${Entity}List } from "@/features/${feature}";
import { list${Entity}s } from "<api-or-db>";

export default async function ${Entity}sPage() {
  const { data } = await list${Entity}s();

  return (
    <div>
      <PageHeader title="${Entity}s" />
      <${Entity}List ${entities}={data} />
    </div>
  );
}
```

## Rules

1. **"use client"** only on components with hooks/interactivity
2. **Server Components by default** — fetch data in server components, pass as props
3. **Derive types from API** — never manually duplicate response types
4. **Use project's data fetching library** — TanStack Query, SWR, or plain fetch
5. **Reuse UI components** — check existing shared components before creating new ones
6. **`router.refresh()`** after mutations — to update server components
7. **Separate data from presentation** — hooks fetch, components render
