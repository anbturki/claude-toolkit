---
name: nextjs-feature
description: Scaffold a Next.js feature folder with the right server/client split. Defines folder structure and Server Component vs "use client" boundaries. Pairs with react-hooks for data hooks. Use when adding a new UI feature to a Next.js app.
user-invocable: true
argument-hint: "[feature-name]"
allowed-tools: Read, Write, Edit, Grep, Glob, Bash(ls *)
---

# Next.js Feature Structure

Scaffold a new frontend feature for `$ARGUMENTS`. This skill defines **structure and boundaries** - for hook bodies see [[react-hooks]], for component rules see [[react-components]].

## Step 1: Learn project patterns

1. **Read CLAUDE.md** for frontend conventions
2. **Detect router**: App Router (`app/` directory) vs Pages Router (`pages/` directory)
3. **Detect data fetching**: Server Components, server actions, TanStack Query, SWR
4. **Find existing features**:
   ```
   Glob("**/features/*/")
   Glob("**/modules/*/")
   Glob("**/app/**/page.tsx")
   ```
5. **Read 2-3 existing features** to learn the project's specific conventions

## Step 2: Folder structure

```
features/${feature}/
  index.ts                       # Barrel exports
  components/
    ${entity}-list.tsx           # List component
    ${entity}-card.tsx           # Card/item component
    create-${entity}-dialog.tsx  # Create dialog (if needed)
  hooks/
    use-queries.ts               # Query hooks (see react-hooks skill)
    use-mutations.ts             # Mutation hooks (see react-hooks skill)
  types.ts                       # Feature-specific types (optional)
```

## Step 3: Server / client split

### Page (Server Component, App Router)

Fetch data in the server component, pass as props to client components.

```typescript
// app/${entities}/page.tsx
import { ${Entity}List } from "@/features/${feature}";
import { list${Entity}s } from "<server-side-data-source>";

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

### `"use client"` rules

- Only on components that use hooks, browser APIs, or event handlers
- Push the boundary as deep as possible - keep parents server-rendered
- Server Components can render Client Components, but not vice versa

### Mutation flow

1. Server Component renders a Client Component with a form / button
2. Client Component calls a hook from `hooks/use-mutations.ts` (see [[react-hooks]])
3. After success, call `router.refresh()` to re-render server components, or `revalidatePath()` from a server action

## Rules

1. **Server Components by default** - opt into client only for interactivity
2. **`"use client"` only on the leaf that needs it** - don't mark a whole tree as client
3. **Derive types from API** - never manually duplicate response types
4. **Use the project's data-fetching library** - don't introduce a new one
5. **Separate hooks from components** - hooks in `hooks/`, components in `components/`
6. **`router.refresh()`** after mutations to update Server Components

## See also

- [[react-hooks]] - query and mutation hook scaffolding (TanStack Query, SWR, Apollo)
- [[react-components]] - component file rules (max lines, separation of concerns)
- [[react-jsx]] - keep JSX scannable
- [[nextjs-route]] - if the feature needs API routes
