# app_workspace

個人開発アプリ群を並べる作業場。各アプリは **`app_workspace/<name>/` 直下に独立した git リポジトリ** として存在し、ワークスペース自体は git 管理しない。

このファイルは「新規アプリを追加するときの規約」をまとめた一次情報。流儀を毎回 tobari/tanzaku から推測しなくて済むよう、共通項をここに集約している。

## アプリ一覧

- [tobari/](tobari/) — 視界に帳を下ろす超軽量Webアプリ群(静的Web / Cloudflare Pages)
- [tanzaku/](tanzaku/) — 縦書きブラウザアプリ(React Native + Expo)
- [souten/](souten/) — 検索欲をチャージして後で爆発させる思考の発射装置(Svelte / Cloudflare Pages)
- [saisenbako/](saisenbako/) — 「投げ銭」ではなく「賽銭」を体験させるインターネット神具(静的HTML / Cloudflare Workers)

詳細は各アプリ直下の `CLAUDE.md` を参照。ここでは概要と所在のみ。

## 新規アプリの追加手順

1. `app_workspace/<name>/` を作成し、その中で `git init`(または `git clone`)する。ワークスペース直下では git 管理しない。
2. 直下に最低限の足場を置く:
   - `CLAUDE.md` — 一次情報。下記「ファイル役割の規約」参照。
   - `README.md` — 外向け最小入口。下記「ファイル役割の規約」参照。
   - `docs/` — 仕様の本体。
   - `backlog/` — タスク管理(Backlog.md MCP)。
3. `.claude/settings.local.json` が必要ならアプリ単位で持つ。ワークスペース直下には置かない。
4. このファイル(`app_workspace/CLAUDE.md`)の「アプリ一覧」に 1 行追加する。

## ファイル役割の規約

各アプリ直下のファイルの責務を分離する。重複させない。

### `CLAUDE.md`(一次情報 / AI エージェント導入文書 / 人間向け索引)

以下を記述する:

- アプリのコンセプトと一言概要
- 技術スタック
- ディレクトリ構成
- 設計思想・運用ルール
- デプロイ手順(あれば)
- `docs/` 配下への索引
- 末尾に Backlog.md MCP の `<CRITICAL_INSTRUCTION>` ブロック(下記「タスク管理」参照)

### `README.md`(外向け最小入口)

以下のみに留める。CLAUDE.md と内容を重複させない:

- 一言概要
- 公開URL(あれば)
- 主要ドキュメントへのリンク(`CLAUDE.md` / `docs/`)
- Backlog ボード(自動生成、下記参照)

### `docs/`(仕様の本体)

下記いずれかのパターンを採用する:

- **単一仕様パターン**(tobari 型): `docs/SPEC.md` 1 枚に集約。
- **分割パターン**(tanzaku 型): `docs/concept.md` / `docs/architecture.md` / `docs/features.md` / `docs/roadmap.md` / `docs/setup.md` などにトピック分割。

ブレインストーミング段階のメモは `memo.md` などをルート直下に残しても良い(souten がこのパターン)。

### `backlog/`(タスク管理)

Backlog.md のタスク・設定。`backlog init` で生成される。

## README.md の Backlog ボード自動生成

`README.md` に以下のマーカーを置くと、`backlog board export --readme` 実行時にマーカー間が自動更新される。**マーカー間は手で編集しない**:

```markdown
<!-- BOARD_START -->
<!-- BOARD_END -->
```

## タスク管理(Backlog.md MCP)

各アプリで Backlog.md MCP を使う。詳細な運用ルールは **アプリ側 `CLAUDE.md` 末尾の `<CRITICAL_INSTRUCTION>` ブロック** に置く(tobari の CLAUDE.md がリファレンス実装)。

そのブロックは MCP リソース `backlog://workflow/overview` を読むよう Claude に指示するもので、決定フレームワークと search-first ワークフローはそちらに集約されている。新規アプリ作成時は tobari のブロックをそのままコピーすれば良い。

## 記述言語

ドキュメント・コードコメント・タスク・コミットメッセージは原則日本語。
