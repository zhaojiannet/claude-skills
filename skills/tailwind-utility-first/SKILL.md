---
name: tailwind-utility-first
description: Enforce Tailwind CSS v4 utility-first conventions. Use this skill when editing .vue, .html, .tsx, .css, .scss files, or when the user mentions Tailwind, utility class, scoped CSS, theme variable, gradient, dark mode, container query, hover/focus, or asks to style any UI element.
paths:
  - "**/*.vue"
  - "**/*.html"
  - "**/*.tsx"
  - "**/*.css"
  - "**/*.scss"
allowed-tools:
  - Read
  - Grep
---

This skill enforces Tailwind CSS v4 conventions. The rule is utility-first: express styling through class composition, not raw CSS files.

Apply only when the project uses `tailwindcss ^4.0` or higher. If `package.json` pins an older major, **STOP** and ask the user before applying — do not write compatibility shims.

## Core principles

- **Express styling through utilities**, not custom CSS. If a utility (or composition + static arbitrary values) covers it, use it.
- **State and responsiveness use variants**, not JS class toggles. Use `hover:` `focus:` `disabled:` `checked:` `group-*:` `peer-*:` `has-[*]:` `aria-*:` `data-*:` `*:` `**:` for state, and `md:` / `@container @md:` for size.
- **Theme tokens go in `@theme`** with namespace prefixes (`--color-*`, `--spacing-*`, `--breakpoint-*`, `--radius-*`, `--shadow-*`, `--font-*`, `--text-*`, etc.). Non-namespaced variables go in `:root`. Custom colors use `oklch()` to match the v4 palette.
- **Class-based dark mode is configured in CSS**: `@custom-variant dark (&:where(.dark, .dark *));`. Not in JS config.
- **Vite projects use `@tailwindcss/vite`**, not `@tailwindcss/postcss`.

## Forbidden patterns

### Config / build

- `tailwind.config.js`/`.ts` with `theme` / `content` / `darkMode` — express in CSS via `@theme` / `@source` / `@custom-variant`
- `@tailwind base/components/utilities` — use `@import "tailwindcss";`
- `@tailwindcss/postcss` in Vite projects — use `@tailwindcss/vite`
- Manual `postcss-import` / `autoprefixer` — built in
- Legacy directives `@variants` / `@responsive` / `@screen` — gone, use variant prefixes directly

### CSS authoring

- `<style scoped>` in component files, scattered raw `.css/.scss`. **Allowed**: entry CSS may have `@layer components { ... }` for reusable composite classes and `@layer base { ... }` for preflight overrides.
- `[var(--xxx)]` for variable arbitrary values — use `(--xxx)` parens. Square brackets are for **static** values only (`w-[500px]`, `grid-cols-[max-content_auto]`).
- `theme()` function in CSS — use `var(--color-xxx)` or `--alpha(var(--color-xxx) / 50%)`.
- `!flex` prefix important — use `flex!` suffix.
- Static inline `style="..."` for things expressible as utilities. **Allowed**: dynamic runtime values (props/state) may combine inline style for the dynamic part with utility class for the static part.
- Numeric arbitrary values like `text-[16px]` / `p-[8px]` — use the token (`text-base` / `p-2`).
- Custom colors as `#hex` / `rgb()` — write `oklch(L C H)` to match the v4 palette.
- JS-driven class toggles for hover/focus/resize that a Tailwind variant or container query already covers.
- Hand-written `@utility name { ... }` or `@theme { --token-X: ... }` when the entry CSS (`app.css` / `main.css`) already defines an equivalent. grep first; reuse if found.

## Deprecated → Current

| ❌ Don't write | ✅ Use instead |
|---|---|
| `bg-opacity-50` / `text-opacity-*` and other `*-opacity-*` | `bg-color/50` / `text-color/50` (slash opacity) |
| `flex-shrink-*` / `flex-grow-*` | `shrink-*` / `grow-*` |
| `bg-gradient-to-r` | `bg-linear-to-r` |
| `outline-none` | `outline-hidden` |
| `shadow-sm` / `shadow` (scale shifted down) | `shadow-xs` / `shadow-sm` |
| `rounded-sm` / `blur-sm` | `rounded-xs` / `blur-xs` |
| `ring` (default 3px blue-500) | `ring-3 ring-blue-500` (default is now 1px currentColor) |
| `!flex` (prefix important) | `flex!` (suffix important) |
| `first:*:pt-0` (variant right→left) | `*:first:pt-0` (variant left→right) |
| `@layer utilities { .x {} }` | `@utility x {}` |
| `theme(spacing.12)` | `var(--spacing-12)` or `--spacing(12)` |
| `overflow-ellipsis` | `text-ellipsis` |
| `decoration-slice` / `decoration-clone` | `box-decoration-slice` / `box-decoration-clone` |
| `space-y-*` (descendant selector) | `flex flex-col gap-*` (more predictable) |
| `border` defaulting to gray-200 | explicit `border-gray-200` (default is now `currentColor`) |

