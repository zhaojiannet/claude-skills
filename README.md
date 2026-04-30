# claude-skills

> 让 Claude Code 写代码时跟框架官方推荐做法**齐步走**。按文件类型自动激活的 skill 集合，强制 AI 用最新版 Tailwind v4 / Nuxt UI v4 / Vue 3.5 / TypeScript 6 / Go Echo v5 / Fastify v5 / PostgreSQL / Astro 6 官方写法，禁止自定义 CSS、绕过官方 API、写过时语法。代号 `lockstep-run`（"齐步跑"）。

**Languages**: **简体中文** · [繁體中文](README.zh-Hant.md) · [English](README.en.md) · [日本語](README.ja.md)

> 维护：[zhaoJian](https://www.zhaojian.net) · 仓库：https://github.com/zhaojiannet/claude-skills

## 解决什么问题

写代码反复遇到这种情形：Tailwind、Nuxt UI、Vue、TypeScript、Echo、Fastify、PostgreSQL 官方明明给了最直接的写法，AI 在长 context 之后转头就忘，自己造 CSS、绕过官方 API、写上一代旧语法。每次发现都得手动纠正一遍。CLAUDE.md、全局 memory 这种软约束在长会话后压不住。

`claude-skills` 用 Claude Code 官方的 [skill 机制](https://code.claude.com/docs/en/skills) 解决这个问题。每个 skill 是一份 markdown 规则文档，按文件类型 `paths` 自动激活：编辑 `.vue` 时 Vue / Nuxt UI 规则进 context，编辑 `.css/.scss` 时 Tailwind 规则进 context，编辑 `.go` 时 Echo / sqlc 规则进 context，编辑 `migrations/*.sql` 时 PostgreSQL migration 安全规则进 context。规则现读现用，不会像 CLAUDE.md 在长会话后被淡忘。

让 AI 在你的项目里不能再用过时写法，不能再自己造轮子绕过官方组件。

## 适合谁用

- 用 **Vue / Nuxt** 项目，希望 AI 别再写 `<style scoped>`、别再绕过 Nuxt UI 组件手写 raw `<button>` / `<input>`
- 用 **Tailwind v4** 项目，希望 AI 别再写 `bg-opacity-50` / `bg-gradient-to-r` 等 deprecated utility
- 用 **TypeScript** 项目，希望 AI 别再写 `any` / `as any` / `@ts-ignore`
- 用 **Go + Echo v5** 后端，希望 AI 用 `HTTPError` + 集中 `HTTPErrorHandler` 而不是每个 handler 手写 JSON 错误响应
- 用 **PostgreSQL** + migrations，希望 AI 不会写裸 `DROP TABLE` 或一次性 `ALTER TABLE ... NOT NULL` 锁住生产
- 写代码时希望 AI 跟你"齐步走"，不是各走各的

## Installation

在 Claude Code 里执行：

```bash
# 1. 注册本 marketplace
/plugin marketplace add zhaojiannet/claude-skills

# 2. 安装唯一的 plugin（含全部 10 个 skill）
/plugin install lockstep@lockstep-run

# 3. 重新加载使其生效
/reload-plugins
```

> v0.4 起合并为单一 plugin。skill 全部按文件类型 `paths` 自动激活，装一次即可，按项目栈无需挑选。  
> 老版本（v0.3 及之前）装过 `vue-skills` / `tailwind-skills` / `typescript-skills` / `go-skills` / `node-skills` / `postgres-skills` / `astro-skills` 的，先全卸再装新版（见下方 Upgrade）。

验证：输入 `/plugin` 进 **Installed** tab，能看到 `lockstep`。再用 `What skills are available?` 让 Claude 列出 skill。

## Upgrade（从 v0.3 升级）

```bash
# 1. 卸载老 7 个 plugin
/plugin uninstall vue-skills@lockstep-run
/plugin uninstall tailwind-skills@lockstep-run
/plugin uninstall typescript-skills@lockstep-run
/plugin uninstall go-skills@lockstep-run
/plugin uninstall node-skills@lockstep-run
/plugin uninstall postgres-skills@lockstep-run
/plugin uninstall astro-skills@lockstep-run

# 2. 拉取 marketplace 最新版
/plugin marketplace update lockstep-run

# 3. 装新合并 plugin
/plugin install lockstep@lockstep-run

# 4. 重新加载
/reload-plugins
```

## Skills

| Skill | 触发 paths | 功能 |
|---|---|---|
| `prefer-nuxt-ui` | `**/*.vue` | Vue 文件强制用 Nuxt UI v4 组件（121 个 U 前缀组件）而非手写 raw `<button>` / `<input>` / `<dialog>` |
| `tailwind-utility-first` | `**/*.vue, .html, .tsx, .css, .scss` | Tailwind v4 utility-first：禁 `<style scoped>` / 旧 utility / 任意值变量；强制 `oklch()` / `(--xxx)` 圆括号 / `@custom-variant dark` / Vite 用 `@tailwindcss/vite` |
| `vue-component-conventions` | `**/*.vue` | Vue 3.5+ SFC：强制 `<script setup>` + 类型化 `defineProps`/`defineEmits` + `defineModel` + `useTemplateRef`；禁 Options API / mixins / `PropType` |
| `typescript-strict-rules` | `**/*.ts, .tsx` | TypeScript strict：禁 `any` / `@ts-ignore` / namespace / 非 const enum；强制 strict tsconfig + `unknown` 替代 any + 类型推导 |
| `echo-handler-patterns` | `**/*.go` | Echo v5 错误处理：handler 返回 error 冒泡、`echo.NewHTTPError` 统一错误、自定义 `HTTPErrorHandler` 集中转换、`errors.Is`/`errors.As` 区分业务错误、`%w` wrap chain |
| `sqlc-codegen-rules` | `**/queries/*.sql, sqlc.yaml` | sqlc v2 + pgx/v5 + 命名规范（Get/Find/List/Count/Create/Update/Delete）+ 禁手写 SQL 调用 |
| `fastify-plugin-patterns` | 含 fastify import 的 `.ts/.js` | Fastify v5 plugin async 写法、`fastify-plugin` (fp) 何时用、JSON schema 验证、encapsulation、graceful onClose |
| `postgresql-schema-design` | `**/migrations/*.sql, **/schema/*.sql` | snake_case + BIGSERIAL/UUID PK + timestamptz + 外键 ON DELETE 显式 + jsonb 仅用于 schemaless 数据 |
| `postgresql-migration-safety` | `**/migrations/*.sql` | 事务包裹 + IF EXISTS 守卫 + 禁裸 DROP/TRUNCATE + NOT NULL 加列分三步 + `CREATE INDEX CONCURRENTLY` + 列重命名分两阶段 |
| `astro-static-first` | `**/*.astro` | Astro 6+ 静态优先：默认 zero JS、`client:visible`/`client:idle` 优于 `client:load`、`server:defer` 替代全页 SSR、Content Collections 替代 `Astro.glob` |

激活方式：

- **自动**：编辑匹配 `paths` 的文件时 Claude Code 自动 load skill 进 context
- **手动**：`/lockstep:<skill-name>`，例如 `/lockstep:echo-handler-patterns`

## How it works

每个 skill 是一份 markdown 规则文档，包含：

- **Core principles**：核心原则（"用什么"）
- **Forbidden patterns**：禁用项（"不用什么"）+ 简短 reason
- **Deprecated → Current**：旧 API → 新 API 对照表
- **STOP 信号**：utility / 官方 API 表达不出来时报告而非绕过规则
- 场景化规则：Tailwind 的 Preflight defaults、Vue 的 decomposition triggers、PostgreSQL 的 NOT NULL 加列三步法、Echo 的 HTTPErrorHandler 模板等
- **验证 grep**：写完后自查的命令清单

Claude Code 在编辑匹配 `paths` 的文件时把对应 SKILL.md 内容载入 context。规则不像 CLAUDE.md 长期 always-load，按文件类型按需加载。

## Roadmap

首版 10 个 skill 已覆盖主流前后端栈（Vue / Tailwind / TypeScript / Go+Echo+sqlc / Node+Fastify / PostgreSQL / Astro）。后续按真实使用反馈扩展。

## Development

本地加载（开发/调试用，不需安装）：

```bash
cd <你的项目>
claude --plugin-dir ~/Cores/Projects/claude-skills
```

修改 SKILL.md 后在已运行的 Claude Code 内执行 `/reload-plugins` 即可生效。

调试激活：用 `What skills are available?` 让 Claude 列出当前可用的所有 skill。漏触发或误触发时调 SKILL.md 的 `description` 关键词或 `paths` glob。

官方参考：[Skills](https://code.claude.com/docs/en/skills) / [Plugins](https://code.claude.com/docs/en/plugins) / [Plugin Marketplaces](https://code.claude.com/docs/en/plugin-marketplaces)。

## 名字由来

`lockstep` 就是"步调一致"、"齐步走"的意思，很适合这个 skill。但我突然想起了我的好兄弟薛贵文（跑哥）。大学军训时，教官一脸严肃地让他带领全系喊口号，他连续 N 次把"齐步走"喊成了"齐步跑"，从此成为传说中的跑哥。

谨以此名献给跑哥，纪念那个被汗水浸透、被烈日晒透的夏天，酣畅淋漓，却再也回不去了。

## License

MIT
