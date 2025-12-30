# Modrinth Monorepo 構造ガイド（トップダウン）

このドキュメントは、Modrinth のコードモノレポ全体をトップダウンで俯瞰するための日本語ガイドです。  
フロントエンド（Nuxt/Vite/Tauri）、バックエンド（Rust/Actix + SQLx）、共通ライブラリ群（TypeScript・Rust）の構成と、主要ディレクトリが何を担っているかをできるだけ平易にまとめています。

- **パッケージ管理**: `pnpm` を用いた JavaScript/TypeScript のワークスペース管理と、`Cargo` を用いた Rust ワークスペース管理のハイブリッド構成。
- **タスク実行**: `turbo` によるパイプライン（ビルド・lint・test・prepr）が全体を束ねる。
- **主要プロダクト**: Web（`apps/frontend`）、デスクトップアプリ（`apps/app` + `apps/app-frontend`）、API サーバー（`apps/labrinth`）、ドキュメント（`apps/docs`）、ユーティリティ群（`packages/*`）。

---

## 1. ルート直下の構成

- `package.json` / `pnpm-workspace.yaml` / `pnpm-lock.yaml`  
	- JavaScript/TypeScript ワークスペースの中枢。`turbo` のエントリや `prepr` 系スクリプトをここからキックする。
- `Cargo.toml` / `Cargo.lock`  
	- Rust ワークスペースのエントリ。`apps/*` や `packages/*` の Rust クレートがメンバーとして登録されている。
- `turbo.jsonc`  
	- `build`・`lint`・`test`・`prepr` などのタスク依存とキャッシュ設定を一元管理。
- `docker-compose.yml`  
	- ローカル開発で使う周辺サービス（例: データベースや ClickHouse）の起動定義。
- `scripts/`  
	- `scripts/run.mjs` など、monorepo で使う共通スクリプト置き場。
- `patches/`  
	- `pnpm` の patchedDependencies を格納。依存のピン留めや修正をここに保持する。
- `CLAUDE.md` / `COPYING.md` / `README.md` / `tombi.toml` など  
	- プロジェクトメタ情報。`README.md` から Web・App の個別 README へ誘導。

---

## 2. `apps/`（完成物・実行物が入る階層）

### 2.1 `apps/frontend` — Web フロントエンド
- **技術**: Nuxt 3（`nuxi`）、Tailwind v3、Vue 3。  
- **依存**: `@modrinth/ui`・`@modrinth/assets`・`@modrinth/api-client`・`@modrinth/blog` など社内パッケージ。  
- **主な構成**:
	- `src/` … ページ、レイアウト、コンポーネント、i18n リソース。
	- `package.json` … `dev`/`build`/`lint`/`intl:extract` スクリプト。
	- `tailwind.config.ts` / Nuxt 設定ファイル（通常は `nuxt.config.ts`。Nuxt CLI である `nuxi` が参照する）  
	- `.env` テンプレートに基づく環境変数依存（`SITE_URL` など）。
- **開発コマンド**: `pnpm web:dev`、`pnpm web:build`。PR 前チェックは `pnpm prepr:frontend:web` が推奨。

### 2.2 `apps/app` — デスクトップアプリ（Tauri Rust バンドル）
- **役割**: Electron 代替の Tauri を用いたネイティブラッパー。  
- **構成**:
	- `src/` … Rust 側エントリとコマンド定義。
	- `icons/`, `nsis/`, `capabilities/` … 配布用アセット・インストーラー設定。
	- `package.json` … `tauri dev`/`tauri build`、`cargo fmt`/`clippy`/`nextest` を束ねる lint/test。
	- `Cargo.toml` … Tauri 依存と `theseus`（`packages/app-lib`）などを参照。
- **開発コマンド**: `pnpm app:dev`（Rust とフロントをまとめて起動）。

### 2.3 `apps/app-frontend` — デスクトップアプリ用フロントエンド
- **役割**: Tauri ウィンドウに表示される Vue 3 + Vite の UI 層。  
- **構成**:
	- `src/` … ページ・コンポーネント・Pinia ストア。  
	- `vite.config.ts` / `tailwind.config.ts` / `tsconfig*.json`。  
	- `package.json` … `dev`/`build`/`lint`/`intl:extract`。
- **依存**: `@modrinth/ui`・`@modrinth/assets`・`@modrinth/api-client`・`@tanstack/vue-query`・`tauri` プラグイン群。

### 2.4 `apps/app-playground`
- **役割**: Rust 製のサンドボックス/実験用 Tauri プロジェクト。  
- **構成**: `.cargo/`, `src/` を含む最小限の Tauri ランナー。プロトタイピング用途。

