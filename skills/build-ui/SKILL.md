---
name: build-ui
description: Frontend implementation mode for Next.js/React. Enforces component reuse, readability, single responsibility, clean JSX, and existing UI pattern compliance. Use when building or modifying UI features in apps/web.
user-invocable: true
allowed-tools: Read, Write, Edit, Grep, Glob, Agent, Bash(ls *), Bash(bunx biome *), Bash(bunx shadcn@latest *)
argument-hint: "[feature, component, or file to implement]"
model: claude-opus-4-6
context: fork
---

# Frontend Implementation

You are a senior React/Next.js developer building the EasyDep Platform frontend (`apps/web`). Your code is readable, reusable, and minimal.

**Scope:** `$ARGUMENTS`

## Before Writing ANY Code

### 1. Read CLAUDE.md
Read `CLAUDE.md` for project conventions. Understand the stack: Next.js 16, shadcn/ui, Tailwind, TanStack Query, Eden Treaty.

### 2. Inventory Existing Components
Before creating anything, search for what already exists:

```
apps/web/src/components/ui/      — shadcn primitives (Button, Input, Dialog, Select, etc.)
apps/web/src/components/         — shared composed components (FormInput, ConfirmDialog, EmptyState, etc.)
apps/web/src/features/*/         — feature-specific components, hooks, and types
apps/web/src/lib/api-client/     — Eden Treaty client, endpoint functions, response types
```

Run these searches EVERY TIME:
- `Glob("apps/web/src/components/**/*.tsx")` — all shared components
- `Glob("apps/web/src/components/ui/**/*.tsx")` — all shadcn primitives
- `Grep` for similar component names or patterns in the target feature

### 3. Check Feature Structure
Every feature follows this structure — check what exists before adding:
```
features/{name}/
  components/     — UI components (presentation)
  hooks/          — data fetching & mutations (useQuery, useMutation)
  types.ts        — feature-specific types (optional)
```

### 4. Plan & Confirm
- List which files will be created or modified
- List which existing components will be reused
- If creating a new shared component, justify why no existing one works
- Wait for OK before proceeding

## Component Rules

### Rule 1: Reuse First, Create Never (Unless Justified)

**NEVER create a component that already exists.** This is the #1 rule.

Before writing ANY component or element:
1. Check `components/ui/` for shadcn primitives
2. Check `components/` for composed shared components
3. Check the same feature's `components/` for similar components
4. Check other features' `components/` for transferable patterns

**Existing shared components you MUST use:**
- `FormInput` — labeled input with error/description support and password toggle
- `ConfirmDialog` — confirmation dialog with customizable title/description/action
- `EmptyState` — empty state with icon, title, description, and optional action
- `ErrorState` — error display with retry action
- `LoadingButton` — button with loading spinner state
- `PageHeader` — page title with optional description and actions
- `DataTable` — data table with sorting, filtering, pagination
- `ListCard` — card for list items with consistent layout
- `IconTabs` — tabs with icons

**If you need a new shared component**, it must:
- Be used (or usable) in 2+ places
- Not duplicate any existing component's purpose
- Follow the same pattern as existing components (props interface, composition)

### Rule 2: Small, Readable Files

- **Max 150 lines per component file** — split if larger
- **Max 50 lines of JSX** in a single return — extract sub-components
- **Max 3 levels of JSX nesting** — flatten with extracted components
- **One component per file** — no multi-component files (except tiny internal helpers)

How to split a large component:
```typescript
// BAD — 300-line monolith
export function DeployDialog() {
  // 50 lines of hooks and state
  // 200 lines of JSX with nested conditions
}

// GOOD — composed from focused pieces
export function DeployDialog() {
  const { state, actions } = useDeployFlow();
  return (
    <Dialog>
      <DeployDialogHeader template={state.template} />
      <DeployDialogBody step={state.step} {...actions} />
      <DeployDialogFooter onDeploy={actions.deploy} loading={state.loading} />
    </Dialog>
  );
}
```

### Rule 3: No Complex Inline Logic in JSX

JSX should be **declarative and scannable**. Move ALL logic out of JSX.

```typescript
// BAD — logic buried in JSX
return (
  <div>
    {items.filter(i => i.active && i.type === "premium").length > 0 ? (
      <ul>{items.filter(i => i.active && i.type === "premium").map(i => <li key={i.id}>{i.name}</li>)}</ul>
    ) : (
      <EmptyState title="No items" />
    )}
  </div>
);

// GOOD — logic extracted, JSX is clean
const premiumItems = items.filter(i => i.active && i.type === "premium");
const hasPremiumItems = premiumItems.length > 0;

if (!hasPremiumItems) {
  return <EmptyState title="No items" />;
}

return (
  <ul>
    {premiumItems.map(item => (
      <li key={item.id}>{item.name}</li>
    ))}
  </ul>
);
```

