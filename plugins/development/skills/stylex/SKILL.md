---
name: stylex
description: Write StyleX styles correctly — longhand properties, nested pseudo-classes/media queries, no shorthands
---

# StyleX Authoring Guide

## Writing styles

Styles must be created using `stylex.create()`. Define styles as an object with namespaces containing CSS properties.

```tsx
import * as stylex from '@stylexjs/stylex';

const styles = stylex.create({
  container: {
    display: 'flex',
    alignItems: 'center',
    padding: 16,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    color: 'navy',
  },
});
```

**CRITICAL RULES:**
- Use **longhand properties** only. Multi-value shorthands like `border`, `margin`, `padding` (with multiple values), `background`, `flex`, `overflow` DO NOT WORK.
- Use `null` to unset properties.
- Length properties are in pixels by default (numbers without units).

### Shorthand → Longhand Conversions

These shorthands **DO NOT WORK** in StyleX. Always expand them:

```tsx
// WRONG — border shorthand
border: "1px solid var(--border-primary)",

// CORRECT — longhand properties
borderWidth: 1,
borderStyle: "solid",
borderColor: "var(--border-primary)",

// WRONG — individual side border shorthand
borderLeft: "3px solid red",

// CORRECT
borderLeftWidth: 3,
borderLeftStyle: "solid",
borderLeftColor: "red",

// WRONG — margin/padding multi-value
margin: "8px 16px",
padding: "0.75rem 1.5rem",

// CORRECT
marginTop: "8px",
marginRight: "16px",
marginBottom: "8px",
marginLeft: "16px",
paddingTop: "0.75rem",
paddingRight: "1.5rem",
paddingBottom: "0.75rem",
paddingLeft: "1.5rem",

// WRONG — non-standard shorthands (these silently produce 0px!)
paddingX: "1rem",
paddingY: "2rem",
marginX: "auto",

// CORRECT
paddingLeft: "1rem",
paddingRight: "1rem",
paddingTop: "2rem",
paddingBottom: "2rem",
marginLeft: "auto",
marginRight: "auto",

// WRONG — background shorthand
background: "linear-gradient(to right, red, blue)",

// CORRECT
backgroundImage: "linear-gradient(to right, red, blue)",

// WRONG — flex shorthand
flex: "1 0 auto",

// CORRECT
flexGrow: 1,
flexShrink: 0,
flexBasis: "auto",

// WRONG — overflow shorthand with two values
overflow: "hidden auto",

// CORRECT
overflowX: "hidden",
overflowY: "auto",

// OK — single-value shorthands that DO work:
padding: 16,           // single value is fine
margin: 0,             // single value is fine
borderRadius: 8,       // single value is fine (applies to all corners)
overflow: "hidden",    // single value is fine
```

### Allowed single-value shorthands

These work because they set a single value:
- `padding: 16` (all sides same value)
- `margin: 0` (all sides same value)
- `borderRadius: 8` (all corners same value)
- `overflow: "hidden"` (both axes same value)
- `gap: 8` (both row and column gap)

---

## Applying styles

Convert StyleX style objects to props using `stylex.props()`:

```tsx
function Component() {
  return (
    <div {...stylex.props(styles.container)}>
      <h1 {...stylex.props(styles.title)}>Hello</h1>
    </div>
  );
}
```

### Merging styles

Pass multiple styles to merge them. The last style wins for conflicting properties:

```tsx
<div {...stylex.props(styles.base, styles.highlighted)} />
<div {...stylex.props([styles.base, styles.highlighted])} />
```

### Conditional styles

```tsx
<div
  {...stylex.props(
    styles.base,
    isActive && styles.active,
    isDisabled && styles.disabled,
    variant === 'primary' ? styles.primary : styles.secondary,
  )}
/>
```

### Passing styles as props

```tsx
import type { StyleXStyles } from '@stylexjs/stylex';

type Props = {
  children: React.ReactNode;
  style?: StyleXStyles;
};

function Card({ children, style }: Props) {
  return <div {...stylex.props(styles.card, style)}>{children}</div>;
}
```

---

## Pseudo-classes and pseudo-elements

**Nest inside property values** — NEVER at the top level:

```tsx
// WRONG — pseudo-class at top level
const styles = stylex.create({
  button: {
    ':hover': {
      backgroundColor: 'blue',
    },
  },
});

// CORRECT — nested inside property value
const styles = stylex.create({
  button: {
    backgroundColor: {
      default: 'lightblue',
      ':hover': 'blue',
      ':active': 'darkblue',
      ':focus-visible': 'royalblue',
      ':disabled': 'gray',
    },
    cursor: {
      default: 'pointer',
      ':disabled': 'not-allowed',
    },
  },
});
```

Pseudo-elements as top-level keys within a namespace:

```tsx
const styles = stylex.create({
  input: {
    color: 'black',
    '::placeholder': {
      color: 'gray',
    },
  },
});
```

---

## Media queries

**Nest inside property values** — NEVER at the top level:

```tsx
// WRONG
const styles = stylex.create({
  container: {
    '@media (min-width: 768px)': {
      padding: 16,
    },
  },
});

// CORRECT
const styles = stylex.create({
  container: {
    padding: {
      default: 8,
      '@media (min-width: 768px)': 16,
    },
  },
});
```

The `default` key is **required** when using nested conditions.

---

## Dynamic styles

Use arrow functions for runtime values:

```tsx
const styles = stylex.create({
  bar: (width: number) => ({
    width,
  }),
  positioned: (x: number, y: number) => ({
    transform: `translate(${x}px, ${y}px)`,
  }),
});

<div {...stylex.props(styles.bar(100))} />
```

---

## Keyframe animations

```tsx
const fadeIn = stylex.keyframes({
  from: { opacity: 0 },
  to: { opacity: 1 },
});

const styles = stylex.create({
  animated: {
    animationName: fadeIn,
    animationDuration: '0.3s',
    animationTimingFunction: 'ease-out',
  },
});
```

---

## Variables and Constants

Use `stylex.defineVars()` for themed values, `stylex.defineConsts()` for static values. Must be in `.stylex.ts` files with named exports only.

```tsx
// tokens.stylex.ts
export const colors = stylex.defineVars({
  primary: 'blue',
  background: 'white',
});

// constants.stylex.ts
export const breakpoints = stylex.defineConsts({
  small: '@media (max-width: 600px)',
  large: '@media (min-width: 1025px)',
});
```

---

## Common antipatterns

1. **Don't use multi-value shorthands** — `border`, `margin: "8px 16px"`, `padding: "0 1rem"`, `background`, `flex: "1 0 auto"` all fail silently
2. **Don't use `paddingX`/`paddingY`/`marginX`/`marginY`** — these are NOT valid CSS or StyleX properties and produce 0px silently
3. **Don't nest pseudo-classes/media queries at top level** — they must be inside property values
4. **Don't import non-StyleX values into `stylex.create()`** — use `defineVars`/`defineConsts` instead
5. **Don't mix `className`/`style` props with `stylex.props()`**
6. **Don't use `:focus`** — use `':focus-visible'` for keyboard focus styling (better accessibility)
