# kaggle_workspace: AI実験プロジェクト構成

## ディレクトリ構成

- `gdrive/` — Google Drive のシンボリックリンク
  - 実体: `~/Library/CloudStorage/GoogleDrive-hira.euclid.norm.root2@gmail.com`（Drive ルート）
  - マイドライブはこの下にある（`gdrive/マイドライブ/`）
  - Google Drive for Desktop が起動していれば自動でアクセス可能
- **作業ディレクトリ: `gdrive/マイドライブ/kaggle_experiments/`**
  - 競合ごとのディレクトリはここに置かれる
  - フルパス: `~/kaggle_workspace/gdrive/マイドライブ/kaggle_experiments/{competition-name}/`

### 新規コンペは `competition-template/` からコピーして始める

`gdrive/マイドライブ/kaggle_experiments/competition-template/` に雛形がある。新規コンペは
`cp -r competition-template {competition-name}` で複製し、`competition-template/SETUP.md` の手順に従う。
以下のディレクトリ構成・命名規則は **2026-05-27 以降の新規コンペ向け**（旧コンペは末尾の互換性注記を参照）。

```
gdrive/マイドライブ/kaggle_experiments/
└── {competition-name}/
    ├── SETUP.md                # 立ち上げ手順
    ├── RESULTS.md              # 実験結果履歴（CV・LB・考察）※コンペ直下
    ├── GUARDRAILS.md           # ダメだったこと（LB を下げたパターン）※コンペ直下
    ├── IDEAS.md                # 実験アイデア（着手前の brainstorm）※コンペ直下
    ├── research/               # コンペ・技術調査資料
    │   ├── competition_overview.md
    │   ├── past_winners.md
    │   └── sota_models.md
    ├── exp/
    │   ├── exp001/             # 実験コード一式（全小文字）
    │   │   ├── train.py        # 学習スクリプト（Colab で実行）
    │   │   ├── infer.py        # 推論スクリプト（Kaggle Notebooks で実行）
    │   │   ├── dataset.py
    │   │   ├── models.py
    │   │   └── config/
    │   │       ├── exp001-001.yaml  # パラメータのみ変える軽量実験
    │   │       └── exp001-002.yaml
    │   └── exp002/             # アーキテクチャ変更など大きな実験
    │       ├── train.py
    │       └── infer.py
    ├── data/
    │   ├── inputs/              # 入力データ（画像等）
    │   └── outputs/             # 実験結果（ログ、モデル重み）
    │       └── exp001-001_YYYY-MM-DD/
    ├── notebooks/
    │   ├── eda.ipynb            # 探索的データ分析
    │   ├── train.ipynb          # 学習ランナー（パラメータを渡して exp/ を呼ぶだけ）
    │   └── infer.ipynb          # 推論・サブミット用
    ├── backlog/                 # タスク管理（backlog init で生成、A スタイル）
    └── scripts/                 # ユーティリティスクリプト
```

## 実験管理システム（exp{NNN}-{MMM}）

### 命名規則（重要）

- 実験 ID は **全小文字** `exp{NNN}-{MMM}`（例: `exp001-001`）。
  - 親 = `exp001`（コード一式 `exp/exp001/`）、子 = `-001`（config `config/exp001-001.yaml`）。
- **大文字を使わない理由**: Kaggle 上でモデルのパス名が小文字化され、パス不一致のバグが出るため。
  ディレクトリ名・config 名・config 内の `exp_id`・Kaggle Dataset 名すべて小文字で統一する。
- 出力は `data/outputs/{exp_id}_{YYYY-MM-DD}/`（例: `data/outputs/exp001-001_2026-06-01/`）。

### 大きな実験（コード変更あり）

アーキテクチャ・データ処理・損失関数などのコード変更を伴う場合：
1. 新しい実験ディレクトリ `exp/exp{N}/` を作成する
2. `train.py` と `infer.py` を必ず両方含める
3. 親番号は連番でインクリメント（exp001, exp002, exp003, ...）

**新 exp が必要なシナリオ例：**
- モデルアーキテクチャの変更
- データ拡張パイプラインの変更
- 損失関数の変更
- 学習戦略の変更（k-fold の分割方法など）

### 小さな実験（パラメータ変更のみ）

