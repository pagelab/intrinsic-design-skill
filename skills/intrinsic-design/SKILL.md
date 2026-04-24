---
name: intrinsic-design
description: 'CRITICAL CSS SKILL: Use this whenever you are writing or reviewing CSS layouts, making components responsive, or the user mentions fluid typography, responsive grids, breakpoints, container queries (@container), clamp(), minmax(), or intrinsic design. You must use this skill to apply modern breakpoint-free CSS layout techniques (auto-fit, flex-wrap, cqi container units, fluid values) instead of standard @media max-width queries. Also use when adding device/accessibility preferences like pointer, hover, or prefers-reduced-motion. Do NOT use for Tailwind configurations, HTML image optimization, or basic CSS alignment bugs.'
---

# Intrinsic Design — Breakpoint-Free CSS Layout

Build responsive interfaces where components adapt to their context by default, without relying on viewport breakpoints as the primary layout engine. Media queries are reserved for device capabilities and user preferences only.

## Core Philosophy

> **Responsive design is moving from breakpoint choreography to intent-driven systems.**

Breakpoints were an excellent answer when multiple screen sizes emerged, but modern interfaces are component-first, nested, and reused across wildly different contexts. Global viewport width is frequently the wrong input for local layout decisions.

This skill implements a **four-method approach** that makes viewport breakpoints optional:

| Method | Purpose | Replaces |
|--------|---------|----------|
| **Intrinsic Layouts** | Structure adapts continuously | Fixed column counts at breakpoints |
| **Fluid Values** | Scale smoothly between min/max | Stepped size changes at breakpoints |
| **Container Units** | Size relative to component, not viewport | Viewport-relative sizing in components |
| **Container Queries** | Structural shifts at component level | Structural shifts at viewport level |

**@media queries** are reserved exclusively for:
- Device capabilities (`hover`, `pointer`, `display-mode`, `update`, `scripting`)
- User preferences (`prefers-color-scheme`, `prefers-reduced-motion`, `prefers-contrast`, `prefers-reduced-data`)

## When to Use This Skill

- Writing or reviewing any CSS layout code
- Building responsive components, grids, or page layouts
- Refactoring breakpoint-heavy CSS to intrinsic patterns
- Creating reusable components that must work in sidebars, modals, feeds, and dashboards
- Any time layout decisions are being made with `@media (min-width)` or `@media (max-width)`

## Method 1: Intrinsic Layouts First

Push adaptation into layout primitives. Instead of "at width X, force N columns," define constraints and let the browser derive the layout continuously.

### Card Grid — auto-fit + minmax

**❌ AVOID: Breakpoint-based grid**
```css
.card-grid {
  display: grid;
  gap: 1em;
  grid-template-columns: 1fr;
}

@media (min-width: 768px) {
  .card-grid { grid-template-columns: repeat(2, 1fr); }
}

@media (min-width: 1024px) {
  .card-grid { grid-template-columns: repeat(3, 1fr); }
}

@media (min-width: 1280px) {
  .card-grid { grid-template-columns: repeat(4, 1fr); }
}
```

**✅ USE: Intrinsic grid**
```css
.card-grid {
  display: grid;
  gap: 1em;
  grid-template-columns: repeat(auto-fit, minmax(320px, 1fr));
}
```

This says: "Make as many columns as fit, but never let them get smaller than 320px." The browser handles the rest. One line replaces four breakpoint blocks.

### Two-Region Layout — flex-wrap + calc(infinity)

**❌ AVOID: Breakpoint sidebar toggle**
```css
.layout { display: block; }
.sidebar { width: 100%; }
.main { width: 100%; }

@media (min-width: 768px) {
  .layout { display: flex; }
  .sidebar { width: 240px; }
  .main { flex: 1; }
}
```

**✅ USE: Intrinsic flex layout**
```css
.layout {
  display: flex;
  flex-wrap: wrap;
}

.main {
  /* main gets priority + readable minimum */
  flex: calc(infinity) 1 360px;
}

.sidebar {
  /* set sidebar idle size */
  flex: 1 1 240px;
}
```

The `calc(infinity)` flex-grow ensures `main` always takes all available space. When the container is too narrow for both, `flex-wrap: wrap` causes the sidebar to drop below — no breakpoint needed.

### Key Intrinsic Patterns

| Pattern | CSS | Intent |
|---------|-----|--------|
| Fluid grid | `repeat(auto-fit, minmax(MIN, 1fr))` | Fill columns, respect minimum |
| Fluid grid (no stretch) | `repeat(auto-fill, minmax(MIN, 1fr))` | Fill columns, don't stretch last row |
| Priority layout | `flex: calc(infinity) 1 BASIS` | Element gets all available space |
| Wrap sidebar | `flex: 1 1 BASIS` + parent `flex-wrap: wrap` | Stack when narrow |

## Method 2: Fluid Values

