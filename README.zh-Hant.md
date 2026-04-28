# claude-skills

> 讓 Claude Code 寫程式時跟框架官方推薦做法**齊步走**。按檔案類型自動啟動的 skill 集合，強制 AI 用最新版 Tailwind v4 / Nuxt UI v4 / Vue 3.5 / TypeScript 6 / Go Echo v5 / Fastify v5 / PostgreSQL / Astro 6 官方寫法，禁止自訂 CSS、繞過官方 API、寫過時語法。代號 `lockstep-run`（「齊步跑」）。

**Languages**: [简体中文](README.md) · **繁體中文** · [English](README.en.md) · [日本語](README.ja.md)

> 維護：[zhaoJian](https://www.zhaojian.net) · 倉庫：https://github.com/zhaojiannet/claude-skills

## 解決什麼問題

寫程式反覆遇到這種情形：Tailwind、Nuxt UI、Vue、TypeScript、Echo、Fastify、PostgreSQL 官方明明給了最直接的寫法，AI 在長 context 之後轉頭就忘，自製 CSS、繞過官方 API、寫上一代舊語法。每次發現都得手動修正一遍。CLAUDE.md、全域 memory 這種軟約束在長會話後壓不住。

`claude-skills` 用 Claude Code 官方的 [skill 機制](https://code.claude.com/docs/en/skills) 解決這個問題。每個 skill 是一份 markdown 規則文件，按檔案類型 `paths` 自動啟動：編輯 `.vue` 時 Vue / Nuxt UI 規則進 context，編輯 `.css/.scss` 時 Tailwind 規則進 context，編輯 `.go` 時 Echo / sqlc 規則進 context，編輯 `migrations/*.sql` 時 PostgreSQL migration 安全規則進 context。規則現讀現用，不會像 CLAUDE.md 在長會話後被淡忘。

讓 AI 在你的專案裡不能再用過時寫法，不能再自製輪子繞過官方元件。

## 適合誰用

- 用 **Vue / Nuxt** 專案，希望 AI 別再寫 `<style scoped>`、別再繞過 Nuxt UI 元件手寫 raw `<button>` / `<input>`
- 用 **Tailwind v4** 專案，希望 AI 別再寫 `bg-opacity-50` / `bg-gradient-to-r` 等 deprecated utility
- 用 **TypeScript** 專案，希望 AI 別再寫 `any` / `as any` / `@ts-ignore`
- 用 **Go + Echo v5** 後端，希望 AI 用 `HTTPError` + 集中 `HTTPErrorHandler` 而不是每個 handler 手寫 JSON 錯誤回應
- 用 **PostgreSQL** + migrations，希望 AI 不會寫裸 `DROP TABLE` 或一次性 `ALTER TABLE ... NOT NULL` 鎖住正式環境
- 寫程式時希望 AI 跟你「齊步走」，不是各走各的

## Installation

在 Claude Code 裡執行：

```bash
# 1. 註冊本 marketplace
/plugin marketplace add zhaojiannet/claude-skills

# 2. 安裝需要的 plugin（按專案技術棧選）
/plugin install vue-skills@lockstep-run             # Vue / Nuxt 專案
/plugin install tailwind-skills@lockstep-run        # 任何用 Tailwind 的專案
/plugin install typescript-skills@lockstep-run      # 任何 TypeScript 專案
/plugin install go-skills@lockstep-run              # Go + Echo + sqlc 後端
/plugin install node-skills@lockstep-run            # Node + Fastify 後端
/plugin install postgres-skills@lockstep-run        # PostgreSQL 專案
/plugin install astro-skills@lockstep-run           # Astro 靜態站

# 3. 重新載入使其生效
/reload-plugins
```

驗證：輸入 `/plugin` 進 **Installed** tab，能看到裝好的 plugin。再用 `What skills are available?` 讓 Claude 列出 skill。

## Plugins

| Plugin | 含 skill | 適用 |
|---|---|---|
| `vue-skills` | `prefer-nuxt-ui` / `vue-component-conventions` | Vue / Nuxt 專案 |
| `tailwind-skills` | `tailwind-utility-first` | 任何用 Tailwind 的專案（Vue / React / Astro / 純 HTML） |
| `typescript-skills` | `typescript-strict-rules` | 任何 TypeScript 專案 |
| `go-skills` | `echo-handler-patterns` / `sqlc-codegen-rules` | Go + Echo v5 + sqlc + PostgreSQL |
| `node-skills` | `fastify-plugin-patterns` | Node + Fastify v5 |
| `postgres-skills` | `postgresql-schema-design` / `postgresql-migration-safety` | PostgreSQL schema / migrations |
| `astro-skills` | `astro-static-first` | Astro 6+ 靜態站 |

## Skills

| Skill | 觸發 paths | 功能 |
|---|---|---|
| `prefer-nuxt-ui` | `**/*.vue` | Vue 檔案強制用 Nuxt UI v4 元件（121 個 U 前綴元件）而非手寫 raw `<button>` / `<input>` / `<dialog>` |
| `tailwind-utility-first` | `**/*.vue, .html, .tsx, .css, .scss` | Tailwind v4 utility-first：禁 `<style scoped>` / 舊 utility / 任意值變數；強制 `oklch()` / `(--xxx)` 圓括號 / `@custom-variant dark` / Vite 用 `@tailwindcss/vite` |
| `vue-component-conventions` | `**/*.vue` | Vue 3.5+ SFC：強制 `<script setup>` + 類型化 `defineProps`/`defineEmits` + `defineModel` + `useTemplateRef`；禁 Options API / mixins / `PropType` |
| `typescript-strict-rules` | `**/*.ts, .tsx` | TypeScript strict：禁 `any` / `@ts-ignore` / namespace / 非 const enum；強制 strict tsconfig + `unknown` 替代 any + 類型推導 |
| `echo-handler-patterns` | `**/*.go` | Echo v5 錯誤處理：handler 回傳 error 冒泡、`echo.NewHTTPError` 統一錯誤、自訂 `HTTPErrorHandler` 集中轉換、`errors.Is`/`errors.As` 區分業務錯誤、`%w` wrap chain |
| `sqlc-codegen-rules` | `**/queries/*.sql, sqlc.yaml` | sqlc v2 + pgx/v5 + 命名規範（Get/Find/List/Count/Create/Update/Delete）+ 禁手寫 SQL 呼叫 |
| `fastify-plugin-patterns` | 含 fastify import 的 `.ts/.js` | Fastify v5 plugin async 寫法、`fastify-plugin` (fp) 何時用、JSON schema 驗證、encapsulation、graceful onClose |
| `postgresql-schema-design` | `**/migrations/*.sql, **/schema/*.sql` | snake_case + BIGSERIAL/UUID PK + timestamptz + 外鍵 ON DELETE 顯式 + jsonb 僅用於 schemaless 資料 |
| `postgresql-migration-safety` | `**/migrations/*.sql` | 交易包裹 + IF EXISTS 守衛 + 禁裸 DROP/TRUNCATE + NOT NULL 加欄位分三步 + `CREATE INDEX CONCURRENTLY` + 欄位重新命名分兩階段 |
| `astro-static-first` | `**/*.astro` | Astro 6+ 靜態優先：預設 zero JS、`client:visible`/`client:idle` 優於 `client:load`、`server:defer` 替代全頁 SSR、Content Collections 替代 `Astro.glob` |

啟動方式：

- **自動**：編輯匹配 `paths` 的檔案時 Claude Code 自動 load skill 進 context
- **手動**：`/<plugin-name>:<skill-name>`，例如 `/go-skills:echo-handler-patterns`

## How it works

每個 skill 是一份 markdown 規則文件，包含：

- **Core principles**：核心原則（「用什麼」）
- **Forbidden patterns**：禁用項（「不用什麼」）+ 簡短 reason
- **Deprecated → Current**：舊 API → 新 API 對照表
- **STOP 信號**：utility / 官方 API 表達不出來時報告而非繞過規則
- 場景化規則：Tailwind 的 Preflight defaults、Vue 的 decomposition triggers、PostgreSQL 的 NOT NULL 加欄位三步法、Echo 的 HTTPErrorHandler 模板等
- **驗證 grep**：寫完後自查的指令清單

Claude Code 在編輯匹配 `paths` 的檔案時把對應 SKILL.md 內容載入 context。規則不像 CLAUDE.md 長期 always-load，按檔案類型按需載入。

## Roadmap

第一/二輪 7 個 plugin、10 個 skill 已覆蓋主流前後端棧（Vue / Tailwind / TypeScript / Go+Echo+sqlc / Node+Fastify / PostgreSQL / Astro）。後續按真實使用回饋擴展。

## Development

本地載入（開發/除錯用，不需安裝）：

```bash
cd <你的專案>
claude --plugin-dir ~/Cores/Projects/claude-skills
```

修改 SKILL.md 後在已執行的 Claude Code 內執行 `/reload-plugins` 即可生效。

除錯啟動：用 `What skills are available?` 讓 Claude 列出當前可用的所有 skill。漏觸發或誤觸發時調 SKILL.md 的 `description` 關鍵字或 `paths` glob。

官方參考：[Skills](https://code.claude.com/docs/en/skills) / [Plugins](https://code.claude.com/docs/en/plugins) / [Plugin Marketplaces](https://code.claude.com/docs/en/plugin-marketplaces)。

## 名字由來

`lockstep` 就是「步調一致」、「齊步走」的意思，很適合這個 skill。但我突然想起了我的好兄弟薛貴文（跑哥）。大學軍訓時，教官一臉嚴肅地讓他帶領全系喊口號，他連續 N 次把「齊步走」喊成了「齊步跑」，從此成為傳說中的跑哥。

謹以此名獻給跑哥，紀念那個被汗水浸透、被烈日曬透的夏天，酣暢淋漓，卻再也回不去了。

## License

MIT