ハイパーパラメータのチューニングのみの場合：
1. 同じ `exp/exp{N}/train.py` を使い続ける
2. 新しい config ファイル `exp/exp{N}/config/exp{N}-{M}.yaml` を作成する
3. 子番号は連番でインクリメント（001, 002, 003, ...）

**子 config が適切なシナリオ例：**
- 学習率・バッチサイズ・エポック数の変更
- データ拡張の確率変更
- 損失の重み変更
- モデルのハイパーパラメータ（dropout, hidden_dim など）

### 指示の例

```
exp001 の下に exp001-003 を作成して、loss を SmoothL1 に変更して
```

## 作業管理ルール

コンペごとに以下のいずれかのスタイルで管理する。

### A. Backlog.md スタイル（**birdclef-2026 で運用中、2026-05-11 〜**）

birdclef-2026 のみ採用。タスク管理を `backlog/` ディレクトリで Kanban 形式に切り出す。設計決定とアイディアの肥大化を防ぐのが目的。

**backlog CLI のインストール（未インストール時）:**

```bash
# 確認
which backlog && backlog --version

# 未インストールなら npm（volta 経由）でグローバルインストール
npm install -g backlog.md

# bun が入っている環境では bun の方が公式推奨
bun install -g backlog.md
```

インストール後、`/Users/hiranot/.volta/bin/backlog` 等が PATH に入っていることを確認。新規コンペで Backlog.md スタイルを採用する場合は以下のコマンドで初期化:

```bash
cd {competition-name}
backlog init "{competition-name}" \
  --no-git \
  --defaults \
  --task-prefix TASK \
  --zero-padded-ids 3 \
  --agent-instructions claude \
  --integration-mode cli \
  --backlog-dir backlog

# 初期化後、backlog/config.yml の statuses と labels を編集:
#   statuses: ["To Do", "In Progress", "Done", "Frozen"]
#   labels:   ["exp001", "exp002", ..., "ensemble", "infer-only", "infra", "data", "research"]
```

**重要:** `backlog init` は `{competition-name}/CLAUDE.md` に backlog CLI の詳細インストラクション（約 31KB）を生成する。これは project-scoped 指示として AI が読むので消さない。

ディレクトリ構成（コンペ直下）:
```
{competition-name}/
├── backlog/
│   ├── tasks/        # TASK-001 等のアクティブ/凍結タスク
│   ├── decisions/    # decision-001 等の設計決定 (ADR)
│   ├── docs/         # backlog 用ドキュメント
│   ├── drafts/       # コミット前のアイディア
│   ├── milestones/   # マイルストーン
│   └── completed/    # 完了タスクのアーカイブ先
├── RESULTS.md        # 実験結果と考察の履歴（コンペ直下）
├── GUARDRAILS.md     # ダメだったこと（コンペ直下）
├── IDEAS.md          # 実験アイデア（コンペ直下、着手可になったら task 化）
└── (MEMORY.md は廃止)
```

`IDEAS.md` は着手前の brainstorm プール、backlog の `tasks/` は着手可能になったアクション。
アイデアが実行段階に入ったら `backlog task create` で task に昇格させる。

**AI への指示（Backlog.md スタイル時）:**

会話の開始時:
1. `backlog task list --plain` で現在のタスク状況を確認（`In Progress` → `To Do` 優先順）
2. 必要なら `backlog task TASK-N --plain` で詳細を読む
3. ユーザーから「TASK-N をやって」「次の P1 を進めて」のような指示が来たら該当タスクで作業を開始する

設計決定を下したとき:
- `backlog decision create "決定タイトル"` でスケルトン作成 → ファイルを直接編集して `## Context` / `## Decision` / `## Consequences` を埋める
- date は決定した日付（auto-fill ではなく実際の決定日）

タスクを進めたとき:
- `backlog task edit TASK-N -s "In Progress"` で着手をマーク
- 完了したら `backlog task edit TASK-N -s Done`
- 凍結（再着手しない判断）したら `backlog task edit TASK-N -s Frozen`
- ユーザーから新しい作業依頼が来たら `backlog task create "..."` で新規タスク作成

新規タスク作成時の規約:
- `--priority high/medium/low`（high = 直近着手、low = wishlist）
- `--labels` は `config.yml` の labels から選ぶ（exp001, exp002, exp003, exp004, ensemble, infer-only, infra, data, research）
- `--ac` で受け入れ条件を 2〜4 個書く（後で `backlog task edit TASK-N --check-ac` でチェック可能）
- 依存関係があれば `--dep TASK-M`

