---
name: react-performance
description: Avoid re-renders and broken memoization. Stable keys, useMemo/useCallback only when needed, never define components inside components, never use useEffect for derived state. Use when investigating perf or reviewing renders.
user-invocable: true
allowed-tools: Read, Edit, Grep, Glob
argument-hint: "[component or file]"
---

# React Performance

The default React render is fast. Most perf problems come from broken memoization, accidental child remounts, or `useEffect` misuse - not from missing memoization.

## Scope

`$ARGUMENTS`

## Rules

### 1. Memoize only when needed

`useMemo` and `useCallback` aren't free - they add memory overhead and dependency tracking. Apply only when:

- The computation is genuinely expensive (measured, not assumed)
- The value is passed to a memoized child (`React.memo`) and would otherwise break memoization
- The value is a dependency of another hook

For everything else, plain recomputation is faster and cleaner.

### 2. Never define components inside components

This creates a new component type every render, remounting all children and losing state.

```typescript
// BAD - Inner is a new function every render
function Outer() {
  function Inner() { return <div />; }
  return <Inner />;
}

// GOOD - hoist outside
function Inner() { return <div />; }
function Outer() { return <Inner />; }
```

### 3. Stable keys, never array index for dynamic lists

```typescript
// BAD - reordering or insertion breaks state/animation
{items.map((item, i) => <Item key={i} {...item} />)}

// GOOD - unique stable identifier
{items.map(item => <Item key={item.id} {...item} />)}
```

Array index is acceptable for static lists that never reorder.

### 4. Don't create objects/arrays inline in props to memoized children

```typescript
// BAD - new reference every render, defeats React.memo
<MemoChild options={{ fast: true }} />

// GOOD - stable reference
const OPTIONS = { fast: true };
<MemoChild options={OPTIONS} />

// or with deps
const options = useMemo(() => ({ fast: speed > 0 }), [speed]);
<MemoChild options={options} />
```

### 5. Don't use `useEffect` for derived state

If a value can be computed from props/state, just compute it - don't `useState` + `useEffect`.

```typescript
// BAD - extra render, easy to desync
const [fullName, setFullName] = useState("");
useEffect(() => {
  setFullName(`${first} ${last}`);
}, [first, last]);

// GOOD - just compute it
const fullName = `${first} ${last}`;
```

### 6. Don't use `useEffect` to sync state between sources

Lift state up or treat one source as authoritative. Effects that sync are a code smell - they create races and double renders.

## See also

- [[react-components]] - presentational components with `React.memo` only when measured
- [[react-jsx]] - inline object creation is also a JSX cleanliness issue
- [[react-hooks]] - cache invalidation patterns for data hooks
