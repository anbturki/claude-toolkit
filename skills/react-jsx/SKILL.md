---
name: react-jsx
description: Keep JSX declarative and scannable. No inline complex logic, no ternary chains, no inline object/handler creation. Extract logic above the return statement. Use when reviewing JSX.
user-invocable: true
allowed-tools: Read, Edit, Grep, Glob
argument-hint: "[component or file]"
---

# Clean JSX

JSX should be declarative and scannable. Move ALL logic out of JSX.

## Scope

`$ARGUMENTS`

## Rules

### 1. No complex inline logic

```typescript
// BAD - logic buried in JSX
return (
  <div>
    {items.filter(i => i.active && i.type === "premium").length > 0 ? (
      <ul>{items.filter(i => i.active && i.type === "premium").map(i => <li key={i.id}>{i.name}</li>)}</ul>
    ) : (
      <EmptyState title="No items" />
    )}
  </div>
);

// GOOD - logic extracted, JSX is clean
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

### 2. No long ternaries

Ternaries longer than 1 line -> use early returns or extract to a variable.

### 3. No `&&` chains for conditional rendering

Use early returns instead - they read top-to-bottom and avoid the falsy-zero bug.

```typescript
// BAD - if items.length is 0, this renders "0"
{items.length && <List items={items} />}

// GOOD - early return
if (items.length === 0) return <EmptyState />;
return <List items={items} />;
```

### 4. No inline object/array creation in props

Each render creates a new reference, breaking memoization downstream.

```typescript
// BAD - new object every render
<Card style={{ padding: 16 }} options={{ closable: true }} />

// GOOD - hoist to module scope or useMemo
const CARD_STYLE = { padding: 16 };
const CARD_OPTIONS = { closable: true };
<Card style={CARD_STYLE} options={CARD_OPTIONS} />
```

### 5. No inline handlers with logic

Extract to a named function. Keeps the JSX scannable and makes the handler testable.

```typescript
// BAD - inline complex handler
<Button onClick={() => {
  setLoading(true);
  api.deleteServer(id).then(() => {
    toast.success("Deleted");
    router.push("/servers");
  }).catch(err => toast.error(err.message));
}}>Delete</Button>

// GOOD - extracted named handler
function handleDelete() {
  deleteMutation.mutate(id);
}

<Button onClick={handleDelete} loading={deleteMutation.isPending}>
  Delete
</Button>
```

### 6. No string concatenation for classNames

Use `cn()` / `clsx` / `tailwind-merge` (whatever the project uses).

```typescript
// BAD
<div className={"card " + (active ? "card--active" : "") + " " + size}>

// GOOD
<div className={cn("card", active && "card--active", size)}>
```

## See also

- [[react-components]] - file-level structure (this skill handles per-return)
- [[react-performance]] - inline objects/handlers are also a performance trap
- [[clean-code-readability]] - the underlying principle (no inline complex logic)
