# claude-skills

> Claude Code に Tailwind v4 / Nuxt UI v4 / Vue 3.5 / TypeScript 6 / Go Echo v5 / Fastify v5 / PostgreSQL / Astro 6 の公式推奨パターンと**歩調を揃えて**コードを書かせる skill 集。ファイルタイプごとに自動で読み込まれ、最新版の公式パターンを強制し、deprecated API、独自 CSS、迂回構文を禁止します。コードネームは `lockstep-run`（「歩調を揃えて走る」）。

**Languages**: [简体中文](README.md) · [繁體中文](README.zh-Hant.md) · [English](README.en.md) · **日本語**

> メンテナンス：[zhaoJian](https://www.zhaojian.net) · リポジトリ：https://github.com/zhaojiannet/claude-skills

## 何を解決するか

開発でよく出会う場面があります。Tailwind、Nuxt UI、Vue、TypeScript、Echo、Fastify、PostgreSQL は公式が一番直接的な書き方を提示しているのに、AI は長い context のあと忘れて、独自 CSS、公式 API の迂回、一世代前の構文を書き始めます。気づくたびに手で直すしかありません。CLAUDE.md やプロジェクトメモリのようなソフトな制約は、長い会話のあとでは効きません。

`claude-skills` は Claude Code 公式の [skill 機構](https://code.claude.com/docs/en/skills) でこの問題を解決します。各 skill はファイルタイプ `paths` で自動読み込みされる markdown のルール文書です。`.vue` を開けば Vue と Nuxt UI のルールが、`.css` / `.scss` を開けば Tailwind のルールが、`.go` を開けば Echo / sqlc のルールが、`migrations/*.sql` を開けば PostgreSQL migration 安全のルールが context に入ります。ファイルを開くたびに読み直されるので、CLAUDE.md のように会話の途中で薄れません。

これで AI は古い書き方や、公式コンポーネントを迂回した手作りに戻れなくなります。

## こんな人におすすめ

- **Vue / Nuxt** プロジェクトで、AI に `<style scoped>` を書かせたくない、Nuxt UI コンポーネントを迂回して `<button>` / `<input>` を生で書かせたくない
- **Tailwind v4** プロジェクトで、AI に `bg-opacity-50` / `bg-gradient-to-r` などの deprecated utility を書かせたくない
- **TypeScript** プロジェクトで、AI に `any` / `as any` / `@ts-ignore` を使わせたくない
- **Go + Echo v5** バックエンドで、handler ごとに JSON エラー応答を手書きする代わりに、`HTTPError` と集中型 `HTTPErrorHandler` を使わせたい
- **PostgreSQL** + migrations で、AI に裸の `DROP TABLE` や本番をロックする一発 `ALTER TABLE ... NOT NULL` を書かせたくない
- 同じ手のずれを毎ファイル直すのに飽きた

## Installation

Claude Code の中で実行してください：

```bash
# 1. marketplace を登録
/plugin marketplace add zhaojiannet/claude-skills

# 2. 必要な plugin をインストール
/plugin install vue-skills@lockstep-run             # Vue / Nuxt プロジェクト
/plugin install tailwind-skills@lockstep-run        # Tailwind を使うすべてのプロジェクト
/plugin install typescript-skills@lockstep-run      # すべての TypeScript プロジェクト
/plugin install go-skills@lockstep-run              # Go + Echo + sqlc バックエンド
/plugin install node-skills@lockstep-run            # Node + Fastify バックエンド
/plugin install postgres-skills@lockstep-run        # PostgreSQL プロジェクト
/plugin install astro-skills@lockstep-run           # Astro 静的サイト

# 3. 再読み込みして有効化
/reload-plugins
```

確認：`/plugin` を入力して **Installed** タブで導入された plugin を確認できます。`What skills are available?` を Claude に聞けば、利用できる skill 一覧が出ます。

## Plugins

| Plugin | 含まれる skill | 対象 |
|---|---|---|
| `vue-skills` | `prefer-nuxt-ui` / `vue-component-conventions` | Vue / Nuxt プロジェクト |
| `tailwind-skills` | `tailwind-utility-first` | Tailwind を使うすべてのプロジェクト（Vue / React / Astro / 純 HTML） |
| `typescript-skills` | `typescript-strict-rules` | すべての TypeScript プロジェクト |
| `go-skills` | `echo-handler-patterns` / `sqlc-codegen-rules` | Go + Echo v5 + sqlc + PostgreSQL |
| `node-skills` | `fastify-plugin-patterns` | Node + Fastify v5 |
| `postgres-skills` | `postgresql-schema-design` / `postgresql-migration-safety` | PostgreSQL schema / migrations |
| `astro-skills` | `astro-static-first` | Astro 6+ 静的サイト |

## Skills

| Skill | トリガー paths | 内容 |
|---|---|---|
| `prefer-nuxt-ui` | `**/*.vue` | Vue ファイル内で Nuxt UI v4 コンポーネント（U 接頭辞 121 個）を強制し、生の `<button>` / `<input>` / `<dialog>` の手書きを禁止 |
| `tailwind-utility-first` | `**/*.vue, .html, .tsx, .css, .scss` | Tailwind v4 utility-first。`<style scoped>`、散在する raw CSS、`[var(--xxx)]`、旧版 utility を禁止。`oklch()` カラー、`(--xxx)` 括弧構文、`@custom-variant dark`、Vite では `@tailwindcss/vite` を強制 |
| `vue-component-conventions` | `**/*.vue` | Vue 3.5+ SFC。`<script setup>` + 型ベース `defineProps`/`defineEmits` + `defineModel` + `useTemplateRef` を強制。Options API、mixin、`PropType` を禁止 |
| `typescript-strict-rules` | `**/*.ts, .tsx` | TypeScript strict。`any`、`@ts-ignore`、namespace、非 const enum を禁止。strict tsconfig、`unknown` を `any` の代わり、型推論を強制 |
| `echo-handler-patterns` | `**/*.go` | Echo v5 エラー処理モデル。handler は error を return し冒泡、`echo.NewHTTPError` でクライアント向けエラー、独自 `HTTPErrorHandler` で応答形式を集中、`errors.Is`/`errors.As` で業務エラー判別、`%w` wrap chain |
| `sqlc-codegen-rules` | `**/queries/*.sql, sqlc.yaml` | sqlc v2 + pgx/v5 + 命名規約（Get/Find/List/Count/Create/Update/Delete）+ sqlc が扱える場合は手書き SQL を禁止 |
| `fastify-plugin-patterns` | fastify を import する `.ts/.js` | Fastify v5 async plugin、必要な場合の `fastify-plugin` (fp)、JSON schema 検証、encapsulation、graceful onClose |
| `postgresql-schema-design` | `**/migrations/*.sql, **/schema/*.sql` | snake_case + BIGSERIAL/UUID PK + timestamptz + 明示的な外部キー ON DELETE + jsonb は schemaless データのみ |
| `postgresql-migration-safety` | `**/migrations/*.sql` | トランザクション包囲 + IF EXISTS ガード + 裸の DROP/TRUNCATE 禁止 + NOT NULL 追加は 3 ステップ + `CREATE INDEX CONCURRENTLY` + カラム名変更は 2 段階 |
| `astro-static-first` | `**/*.astro` | Astro 6+ 静的優先。デフォルト zero JS、`client:visible`/`client:idle` を `client:load` より優先、`server:defer` を全ページ SSR の代わり、Content Collections を `Astro.glob` の代わり |

起動方法：

- **自動**：`paths` glob にマッチするファイルを編集すると Claude Code が自動的に skill を context に読み込みます
- **手動**：`/<plugin-name>:<skill-name>` の形で呼び出します。例：`/go-skills:echo-handler-patterns`

## How it works

各 skill は markdown のルール文書で、次のセクションを含みます：

- **Core principles**：核心原則（「何を使うか」）
- **Forbidden patterns**：禁止項目（「何を避けるか」）と簡潔な理由
- **Deprecated → Current**：旧 API → 新 API の対応表
- **STOP signal**：utility / 公式 API で表現できないときは回避せず報告
- シナリオ別ルール：Tailwind の Preflight デフォルト、Vue の decomposition triggers、PostgreSQL の NOT NULL 追加 3 ステップ法、Echo の HTTPErrorHandler テンプレート
- **検証 grep**：編集後の自己チェック用コマンド一覧

Claude Code は `paths` にマッチするファイルを開いたときに該当する SKILL.md を context に読み込みます。CLAUDE.md のように常時 load されるのではなく、ファイルタイプごとに必要なときだけ load されます。

## Roadmap

第 1・2 波（7 plugin、10 skill）で主流の前後端スタック（Vue / Tailwind / TypeScript / Go+Echo+sqlc / Node+Fastify / PostgreSQL / Astro）をカバー済みです。今後は実利用フィードバックに応じて追加します。

## Development

ローカル読み込み（開発・デバッグ用、インストール不要）：

```bash
cd <あなたのプロジェクト>
claude --plugin-dir ~/Cores/Projects/claude-skills
```

SKILL.md を編集したら、起動中の Claude Code で `/reload-plugins` を実行すれば反映されます。

起動デバッグ：`What skills are available?` を Claude に聞いて現在 load されている skill を確認できます。発火しない・発火しすぎるときは SKILL.md の `description` キーワードや `paths` glob を調整してください。

公式リファレンス：[Skills](https://code.claude.com/docs/en/skills) / [Plugins](https://code.claude.com/docs/en/plugins) / [Plugin Marketplaces](https://code.claude.com/docs/en/plugin-marketplaces)。

## 名前の由来

`lockstep` は「歩調が合った状態」「歩調を揃えて行進する」という意味で、この skill によく合います。けれども書きながら、大学軍事訓練時代の親友、薛貴文（通称：跑哥）を思い出しました。教官が真顔で彼に系全員の号令をかけさせたとき、彼は「歩調を揃えて行進」と言うべきところを N 回連続で「歩調を揃えて走る」と叫び、その日から伝説の「跑哥（走る兄貴）」になりました。

この名前は跑哥に贈ります。汗まみれで焼け付くような夏、思い切り叫んでいたあの夏、もう二度と戻れないあの夏のために。

## License

MIT
