---
name: react-components
description: React component rules - reuse-first inventory, max 150 lines, separate data from presentation, follow project patterns. Use when building or refactoring components.
user-invocable: true
allowed-tools: Read, Write, Edit, Grep, Glob
argument-hint: "[component or feature]"
---

# React Components

Components are reusable, focused, and predictable. Reuse first, create only when justified.

## Scope

`$ARGUMENTS`

## Before writing any component

### 1. Inventory existing components

Before creating anything, search for what already exists:

```
components/ui/      - UI primitives (Button, Input, Dialog, Select, etc.)
components/         - shared composed components
features/*/         - feature-specific components, hooks, and types
```

Run these searches EVERY TIME:

- `Glob("**/components/**/*.tsx")` - all shared components
- `Glob("**/components/ui/**/*.tsx")` - all UI primitives
- `Grep` for similar component names or patterns in the target feature

### 2. Check feature structure

```
features/{name}/
  components/     - UI components (presentation)
  hooks/          - data fetching & mutations
  types.ts        - feature-specific types (optional)
```

## Rules

### 1. Reuse first, create never (unless justified)

**Never create a component that already exists.** This is the #1 rule.

Before writing ANY component or element:

1. Check `components/ui/` for UI primitives
2. Check `components/` for composed shared components
3. Check the same feature's `components/` for similar components
4. Check other features' `components/` for transferable patterns

**If you need a new shared component**, it must:

- Be used (or usable) in 2+ places
- Not duplicate any existing component's purpose
- Follow the same pattern as existing components (props interface, composition)

### 2. Small, focused files

- **Max 150 lines per component file** - split if larger
- **Max 50 lines of JSX** in a single return - extract sub-components
- **Max 3 levels of JSX nesting** - flatten with extracted components
- **One component per file** - no multi-component files (except tiny internal helpers)

```typescript
// BAD - 300-line monolith
export function DeployDialog() {
  // 50 lines of hooks and state
  // 200 lines of JSX with nested conditions
}

// GOOD - composed from focused pieces
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

### 3. Separate data from presentation

Components should EITHER fetch data OR render UI - never both.

```typescript
// BAD - fetching AND rendering in one component
export function ServerList() {
  const { data, isLoading } = useQuery({ queryKey: ["servers"], queryFn: listServers });
  if (isLoading) return <Skeleton />;
  return <div>{data.map(...)}</div>;
}

// GOOD - hook in parent/page, presentation component receives props
export function useServers() {
  return useQuery({ queryKey: ["servers"], queryFn: listServers });
}

interface ServerListProps { servers: Server[]; }
export function ServerList({ servers }: ServerListProps) {
  return <div>{servers.map(...)}</div>;
}
```

**Exception:** page-level components (route files) can compose hooks and presentation - they ARE the integration point.

### 4. Consistent patterns

Follow the existing codebase patterns exactly:

- **API calls:** go through the project's data-fetching pattern (API client -> hook -> component)
- **Form handling:** use the project's form library (React Hook Form, Formik, etc.) with schema validation
- **Loading states:** use the project's loading/skeleton components
- **Error handling:** follow the project's error handling pattern
- **Empty states:** use existing empty state components with meaningful message and action
- **Dialogs:** compose with the project's dialog primitives

## File naming

```
components/server-card.tsx        - kebab-case, noun-based
hooks/use-server-queries.ts       - use-* prefix, kebab-case
types.ts                          - plain types.ts per feature
```

## See also

- [[react-jsx]] - keeping JSX clean
- [[react-hooks]] - the hooks that feed these components
- [[react-performance]] - memoization rules
- [[react-accessibility]] - semantic HTML and ARIA
- [[clean-code-srp]] - the underlying principle