Many "responsive" rules are just scalar tuning. Typography, spacing, radius, and component size should be continuous, not stepped.

**❌ AVOID: Stepped values**
```css
.card {
  font-size: 20px;
  padding: 2em;
}

@media (max-width: 720px) {
  .card { font-size: 18px; padding: 1.5em; }
}

@media (max-width: 380px) {
  .card { font-size: 16px; padding: 1em; }
}
```

**✅ USE: Fluid values with clamp()**
```css
.card {
  font-size: clamp(16px, calc(11.529px + 1.176vi), 20px);
  padding: clamp(1em, calc(-0.118em + 4.706vi), 2em);
}
```

### Fluid Value Formula

The `clamp()` pattern: `clamp(MIN, PREFERRED, MAX)`

- **MIN** — smallest acceptable value (mobile floor)
- **PREFERRED** — fluid calculation using viewport units (`vi`, `vw`)
- **MAX** — largest acceptable value (desktop ceiling)

Use the `vi` unit (viewport inline) instead of `vw` for writing-mode-independent sizing.

### Common Fluid Tokens

```css
:root {
  /* Typography */
  --text-sm: clamp(0.875rem, 0.8rem + 0.25vi, 1rem);
  --text-base: clamp(1rem, 0.9rem + 0.5vi, 1.25rem);
  --text-lg: clamp(1.25rem, 1rem + 1vi, 1.75rem);
  --text-xl: clamp(1.5rem, 1rem + 1.5vi, 2.25rem);
  --text-2xl: clamp(2rem, 1.2rem + 2.5vi, 3rem);

  /* Spacing */
  --space-xs: clamp(0.25rem, 0.15rem + 0.5vi, 0.5rem);
  --space-sm: clamp(0.5rem, 0.3rem + 1vi, 1rem);
  --space-md: clamp(1rem, 0.6rem + 2vi, 2rem);
  --space-lg: clamp(1.5rem, 0.8rem + 3vi, 3rem);
  --space-xl: clamp(2rem, 1rem + 4vi, 4rem);

  /* Border radius */
  --radius-sm: clamp(2px, 0.25vi, 6px);
  --radius-md: clamp(4px, 0.5vi, 12px);
  --radius-lg: clamp(8px, 1vi, 24px);
}
```

### Calculating clamp() Values

Use a clamp calculator to generate exact values from two anchor points:
- **Point A**: viewport width where MIN applies
- **Point B**: viewport width where MAX applies

The formula for the preferred value:
```
preferred = MIN + (MAX - MIN) × ((100vi - A) / (B - A))
```

Simplified to: `calc(INTERCEPT + SLOPEvi)`

