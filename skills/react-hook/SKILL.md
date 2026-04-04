---
name: react-hook
description: Scaffold React hooks for data fetching (queries) and mutations. Auto-adapts to TanStack Query, SWR, or custom hooks. Use when adding data hooks to a frontend feature.
argument-hint: [entity-name]
allowed-tools: Read, Write, Edit, Grep, Glob, Bash(ls *)
---

# Create React Hook

Scaffold data hooks for `$ARGUMENTS`.

## Step 1: Detect Data Fetching Library

1. **Read CLAUDE.md** for data fetching conventions
2. **Detect library**:
   - `@tanstack/react-query` → TanStack Query
   - `swr` → SWR
   - `@apollo/client` → Apollo Client (GraphQL)
   - Custom hooks → match existing pattern
3. **Find existing hooks**:
   ```
   Glob("**/hooks/use-*")
   Glob("**/hooks/*-queries*")
   Glob("**/hooks/*-mutations*")
   ```
4. **Read 2-3 existing hooks** — learn patterns, error handling, cache invalidation

## Step 2: Scaffold

### TanStack Query

#### Query Hooks
```typescript
"use client"; // if Next.js App Router

import { useQuery } from "@tanstack/react-query";
import { list${Entity}s, get${Entity} } from "<api-client>";

export function use${Entity}s(params?: List${Entity}sInput) {
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

#### Mutation Hooks
```typescript
"use client";

import { useMutation, useQueryClient } from "@tanstack/react-query";
import { create${Entity}, update${Entity}, delete${Entity} } from "<api-client>";

export function useCreate${Entity}() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (input: Create${Entity}Input) => create${Entity}(input),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["${entities}"] });
    },
  });
}

export function useUpdate${Entity}() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ id, data }: { id: string; data: Update${Entity}Input }) =>
      update${Entity}(id, data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["${entities}"] });
    },
  });
}

export function useDelete${Entity}() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (id: string) => delete${Entity}(id),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["${entities}"] });
    },
  });
}
```

### SWR

#### Query Hooks
```typescript
import useSWR from "swr";

const fetcher = (url: string) => fetch(url).then(res => res.json());

export function use${Entity}s(params?: List${Entity}sInput) {
  const query = new URLSearchParams(params as Record<string, string>).toString();
  return useSWR(`/api/${entities}?${query}`, fetcher);
}

export function use${Entity}(id: string | null) {
  return useSWR(id ? `/api/${entities}/${id}` : null, fetcher);
}
```

#### Mutation Hooks (SWR + fetch)
```typescript
import useSWRMutation from "swr/mutation";

export function useCreate${Entity}() {
  return useSWRMutation("/api/${entities}", async (url, { arg }: { arg: Create${Entity}Input }) => {
    const res = await fetch(url, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(arg),
    });
    if (!res.ok) throw new Error("Failed to create");
    return res.json();
  });
}
```

### Apollo Client (GraphQL)

```typescript
import { useQuery, useMutation } from "@apollo/client";
import { GET_${ENTITIES}, CREATE_${ENTITY} } from "../graphql/${entity}.graphql";

export function use${Entity}s() {
  return useQuery(GET_${ENTITIES});
}

export function useCreate${Entity}() {
  return useMutation(CREATE_${ENTITY}, {
    refetchQueries: [{ query: GET_${ENTITIES} }],
  });
}
```

## Usage in Components

```typescript
"use client";

import { use${Entity}s, useCreate${Entity} } from "../hooks/use-${entity}-queries";

export function ${Entity}Page() {
  const { data, isLoading, error } = use${Entity}s();
  const createMutation = useCreate${Entity}();

  if (isLoading) return <Skeleton />;
  if (error) return <ErrorState error={error} />;

  const handleCreate = async (input: Create${Entity}Input) => {
    await createMutation.mutateAsync(input);
  };

  return <${Entity}List ${entities}={data} onCreate={handleCreate} />;
}
```

## Rules

1. **Use the project's data fetching library** — don't introduce a new one
2. **"use client"** directive required for hooks (Next.js App Router)
3. **Consistent query keys** — match existing naming pattern
4. **Invalidate on mutation success** — keep cache in sync
5. **Object destructuring for multi-param mutations** — `{ id, data }` pattern
6. **Import API functions from the project's API client** — don't call fetch directly
7. **Match existing hook file naming** — `use-queries.ts`, `use-mutations.ts`, or `use-${entity}.ts`