### 2.5 `apps/daedalus_client`
- **役割**: Minecraft クライアント関連の Rust ツール（ランチャー連携など）を実験・検証するためのクレート。  
- **構成**: `src/` に CLI/ライブラリの実装がまとまる。

### 2.6 `apps/docs` — ドキュメントサイト
- **技術**: Astro + Starlight。  
- **構成**:
	- `src/` … ページ、コンポーネント、Starlight 設定。  
	- `astro.config.mjs` / `tsconfig.json`。  
	- `package.json` … `astro dev`/`astro build`、`astro check` を利用。
- **依存**: `@modrinth/assets` と Starlight OpenAPI プラグインで API リファレンスを統合。

### 2.7 `apps/labrinth` — バックエンド API
- **役割**: Modrinth の主 API サーバー（Actix + SQLx + Redis + ClickHouse）。  
- **構成**:
	- `src/` … エンドポイント、サービス、ドメインロジック。
	- `migrations/` … SQLx 用マイグレーション。
	- `fixtures/` … テストデータ。
	- `assets/` … 静的ファイル。
	- `tests/` … 結合テスト。
	- `docker_utils/` … 開発用ユーティリティ。
- **開発の要点**:
	- PR 前には `cargo clippy -p labrinth --all-targets` を警告ゼロで通す必要あり。  
	- テスト実行は `cargo test -p labrinth --all-targets`。  
	- SQLx キャッシュ準備は **`apps/labrinth` ディレクトリ内でのみ**実行する。例:  
		```
		cd apps/labrinth
		cargo sqlx prepare
		```
		ルートや `--workspace` で走らせると他クレートのスキーマが混ざって壊れるため。

---

## 3. `packages/`（共有ライブラリ・コンポーネント群）

### 3.1 TypeScript/JavaScript パッケージ
- `@modrinth/api-client` (`packages/api-client`)  
	- Nuxt/Tauri/ブラウザで共通利用できる HTTP クライアント。`mitt` と `ofetch` を使用。
- `@modrinth/ui` (`packages/ui`)  
	- Vue 3 コンポーネントとユーティリティ集合。`intl-messageformat`/`vue-i18n` による i18n 対応、Three.js/CodeMirror/ApexCharts など UI 依存を同梱。
- `@modrinth/assets` (`packages/assets`)  
	- アイコン・イラスト・ブランドアセット。`build/generate-exports.ts` によるアイコン自動生成・検証タスクを持つ。
- `@modrinth/moderation` (`packages/moderation`)  
	- モデレーション UI ロジックとコンポーネント。`@modrinth/api-client`/`@modrinth/ui` と連携。
- `@modrinth/blog` (`packages/blog`)  
	- ブログ記事のビルド/コンパイル・RSS 生成。`jiti` でスクリプトを実行し、`compiled/` に出力する設計。
- `@modrinth/utils` (`packages/utils`)  
	- 共有ユーティリティ。`highlightjs` や `three` など分割されたサブディレクトリに関連リソースを保持。
- `@modrinth/tooling-config` (`packages/tooling-config`)  
	- ESLint/Prettier/Tailwind/TypeScript の共通設定プリセットとスクリプトユーティリティ。
- `@modrinth/assets` と連携する `packages/assets/icons`/`branding`/`styles` などは UI/フロント系で再利用される。
- `@modrinth/path-util` などその他 TS ユーティリティはワークスペースの細分化を補助。

### 3.2 Rust パッケージ
- `packages/app-lib`（crate 名: `theseus`）  
	- Tauri アプリのコアロジック。圧縮・HTTP・ファイルシステム・DB（SQLite via SQLx）などアプリ基盤を提供。
- `packages/daedalus`  
	- Minecraft 関連のユーティリティクレート（NBT/プロファイル処理など）。
- `packages/ariadne`  
	- Rust 内部用のユーティリティ/エラーハンドリング基盤。
- `packages/modrinth-log`  
	- ロギング周り（`tracing` ベース）の共通クレート。
- `packages/modrinth-maxmind`  
	- MaxMind データベース操作のヘルパー。
- `packages/modrinth-util`  
	- バックエンド共通のユーティリティ。
- `packages/path-util`  
	- パス操作周りの共通化。
- `packages/muralpay`  
	- 決済周辺のヘルパー（Stripe 等との連携に対応）。

---

## 4. 共通設定・CI 周辺

- **Lint/Format**  
	- JS/TS: `pnpm lint`（`turbo run lint` 経由で各パッケージの ESLint + Prettier + 補助 lint が走る）。  
	- Rust: 各クレートごとに `cargo fmt --check` + `cargo clippy`、アプリによっては feature 付きで 2 回実行。  
