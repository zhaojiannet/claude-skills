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

### 配置 / 构建

- `tailwind.config.js`/`.ts` with `theme` / `content` / `darkMode` — express in CSS via `@theme` / `@source` / `@custom-variant`
- `@tailwind base/components/utilities` — use `@import "tailwindcss";`
- `@tailwindcss/postcss` in Vite projects — use `@tailwindcss/vite`
- Manual `postcss-import` / `autoprefixer` — built in
- Legacy directives `@variants` / `@responsive` / `@screen` — gone, use variant prefixes directly

### CSS 写法

- `<style scoped>` in component files, scattered raw `.css/.scss`. **Allowed**: entry CSS may have `@layer components { ... }` for reusable composite classes and `@layer base { ... }` for preflight overrides.
- `[var(--xxx)]` for variable arbitrary values — use `(--xxx)` parens. Square brackets are for **static** values only (`w-[500px]`, `grid-cols-[max-content_auto]`).
- `theme()` function in CSS — use `var(--color-xxx)` or `--alpha(var(--color-xxx) / 50%)`.
- `!flex` prefix important — use `flex!` suffix.
- Static inline `style="..."` for things expressible as utilities. **Allowed**: dynamic runtime values (props/state) may combine inline style for the dynamic part with utility class for the static part.
- Numeric arbitrary values like `text-[16px]` / `p-[8px]` — use the token (`text-base` / `p-2`).
- Custom colors as `#hex` / `rgb()` — write `oklch(L C H)` to match the v4 palette.
- JS-driven class toggles for hover/focus/resize that a Tailwind variant or container query already covers.

## Deprecated → Current

| ❌ 不要写 | ✅ 改用 |
|---|---|
| `bg-opacity-50` / `text-opacity-*` 等 `*-opacity-*` | `bg-color/50` / `text-color/50`（slash 透明度） |
| `flex-shrink-*` / `flex-grow-*` | `shrink-*` / `grow-*` |
| `bg-gradient-to-r` | `bg-linear-to-r` |
| `outline-none` | `outline-hidden` |
| `shadow-sm` / `shadow`（scale 整体下移） | `shadow-xs` / `shadow-sm` |
| `rounded-sm` / `blur-sm` | `rounded-xs` / `blur-xs` |
| `ring`（默认 3px blue-500） | `ring-3 ring-blue-500`（默认改 1px currentColor） |
| `!flex` 前缀 important | `flex!` 后缀 important |
| `first:*:pt-0`（变体右→左） | `*:first:pt-0`（变体左→右） |
| `@layer utilities { .x {} }` | `@utility x {}` |
| `theme(spacing.12)` | `var(--spacing-12)` 或 `--spacing(12)` |
| `overflow-ellipsis` | `text-ellipsis` |
| `decoration-slice` / `decoration-clone` | `box-decoration-slice` / `box-decoration-clone` |
| `space-y-*` 后代选择器 | `flex flex-col gap-*`（更可预期） |
| `border` 默认 gray-200 | 显式 `border-gray-200`（默认已改 `currentColor`） |

## When utilities are not enough

Do not write `<style scoped>` to bypass. **STOP** and report:

> Need [visual/behavior]. Tried utility composition [classes attempted]. Cannot express [missing capability]. Approve one of: (A) register `@theme { --xxx: ... }` token, (B) add `@utility xxx { ... }`, (C) add `@layer components { .xxx { ... } }`, (D) use a UI library component.

## Preflight defaults that surprise

| Element | Default | If you want different |
|---|---|---|
| `<button>` | `cursor: default` | `class="cursor-pointer"` 或 `@layer base { button { cursor: pointer; } }` |
| `<dialog>` | `margin: 0` | `class="m-auto"` |
| `<img>/<video>/<canvas>` | `display: block; max-width: 100%` | `class="inline"` / `class="max-w-none"` |
| `<h1>~<h6>` | `font-size/weight: inherit` | utility class（如 `text-2xl font-bold`） |
| `<ul>/<ol>` | `list-style: none` | `class="list-disc list-inside"`；保留 a11y 加 `role="list"` |
| `border` color | `currentColor` | 显式 `border-gray-200` |
| `placeholder` color | 当前文字色 50% | `@layer base { ::placeholder { color: var(--color-gray-400); } }` |
| `[hidden]` 属性 | `display: none !important` | utility 覆盖**不行**，删除 `hidden` 属性 |

## Container queries（组件级响应式）

写可重用组件时优先用容器查询而非视口断点：

```html
<div class="@container">
  <div class="flex flex-col @md:flex-row @max-md:gap-2">
    <!-- 基于父 .@container 宽度，不基于视口 -->
  </div>
</div>
```

命名：`@container/main` + `@sm/main:`。任意尺寸：`@min-[475px]:` / `@max-[600px]:`。自定义：`@theme { --container-8xl: 96rem; }` → `@8xl:`。

## Source detection 与 references

Tailwind v4 自动扫描除以下之外所有文件：`.gitignore` 列表 / `node_modules` / 二进制 / CSS / lock 文件。扩展用 `@source "../path"`（如 monorepo 共享 UI 库）。单文件 `<style>` 块若要用 `@apply` 或 `@variant`，顶部加 `@reference "../app.css";`。

完整语法：https://tailwindcss.com/docs。

## 验证（写完任何样式改动 grep 一遍）

```bash
grep -rE '\[var\(--' .                       # 应改 (--xxx)
grep -r '<style scoped>' .                   # 应删，转 utility 或 @layer components
grep -r '@tailwind ' .                       # 应改 @import "tailwindcss"
grep -rE '@variants|@responsive|@screen' .  # 已弃用
grep -rE 'bg-opacity-|text-opacity-|flex-shrink-|flex-grow-' .
grep -r 'bg-gradient-to-' .                  # 应改 bg-linear-to-
grep -rE 'class="[^"]*![a-z]' .              # 前缀 important
grep -rE 'class="[^"]*(text|p|m|w|h)-\[[0-9]' .  # 数字任意值
grep -r 'darkMode:' .                        # 应迁 @custom-variant
grep -r '@tailwindcss/postcss' .             # Vite 项目应改 @tailwindcss/vite
```