### B. MEMORY.md スタイル（**nemotron / orbit-wars / arc-prize-2026 等、従来運用**）

**目的:** 会話をまたいで現在地・次アクション・設計決定を引き継ぐ作業メモ。
`EXP_SUMMARY.md`（実験スコアの記録）とは別物。

**AIへの指示:**

会話の開始時:
1. `MEMORY.md` を読んで現在地と次アクションを把握する
2. 作業後は「次のアクション」と「完了済み」を更新する

設計決定を下したとき:
- 「なぜその選択をしたか」を「主要な設計決定」テーブルに追記する

作業が完了したとき:
- 対応するチェックボックスを `[x]` に変更し、新たな未完了タスクを追記する

---

## 記録ファイル（RESULTS / GUARDRAILS / IDEAS）の運用ルール

新規コンペでは EXP_SUMMARY.md を 3 ファイルに分割し、**コンペ直下**に置く。
（旧コンペは引き続き `EXP/EXP_SUMMARY.md` 1ファイル運用 — 互換性注記を参照）

| ファイル | 役割 |
|----------|------|
| `RESULTS.md` | 実験結果履歴（CV・LB・fold スコア、考察、モデル比較テーブル） |
| `GUARDRAILS.md` | ダメだったこと（LB を下げた / 効かなかったパターンと次回の指針） |
| `IDEAS.md` | 実験アイデア（着手前の brainstorm。着手可になったら backlog task 化） |

**重要：AIへの指示**

### ユーザーが LB スコアを報告したとき
1. **即座に `RESULTS.md` を読む**
2. 以下を更新する：
   - 該当実験に LB スコアを追記
   - CV-LB ギャップを更新
   - 改善・悪化の考察を追記、モデル比較テーブルを更新
3. 新ベストなら Competition Status セクションも更新
4. LB を下げた原因が判明したら `GUARDRAILS.md` にも 1 項目として転記する

### ユーザーが新しいアイディアを求めたとき
1. **まず `RESULTS.md` と `GUARDRAILS.md` を読む**（何を試したか、何が効いたか / 効かなかったか）
2. 試行済みの手法・GUARDRAILS にある失敗パターンを再提案しない
3. 採用するアイデアは `IDEAS.md` に追記（着手するなら backlog task に昇格）

### ドキュメント基準
- **成功した実験**：CV・LB・fold スコア、主な考察、前実験との比較 → `RESULTS.md`
- **失敗・中断した実験**：中断理由（遅すぎる・CV 改善なしなど）、得られた知見 → `RESULTS.md`、
  LB を下げたパターンは `GUARDRAILS.md` にも転記

## 開発ワークフロー

| 環境 | 用途 |
|------|------|
| **ローカル** | コード編集、記録ファイル（RESULTS.md 等）更新、実験管理 |
| **Google Colab** | 学習スクリプトの実行（GPU が必要） |
| **Kaggle Notebooks** | 推論・サブミット（≤9h, インターネット不可） |

**Claude Code はローカルで学習スクリプトを実行しない。**

### Google Drive との連携（Colab GPU 使用時）
- モデル重みや結果は Google Drive に保存する
- `.ipynb` ファイルは Google Drive 上で管理し、Colab とローカル JupyterLab 両方から開ける
- ローカルからは `gdrive/` 経由でアクセスできる

## コーディング方針

- **ロジックは .py に書く**。ノートブックはパラメータを渡して呼び出すだけにする
- `train.ipynb` は実験ごとに複製しない。`exp{N}-{M}.yaml` でパラメータを切り替える
- 実験結果は `data/outputs/{exp_id}_{YYYY-MM-DD}/`（例 `exp001-001_2026-06-01`）に保存する

### CRITICAL: 後方互換性の維持

既存の `train.py` に新機能を追加するとき、**必ず後方互換性を維持する**。

**ルール：**
1. 新しい config 項目にはデフォルト値を設定する（例：`feature_enabled: false`）
2. Dataset の戻り値に要素を追加する場合、`None` を返さない（DataLoader がエラーになる）
   - 代わりにダミー tensor を返すか、条件分岐で要素数を変える
