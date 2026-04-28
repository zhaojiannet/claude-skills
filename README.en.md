# claude-skills

> Make Claude Code stay in lockstep with the official Tailwind v4 / Nuxt UI v4 / Vue 3.5 / TypeScript 6 / Go Echo v5 / Fastify v5 / PostgreSQL / Astro 6 conventions. A skill set that activates by file path, forces the AI to use the latest official patterns, bans deprecated APIs, and stops it from reaching for hand-rolled CSS or workaround syntax. Codenamed `lockstep-run` ("running in step").

**Languages**: [简体中文](README.md) · [繁體中文](README.zh-Hant.md) · **English** · [日本語](README.ja.md)

> Maintained by [zhaoJian](https://www.zhaojian.net) · Repo: https://github.com/zhaojiannet/claude-skills

## What problem this solves

You write code with Tailwind, Nuxt UI, Vue, TypeScript, Echo, Fastify, PostgreSQL. The official docs give the cleanest way. The AI follows for a while, then drifts after a long context: hand-written CSS, bypassed official APIs, last-generation syntax. You correct it once, next file the same. CLAUDE.md and project memory are soft constraints that fade in long sessions.

`claude-skills` uses the official Claude Code [skill system](https://code.claude.com/docs/en/skills) to fix this. Each skill is a markdown rule document that loads by file path. Open `.vue`, the Vue and Nuxt UI rules enter context. Open `.css` or `.scss`, the Tailwind rules enter context. Open `.go`, the Echo and sqlc rules enter context. Open `migrations/*.sql`, the PostgreSQL migration safety rules enter context. The rules refresh on every file open, so they do not fade like CLAUDE.md does mid-session.

The AI no longer reaches for outdated patterns or invents components your library already provides.

## Who this is for

- **Vue / Nuxt** projects where you do not want the AI writing `<style scoped>` or hand-rolling raw `<button>` / `<input>` when Nuxt UI already has the component
- **Tailwind v4** projects where you do not want the AI writing `bg-opacity-50` / `bg-gradient-to-r` or any deprecated utility
- **TypeScript** projects where you want no more `any` / `as any` / `@ts-ignore`
- **Go + Echo v5** backends where you want `HTTPError` plus a centralized `HTTPErrorHandler` instead of every handler hand-writing JSON error responses
- **PostgreSQL** + migrations where you want the AI to never write a bare `DROP TABLE` or a single-shot `ALTER TABLE ... NOT NULL` that locks production
- Anyone tired of correcting the same kind of drift on every file

## Installation

Run inside Claude Code:

```bash
# 1. Register the marketplace
/plugin marketplace add zhaojiannet/claude-skills

# 2. Install the plugins you need
/plugin install vue-skills@lockstep-run             # Vue / Nuxt projects
/plugin install tailwind-skills@lockstep-run        # any project using Tailwind
/plugin install typescript-skills@lockstep-run      # any TypeScript project
/plugin install go-skills@lockstep-run              # Go + Echo + sqlc backend
/plugin install node-skills@lockstep-run            # Node + Fastify backend
/plugin install postgres-skills@lockstep-run        # PostgreSQL projects
/plugin install astro-skills@lockstep-run           # Astro static sites

# 3. Reload to activate
/reload-plugins
```

Verify: type `/plugin` and check the **Installed** tab. Ask Claude `What skills are available?` to confirm the skills are listed.

## Plugins

| Plugin | Skills | For |
|---|---|---|
| `vue-skills` | `prefer-nuxt-ui` / `vue-component-conventions` | Vue / Nuxt projects |
| `tailwind-skills` | `tailwind-utility-first` | any project using Tailwind (Vue / React / Astro / plain HTML) |
| `typescript-skills` | `typescript-strict-rules` | any TypeScript project |
| `go-skills` | `echo-handler-patterns` / `sqlc-codegen-rules` | Go + Echo v5 + sqlc + PostgreSQL |
| `node-skills` | `fastify-plugin-patterns` | Node + Fastify v5 |
| `postgres-skills` | `postgresql-schema-design` / `postgresql-migration-safety` | PostgreSQL schema / migrations |
| `astro-skills` | `astro-static-first` | Astro 6+ static sites |

## Skills

| Skill | Trigger paths | What it does |
|---|---|---|
| `prefer-nuxt-ui` | `**/*.vue` | Forces Nuxt UI v4 components (121 U-prefixed components) over hand-written raw `<button>` / `<input>` / `<dialog>` |
| `tailwind-utility-first` | `**/*.vue, .html, .tsx, .css, .scss` | Tailwind v4 utility-first. Forbids `<style scoped>`, scattered raw CSS, `[var(--xxx)]`, deprecated utilities. Requires `oklch()` colors, `(--xxx)` paren syntax, `@custom-variant dark`, `@tailwindcss/vite` for Vite |
| `vue-component-conventions` | `**/*.vue` | Vue 3.5+ SFC. Requires `<script setup>` + type-based `defineProps`/`defineEmits` + `defineModel` + `useTemplateRef`. Forbids Options API, mixins, `PropType` |
| `typescript-strict-rules` | `**/*.ts, .tsx` | TypeScript strict. Forbids `any`, `@ts-ignore`, namespace, non-const enum. Requires strict tsconfig, `unknown` over `any`, type inference |
| `echo-handler-patterns` | `**/*.go` | Echo v5 error model. Handlers return `error`, `echo.NewHTTPError` for client-facing errors, custom `HTTPErrorHandler` centralizes response shape, `errors.Is`/`errors.As` for business errors, `%w` wrap chain |
| `sqlc-codegen-rules` | `**/queries/*.sql, sqlc.yaml` | sqlc v2 + pgx/v5 + naming convention (Get/Find/List/Count/Create/Update/Delete). Forbids hand-written SQL when sqlc applies |
| `fastify-plugin-patterns` | `.ts/.js` files importing fastify | Fastify v5 async plugins, `fastify-plugin` (fp) when needed, JSON schema validation, encapsulation, graceful onClose |
| `postgresql-schema-design` | `**/migrations/*.sql, **/schema/*.sql` | snake_case + BIGSERIAL/UUID PK + timestamptz + explicit FK ON DELETE + jsonb only for schemaless data |
| `postgresql-migration-safety` | `**/migrations/*.sql` | Transaction wrapping + IF EXISTS guards + no bare DROP/TRUNCATE + three-step NOT NULL adds + `CREATE INDEX CONCURRENTLY` + two-phase column renames |
| `astro-static-first` | `**/*.astro` | Astro 6+ static-first. Zero JS by default, `client:visible`/`client:idle` over `client:load`, `server:defer` over full SSR, Content Collections over `Astro.glob` |

How they activate:

- **Automatic**: Claude Code loads the skill when you edit a file matching the `paths` glob.
- **Manual**: type `/<plugin-name>:<skill-name>`, e.g. `/go-skills:echo-handler-patterns`.

## How it works

Each skill is a markdown rule document with:

- **Core principles**: what to use
- **Forbidden patterns**: what to avoid, with brief reasons
- **Deprecated → Current**: old API to new API mapping
- **STOP signal**: report instead of bypass when no utility or official API fits
- Scenario-specific rules: Tailwind's Preflight defaults, Vue's decomposition triggers, PostgreSQL's three-step NOT NULL pattern, Echo's HTTPErrorHandler template
- **Verification grep**: post-edit self-check commands

Claude Code loads the matching SKILL.md into context when you open a file matching the `paths`. Unlike CLAUDE.md which loads on every session, skills load by file type, on demand.

## Roadmap

The first two waves (7 plugins, 10 skills) cover the mainstream stack: Vue / Tailwind / TypeScript / Go+Echo+sqlc / Node+Fastify / PostgreSQL / Astro. Future plugins land based on real-world feedback.

## Development

Local load (no install needed for development):

```bash
cd <your-project>
claude --plugin-dir ~/Cores/Projects/claude-skills
```

After editing a SKILL.md, run `/reload-plugins` inside the running Claude Code to pick up changes.

Debug activation: ask Claude `What skills are available?` to see what is currently loaded. If a skill triggers too rarely or too eagerly, tune the `description` keywords or the `paths` glob in the SKILL.md.

Official references: [Skills](https://code.claude.com/docs/en/skills) / [Plugins](https://code.claude.com/docs/en/plugins) / [Plugin Marketplaces](https://code.claude.com/docs/en/plugin-marketplaces).

## The name

`lockstep` means moving in unison, marching in step. It fits the spirit of this marketplace. But writing this name down, I thought of Xue Guiwen, my friend from college military training. The drill instructor handed him the megaphone for the whole department's roll call. He kept shouting "running in step" instead of "marching in step". The nickname stuck. He became the legendary 跑哥 ("Brother Run").

This name is for him, and for that summer of soaked uniforms and burnt necks. We were all in step then, even when the call came out wrong, and that summer is gone for good.

## License

MIT