## When utilities are not enough

Do not write `<style scoped>` to bypass. **STOP** and report:

> Need [visual/behavior]. Tried utility composition [classes attempted]. Cannot express [missing capability]. Approve one of: (A) register `@theme { --xxx: ... }` token, (B) add `@utility xxx { ... }`, (C) add `@layer components { .xxx { ... } }`, (D) use a UI library component.

## Reuse / abstraction ladder (utilities suffice but the same class string repeats across files)

Different from the previous section ("utilities cannot express it") — here utilities cover the styling, but the same class list appears repeatedly across files. Pick by scenario (official `styling-with-utility-classes` guidance):

| Scenario | Solution |
|---|---|
| Repetition within a single file | Loops / multi-cursor editing — **no abstraction** |
| Cross-file reuse of a complex component (multi-element structure) | Component / template partial (official "best strategy") |
| Cross-file reuse of a single-element simple class (btn / card / badge) | `@layer components { .x { CSS properties + var(--token) directly } }` |
| Overriding third-party library styles | `@apply` or `@layer components` — either works |

v4 idiom: the official `@layer components` examples (`.btn-primary` / `.card`) **write CSS properties directly and reference theme variables via `var(--…)`**, not via `@apply`. The official position of `@apply` in the v4 docs is "override third-party library styles", **not the standard tool for reusing utility strings**.

## Complex arbitrary values → @theme token

Arbitrary values are officially positioned as "**once in a while** to break out of constraints" / pixel-perfect tweaks. When complex arbitrary values like `shadow-[0_1px_3px_...]`, `ease-[cubic-bezier(...)]`, or `bg-[oklch(...)]` are reused across files, that violates this positioning — promote them to `@theme` as named tokens. The official `@theme` examples already demonstrate this pattern with `--color-X: oklch(...)` and `--ease-X: cubic-bezier(...)`.

## Preflight defaults that surprise

| Element | Default | If you want different |
|---|---|---|
| `<button>` | `cursor: default` | `class="cursor-pointer"` or `@layer base { button { cursor: pointer; } }` |
| `<dialog>` | `margin: 0` | `class="m-auto"` |
| `<img>/<video>/<canvas>` | `display: block; max-width: 100%` | `class="inline"` / `class="max-w-none"` |
| `<h1>~<h6>` | `font-size/weight: inherit` | utility class (e.g. `text-2xl font-bold`) |
| `<ul>/<ol>` | `list-style: none` | `class="list-disc list-inside"`; for a11y add `role="list"` |
| `border` color | `currentColor` | explicit `border-gray-200` |
| `placeholder` color | current text color at 50% | `@layer base { ::placeholder { color: var(--color-gray-400); } }` |
| `[hidden]` attribute | `display: none !important` | **cannot** be overridden by utilities — remove the `hidden` attribute |

## Container queries (component-level responsiveness)

For reusable components, prefer container queries over viewport breakpoints:

```html
<div class="@container">
  <div class="flex flex-col @md:flex-row @max-md:gap-2">
    <!-- relative to the parent .@container, not the viewport -->
  </div>
</div>
```

Naming: `@container/main` + `@sm/main:`. Arbitrary sizes: `@min-[475px]:` / `@max-[600px]:`. Custom: `@theme { --container-8xl: 96rem; }` → `@8xl:`.

## Source detection & references

Tailwind v4 auto-scans every file except: anything in `.gitignore`, `node_modules`, binaries, CSS, and lock files. Extend with `@source "../path"` (e.g. a shared UI package in a monorepo). Single-file `<style>` blocks that need `@apply` or `@variant` must add `@reference "../app.css";` at the top.

Full syntax: https://tailwindcss.com/docs

## Verification (grep after every styling change)

```bash
grep -rE '\[var\(--' .                       # should use (--xxx)
grep -r '<style scoped>' .                   # remove — switch to utilities or @layer components
grep -r '@tailwind ' .                       # should use @import "tailwindcss"
grep -rE '@variants|@responsive|@screen' .  # deprecated
grep -rE 'bg-opacity-|text-opacity-|flex-shrink-|flex-grow-' .
grep -r 'bg-gradient-to-' .                  # should use bg-linear-to-
grep -rE 'class="[^"]*![a-z]' .              # prefix important
grep -rE 'class="[^"]*(text|p|m|w|h)-\[[0-9]' .  # numeric arbitrary values
grep -r 'darkMode:' .                        # should migrate to @custom-variant
grep -r '@tailwindcss/postcss' .             # Vite projects should use @tailwindcss/vite
```