- **Build/Test**  
	- JS/TS: `pnpm build` / `pnpm test` は `turbo` で各パッケージの `build`・`test` を並列。  
	- Rust: `cargo test` はワークスペース全体で走るが、`apps/labrinth` など重量級は個別実行が推奨。  
- **Prepr**  
	- PR 前の整形・i18n 抽出を `pnpm prepr` / `pnpm prepr:frontend` / `pnpm prepr:frontend:web` / `pnpm prepr:frontend:app` などで統合実行。
- **i18n ワークフロー**  
	- フロント系パッケージは `formatjs extract` を `intl:extract` スクリプトで実行し、`src/locales/en-US` にキーを生成。

---

## 5. ディレクトリ別ミニマップ（トップダウンの道標）

- `apps/`  
	- `frontend/` … Web クライアント  
	- `app/` … Tauri バンドル（Rust）  
	- `app-frontend/` … Tauri 用 Vue UI  
	- `app-playground/` … Tauri 実験場  
	- `daedalus_client/` … Minecraft クライアント向けツール  
	- `docs/` … Astro ドキュメント  
	- `labrinth/` … Actix ベース API サーバー
- `packages/`  
	- TypeScript: `api-client/` `ui/` `assets/` `moderation/` `blog/` `utils/` `tooling-config/` `path-util/` など  
	- Rust: `app-lib/` `ariadne/` `daedalus/` `modrinth-log/` `modrinth-maxmind/` `modrinth-util/` `muralpay/`
- `scripts/` … monorepo 用 Node スクリプト  
- `patches/` … pnpm のパッチ適用  
- `.github/` … CI 設定、テンプレート、アセット  
- `docker-compose.yml` … DB/ClickHouse など開発用サービス定義  
- `tombi.toml` / `clippy.toml` / `rustfmt.toml` など … 各種ツール設定

---

## 6. 開発開始のチェックリスト（抜粋）

1. **依存インストール**  
	- `pnpm install`（Node 側）  
	- Rust ツールチェーン（`rustup`）と必要なら `sqlx` CLI を用意
2. **主要タスク例**  
	- Web: `pnpm web:dev` / `pnpm web:build`  
	- App: `pnpm app:dev` / `pnpm app:build`  
	- Docs: `pnpm docs:dev`（root のショートカット） / `pnpm --filter @modrinth/docs build`（`package.json` の `build` スクリプト経由で `astro build` が走る）  
	- API: `cargo run -p labrinth`（DB が必要、`docker-compose` で起動）  
3. **品質チェック**  
	- JS/TS: `pnpm lint`、必要に応じて `pnpm test`  
	- Rust: 各 crate ごとに `cargo fmt --check` + `cargo clippy --all-targets`、必要に応じ `cargo test`  
4. **i18n 抽出**  
	- フロント系で文言変更時は `pnpm --filter <pkg> intl:extract` を実行し JSON を更新。

---

## 7. 補足: 役割別の関連性

- Web（`apps/frontend`）と App UI（`apps/app-frontend`）は、共通 UI/アセットを `@modrinth/ui`・`@modrinth/assets` から取得し、API 呼び出しは `@modrinth/api-client` を介して行う。  
- デスクトップアプリは Rust ランタイム（`apps/app`）とフロント（`apps/app-frontend`）を Tauri で束ね、ビジネスロジックの多くを `packages/app-lib`（`theseus`）に集約。  
- バックエンド（`apps/labrinth`）は Rust ワークスペースの各ユーティリティクレート（`modrinth-util` や `modrinth-log` など）を利用しつつ、DB マイグレーションや外部サービス連携を行う。  
- ドキュメント（`apps/docs`）は `packages/assets` を再利用し、API スキーマを Starlight OpenAPI で公開する。

---

## 8. さらに深く見るための入口

- **コードリーディングの優先順**  
	1. 目的のアプリ (`apps/frontend` / `apps/app*` / `apps/labrinth`) を選ぶ。  
	2. 対応する共有パッケージ（UI・API クライアント・ユーティリティ）を `packages/` から辿る。  
	3. ルートの `turbo.jsonc` と各 `package.json` / `Cargo.toml` のタスク定義を確認し、実行手順を把握する。  
- **環境変数の確認**  
	- Web/App 前提の `.env` テンプレート、API 用の `DATABASE_URL`/`SQLX_OFFLINE` などを適宜セット。
- **CI/PR 前チェック**  
	- `pnpm prepr:frontend:web` または `pnpm prepr:frontend:app`、Rust 系なら `cargo clippy` + `cargo sqlx prepare`（`apps/labrinth` 直下で実行）を忘れずに。

---

このガイドが、Modrinth モノレポを初めて触る際の地図として役立てば幸いです。各パッケージの詳細は、個別の `README` や `package.json`/`Cargo.toml` を参照してください。