Tools: [clamp-calculator.netlify.app](https://clamp-calculator.netlify.app/), [utopia.fyi](https://utopia.fyi/)

## Method 3: Container Units for Local Responsiveness

When a component can render in a sidebar, modal, feed, or full-width layout, viewport-relative values produce unpredictable results. Container units solve this.

### Setting Up Container Context

```css
.card-container {
  container-type: inline-size;
}
```

### Using Container Units

```css
.card {
  /* Font size scales with the card's actual width, not the viewport */
  font-size: 5cqi;

  /* Once font-size is set, em units cascade from it */
  gap: 1em;
  padding: 1.5em;

  /* Or use clamp with container units */
  border-radius: clamp(4px, calc(-16.87px + 6.522cqi), 64px);
}
```

### Container Units Reference

| Unit | Meaning |
|------|---------|
| `cqi` | 1% of container's inline size |
| `cqb` | 1% of container's block size |
| `cqw` | 1% of container's width |
| `cqh` | 1% of container's height |
| `cqmin` | Smaller of `cqi` and `cqb` |
| `cqmax` | Larger of `cqi` and `cqb` |

### Internal Intrinsic Layout with Container Units

```css
.card {
  display: flex;
  flex-wrap: wrap;
}

.card-content {
  /* Content takes all available space but won't shrink below 40cqi */
  flex: calc(infinity) 1 40cqi;
}
```

Same CSS, completely different results depending on the container's width. Three cards in different contexts adapt perfectly without any conditional logic.

## Method 4: Container Queries for Structural Changes

When a component needs a **true structural shift** (not just scaling), use container queries. The key difference: we ask "How much space does this component have?" instead of "How wide is the screen?"

```css
.container {
  container-type: inline-size;
}

.group {
  display: flex;
  flex-direction: column;
  gap: 1em;

  @container (max-width: 32em) {
    flex-direction: row;
    gap: 0.5em;

    .icon { font-size: 1.5em; }
    p { display: none; }
  }
}
```

### When to Use Container Queries vs Intrinsic Layout

| Situation | Use |
|-----------|-----|
| Number of columns should adapt | `auto-fit` / `minmax()` (intrinsic) |
| Typography/spacing should scale | `clamp()` / container units |
| Layout direction should flip | Container query |
| Elements should show/hide | Container query |
| Completely different composition | Container query |

## @media Queries: Capabilities & Preferences Only

Media queries remain critical — but with a sharper mission. Use them exclusively for understanding the **device** and the **user environment**, not measuring pixels for layout.

### Device Capabilities

```css
/* Hover support — only apply hover effects on devices that support it */
@media (hover: hover) {
  .link:hover { color: var(--color-accent); }
}

/* Pointer accuracy — enlarge touch targets on coarse pointers */
button {
  padding: 1em;

  @media (pointer: coarse) {
    padding: 2em;
  }
}

/* Display mode — adjust for picture-in-picture */
@media (display-mode: picture-in-picture) {
  body { border: 2px solid white; }
}

/* Update frequency — reduce expensive effects on slow displays */
.glass-panel {
  backdrop-filter: blur(16px);

  @media (update: slow) {
    backdrop-filter: none;
    background: rgb(255 255 255 / 0.92);
  }
}

/* Scripting availability */
.no-script-msg {
  display: none;

  @media (scripting: none) {
    display: block;
  }
}
```

### User Preferences (Accessibility)

> The word "preferences" can be misleading. In many cases, these settings reflect **real accessibility needs**, not cosmetic choices.

```css
/* Color scheme */
:root {
  color-scheme: light;

  @media (prefers-color-scheme: dark) {
    color-scheme: dark;
  }
}

/* Higher contrast */
.card {
  color: #333;
  background: #ccc;

  @media (prefers-contrast: more) {
    color: #000;
    background: #fff;
  }
}

/* Reduced data */
@media (prefers-reduced-data: reduce) {
  .hero-video { display: none; }
}

/* Reduced motion — only animate when user has no preference */
.element {
  @media (prefers-reduced-motion: no-preference) {
    animation: spin 4s linear infinite;
  }
}
```

## Migration Checklist

When refactoring existing CSS to intrinsic design:

1. **Audit existing media queries** — Separate scalar changes (size, spacing, typography) from structural changes (composition/layout)
2. **Replace scalar branches first** — Move to `clamp()`, `min()`, `max()` tokens
3. **Shift layout to intrinsic primitives** — Prefer `auto-fit` and `minmax()` over fixed column counts tied to viewport widths
4. **Scope behavior to the component** — Add `container-type` and introduce container units where helpful
5. **Add container queries only where structure changes** — Keep thresholds local to the component
6. **Keep media queries for environment intent** — Favor `hover`, `pointer`, `prefers-reduced-motion`, and `update`
7. **Validate in real placements** — Test the same component in sidebar, modal, and full-content contexts

## Decision Flowchart

When writing new CSS, follow this decision path:

```
Is this a layout/grid decision?
├─ YES → Can auto-fit/minmax or flex-wrap handle it?
│        ├─ YES → Use intrinsic layout (Method 1)
│        └─ NO  → Does the component need a structural shift?
│                 ├─ YES → Use container query (Method 4)
│                 └─ NO  → Use intrinsic layout with constraints
│
Is this a scalar value (size, spacing, typography, radius)?
├─ YES → Should it scale with the viewport?
│        ├─ YES → Use clamp() with vi units (Method 2)
│        └─ NO  → Should it scale with its container?
│                 ├─ YES → Use container units cqi (Method 3)
│                 └─ NO  → Use fixed value or em/rem
│
Is this about device capability or user preference?
├─ YES → Use @media query (hover, pointer, prefers-*, etc.)
└─ NO  → You probably don't need a conditional rule at all
```

## Anti-Patterns to Flag

When reviewing code, flag these patterns for refactoring:

| Anti-Pattern | Better Approach |
|--------------|-----------------|
| `@media (min-width: 768px) { grid-template-columns: repeat(2, 1fr) }` | `repeat(auto-fit, minmax(320px, 1fr))` |
| Multiple `@media` blocks changing only `font-size` | Single `clamp()` declaration |
| Multiple `@media` blocks changing only `padding`/`gap` | Single `clamp()` declaration |
| `@media` toggling `flex-direction` | `flex-wrap: wrap` + `flex` basis constraints |
| Viewport units (`vw`, `vh`) inside reusable components | Container units (`cqi`, `cqb`) |
| `@media` for hover effects | `@media (hover: hover)` |
| Animations without motion preference check | `@media (prefers-reduced-motion: no-preference)` |

## References

- [Building a UI Without Breakpoints — Frontend Masters](https://frontendmasters.com/blog/building-a-ui-without-breakpoints/) by Amit Sheen
- [Modern Fluid Typography with clamp() — Smashing Magazine](https://www.smashingmagazine.com/2022/01/modern-fluid-typography-css-clamp/)
- [Container Queries — MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_containment/Container_size_and_style_queries)
- [@media Reference — MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/@media)
- [Clamp Calculator](https://clamp-calculator.netlify.app/)
- [Utopia Fluid Type Scale](https://utopia.fyi/)