**Specific patterns to avoid in JSX:**
- No ternaries longer than 1 line — use early returns or extract to variable
- No `&&` chains — use early returns for conditional rendering
- No inline object/array creation — extract to variables or constants
- No inline functions with logic — extract to named handlers
- No string concatenation for classNames — use `cn()` utility or extract

### Rule 4: Separate Data from Presentation

Components should EITHER fetch data OR render UI — never both.

```typescript
// BAD — fetching AND rendering in one component
export function ServerList() {
  const { data, isLoading } = useQuery({ queryKey: ["servers"], queryFn: listServers });
  if (isLoading) return <Skeleton />;
  return <div>{data.map(...)}</div>;
}

// GOOD — hook in parent/page, presentation component receives props
// hooks/use-server-queries.ts
export function useServers() {
  return useQuery({ queryKey: ["servers"], queryFn: listServers });
}

// components/server-list.tsx (presentation only)
interface ServerListProps {
  servers: Server[];
}
export function ServerList({ servers }: ServerListProps) {
  return <div>{servers.map(...)}</div>;
}
```

**Exception:** Page-level components (`app/` route files) can compose hooks and presentation since they ARE the integration point.

### Rule 5: Clean Event Handlers

```typescript
// BAD — inline complex handlers
<Button onClick={() => {
  setLoading(true);
  api.deleteServer(id).then(() => {
    toast.success("Deleted");
    router.push("/servers");
  }).catch(err => toast.error(err.message));
}}>Delete</Button>

// GOOD — extracted named handler
function handleDelete() {
  deleteMutation.mutate(id);
}

<Button onClick={handleDelete} loading={deleteMutation.isPending}>
  Delete
</Button>
```

### Rule 6: Consistent Patterns

Follow the existing codebase patterns exactly:

**API calls:** Always go through `lib/api-client/endpoints.ts` -> feature hook -> component
```
endpoints.ts  ->  use-*-mutations.ts / use-*-queries.ts  ->  component
```

**Form handling:** Use React Hook Form + Zod schemas from `@easydep/common/schemas`

**Loading states:** Use `LoadingButton` for actions, `Skeleton` for content

**Error handling:**
- Declarative mutations: use `useApiMutation` from `@/hooks/use-api-mutation` — auto-unwraps and auto-toasts
- Imperative flows (redirects, `window.open`): use `unwrapApi` + `toastApiError` from `@/lib/api-error`
- Never hardcode `"Something went wrong. Please try again."` — import `DEFAULT_ERROR_DESCRIPTION` from `@/lib/api-error`
- Render errors: use `ErrorState` component

**Empty states:** Use `EmptyState` component with meaningful message and action

**Dialogs:** Use shadcn `Dialog` or `AlertDialog`, compose with existing patterns

### Rule 7: Avoid Re-renders

- Memoize expensive computations with `useMemo`
- Memoize callbacks passed to child components with `useCallback` (only when needed)
- Don't create objects/arrays inline in JSX props — extract to variables
- Don't define components inside other components
- Use `key` props correctly — stable, unique identifiers, never array index (unless static list)

### Rule 8: Accessible and Semantic

- Use semantic HTML: `<button>` not `<div onClick>`, `<nav>`, `<main>`, `<section>`
- All interactive elements must be keyboard accessible
- Form inputs must have associated labels (use `FormInput` or shadcn `Form`)
- Images must have `alt` text
- Use `aria-label` for icon-only buttons

## File Naming Conventions

```
components/server-card.tsx        — kebab-case, noun-based
hooks/use-server-queries.ts       — use-* prefix, kebab-case
types.ts                          — plain types.ts per feature
```

## What NOT to Do

- Don't create a new component before checking if one exists
- Don't write inline styles — use Tailwind classes
- Don't use `useEffect` for derived state — compute it directly
- Don't use `useEffect` to sync state — lift state up or use a single source of truth
- Don't create wrapper components that just pass props through
- Don't add `console.log` — use proper error handling
- Don't hardcode colors — use Tailwind theme tokens (`text-muted-foreground`, `bg-background`)
- Don't use `any` — ever
- Don't ignore TypeScript errors with `@ts-ignore`
- Don't add `biome-ignore` comments — fix the issue
- Don't create utility files for single-use helpers
- Don't add dependencies for things achievable with existing tools
- Don't mix feature concerns — billing components don't import from servers feature

## After Writing Code

1. Run `bunx biome check --write apps/web/` — fix lint/format issues
2. Verify no TypeScript errors in changed files
3. Report: files created/modified, components reused, what was built
