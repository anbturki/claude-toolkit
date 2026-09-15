---
name: react-accessibility
description: Semantic HTML, keyboard accessibility, form labels, alt text, aria-label for icon-only buttons. Use when reviewing UI for accessibility or when building new interactive components.
user-invocable: true
allowed-tools: Read, Edit, Grep, Glob
argument-hint: "[component or file]"
---

# Accessibility

Semantic HTML does 80% of the accessibility work for free. The other 20% is labels, focus, and keyboard support.

## Scope

`$ARGUMENTS`

## Rules

### 1. Semantic HTML over generic `<div>`

- `<button>` for clickable things, not `<div onClick>`
- `<a href>` for navigation, not `<div onClick={navigate}>`
- `<nav>`, `<main>`, `<header>`, `<footer>`, `<section>`, `<article>` for layout
- `<ul>` / `<ol>` for lists
- `<h1>` ... `<h6>` for headings (in order, no skipping)

A `<button>` gets keyboard activation, focus styles, and a screen reader role for free. A `<div onClick>` gets none of those.

### 2. All interactive elements must be keyboard accessible

If you can do it with a mouse, you must be able to do it with a keyboard:

- Tab to focus
- Enter / Space to activate buttons
- Escape to close dialogs
- Arrow keys for menus, tabs, and listboxes

If you must use a non-semantic element for interaction (rare), add `role`, `tabIndex={0}`, and `onKeyDown` for Enter/Space.

### 3. Form inputs must have associated labels

```tsx
// BAD - placeholder is not a label
<input placeholder="Email" />

// GOOD - associated label
<label htmlFor="email">Email</label>
<input id="email" />

// GOOD - wrapping label
<label>Email <input /></label>

// GOOD - screen-reader-only label when visual label is elsewhere
<input aria-label="Email" />
```

### 4. Images must have `alt` text

```tsx
// Meaningful image
<img src="/logo.png" alt="Acme Corp" />

// Decorative image - empty alt, not omitted
<img src="/divider.svg" alt="" />
```

### 5. Icon-only buttons need `aria-label`

```tsx
// BAD - screen readers announce "button"
<button onClick={handleClose}><XIcon /></button>

// GOOD
<button onClick={handleClose} aria-label="Close dialog"><XIcon /></button>
```

### 6. Focus management for dialogs and route changes

- When a dialog opens, move focus into it
- When a dialog closes, restore focus to the trigger
- Trap focus inside modal dialogs while open
- Visible focus indicators - don't `outline: none` without a replacement

### 7. Color is not the only signal

Don't convey state with color alone. Pair with text, icon, or pattern.

## See also

- [[react-components]] - composing primitives that handle a11y for you (shadcn, Radix, headless UI)
- [[react-jsx]] - clean JSX makes a11y attributes visible