3. モデルの `forward` 引数を変える場合、デフォルト値を設定する（例：`def forward(self, x, depth=None)`）

**悪い例：**
```python
# depth が None → DataLoader がエラーになる
return image, targets, depth_tensor  # depth_tensor=None はNG
```

**良い例：**
```python
if self.depth_dir is not None:
    depth_tensor = load_depth(...)
else:
    depth_tensor = torch.zeros(1, 32, 32)  # ダミー
return image, targets, depth_tensor
```

## CRITICAL: ガードレール（やってはいけないこと）

実験履歴から学んだ「LB を下げる」パターンを随時ここに追記する。

<!-- 例：
- 強すぎるデータ拡張（デフォルト aug が最も安全）
- OOF 最適化アンサンブル（OOF では改善するが LB に転移しないことが多い）
- Hand Labeling によるノイズクリーニング（過学習パターン）
-->

## コンペ調査（research ディレクトリ）

各コンペの調査資料は `{competition-name}/research/` に保存する。

```
{competition-name}/research/
├── competition_overview.md  # コンペ概要・データ・評価指標・タイムライン
├── past_winners.md          # 過去の優勝アプローチ・有効手法・無効手法
└── sota_models.md           # SOTAモデル・論文リファレンス
```

**調査フロー：**
1. Kaggle CLI でコンペ基本情報を取得
2. WebSearch / WebFetch でコンペ詳細ページ・Discussionを調査
3. Papers with Code（paperswithcode.com）で学術SOTAを調査
4. arXiv / GitHub で過去の優勝解法・関連論文を調査
5. `research/` 以下に上記3ファイルとして保存

**有力な調査ソース：**
- `~/.local/bin/kaggle competitions list --category research --csv` — コンペ基本情報
- [paperswithcode.com](https://paperswithcode.com) — SOTA手法・ベンチマーク（アクセス可）
- [arxiv.org](https://arxiv.org) — 論文
- CLEF working notes（年次論文集） — BirdCLEF系の解法論文
- Kaggle Discussion（WebFetch不可; WebSearchで検索）
- GitHub（優勝者のリポジトリ）

**注意：**
- Kaggle 公式ページは JS レンダリングが必要なため WebFetch では取得不可
- Papers with Code は WebFetch でアクセス可能

## コンペ情報の収集

開催中のメダル付与コンペ一覧は `gdrive/kaggle_experiments/competition_list/` に日付つきファイル名（`YYYY-MM-DD_active-medal-competitions.md`）で保存する。

**収集フロー：**
1. Kaggle CLI で各カテゴリを取得する：
   ```bash
   ~/.local/bin/kaggle competitions list --category featured --csv
   ~/.local/bin/kaggle competitions list --category research --csv
   ```
2. 締め切りが今日以降のものに絞る
3. `gdrive/kaggle_experiments/competition_list/YYYY-MM-DD_active-medal-competitions.md` に保存

**注意：**
- Kaggle 認証情報は `~/.kaggle/kaggle.json` にある
- CLI は `~/.local/bin/kaggle`（pipx でインストール済み）
- メダル付与対象は Featured・Research カテゴリ（masters/playground は対象外）
- Kaggle 公式ページは JS レンダリングが必要なため WebFetch では取得不可

## AIへの依頼時の注意

- コード修正の依頼は `.py` ファイル単位で行う
- ノートブックの JSON をそのまま渡さない
- 実験パラメータの変更は `exp{N}-{M}.yaml` を対象にする
- 新規実験の作成を依頼するときは「exp{N} の下に exp{N}-{M} を作って〜」と指定する

## 互換性注記（旧コンペの運用）

`nemotron` / `orbit-wars` / `arc-prize-2026` 等、**2026-05-27 より前**に立ち上げたコンペは
旧運用のまま：
- 実験ディレクトリは大文字 `EXP/EXP000/`、子 config は `child-exp{M}.yaml`
- 実験記録は `EXP/EXP_SUMMARY.md`（1ファイル）、作業メモは `MEMORY.md`

新規コンペ（`competition-template/` 由来）のみ `exp/exp001/` + `exp001-001.yaml` +
`RESULTS.md` / `GUARDRAILS.md` / `IDEAS.md` を使う。混同しないこと。
