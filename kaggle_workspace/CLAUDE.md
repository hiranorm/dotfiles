# kaggle_workspace: AI実験プロジェクト構成

## ディレクトリ構成

- `gdrive/` — Google Drive（マイドライブ）のシンボリックリンク
  - 実体: `~/Library/CloudStorage/GoogleDrive-hira.euclid.norm.root2@gmail.com/マイドライブ`
  - Google Drive for Desktop が起動していれば自動でアクセス可能
- **作業ディレクトリ: `gdrive/kaggle_experiments/`**
  - 競合ごとのディレクトリはここに置かれる
  - フルパス: `~/kaggle_workspace/gdrive/kaggle_experiments/{competition-name}/`

```
gdrive/kaggle_experiments/
└── {competition-name}/
    ├── research/                # コンペ・技術調査資料
    │   ├── competition_overview.md
    │   ├── past_winners.md
    │   └── sota_models.md
    ├── MEMORY.md                # 現在地・次アクション・設計決定（会話をまたぐ作業メモ）
    ├── EXP/
    │   ├── EXP_SUMMARY.md       # 実験履歴の一元管理（AIの記憶補助）
    │   ├── EXP000/              # ベースライン実験
    │   │   ├── train.py         # 学習スクリプト（Colab で実行）
    │   │   ├── infer.py         # 推論スクリプト（Kaggle Notebooks で実行）
    │   │   └── config/
    │   │       ├── child-exp000.yaml  # パラメータのみ変える軽量実験
    │   │       └── child-exp001.yaml
    │   └── EXP001/              # アーキテクチャ変更など大きな実験
    │       ├── train.py
    │       └── infer.py
    ├── data/
    │   ├── inputs/              # 入力データ（画像等）
    │   └── outputs/             # 実験結果（ログ、モデル重み）
    ├── notebooks/
    │   ├── eda.ipynb            # 探索的データ分析
    │   ├── train.ipynb          # 学習ランナー（パラメータを渡してEXP/を呼ぶだけ）
    │   └── infer.ipynb          # 推論・サブミット用
    └── scripts/                 # ユーティリティスクリプト
```

## 実験管理システム（EXP + child-exp）

### 大きな実験（コード変更あり）

アーキテクチャ・データ処理・損失関数などのコード変更を伴う場合：
1. 新しい実験ディレクトリ `EXP/EXP{N}/` を作成する
2. `train.py` と `infer.py` を必ず両方含める
3. `exp_no` は連番でインクリメント（EXP000, EXP001, EXP002, ...）

**新 EXP が必要なシナリオ例：**
- モデルアーキテクチャの変更
- データ拡張パイプラインの変更
- 損失関数の変更
- 学習戦略の変更（k-fold の分割方法など）

### 小さな実験（パラメータ変更のみ）

ハイパーパラメータのチューニングのみの場合：
1. 同じ `EXP{N}/train.py` を使い続ける
2. 新しい config ファイル `EXP/EXP{N}/config/child-exp{M}.yaml` を作成する
3. `child_no` は連番でインクリメント（000, 001, 002, ...）

**child-exp が適切なシナリオ例：**
- 学習率・バッチサイズ・エポック数の変更
- データ拡張の確率変更
- 損失の重み変更
- モデルのハイパーパラメータ（dropout, hidden_dim など）

### 指示の例

```
EXP113 の下に child-exp003 を作成して、loss を SmoothL1 に変更して
```

## MEMORY.md の運用ルール

**目的:** 会話をまたいで現在地・次アクション・設計決定を引き継ぐ作業メモ。
`EXP_SUMMARY.md`（実験スコアの記録）とは別物。

**AIへの指示:**

### 会話の開始時
1. `MEMORY.md` を読んで現在地と次アクションを把握する
2. 作業後は「次のアクション」と「完了済み」を更新する

### 設計決定を下したとき
- 「なぜその選択をしたか」を「主要な設計決定」テーブルに追記する

### 作業が完了したとき
- 対応するチェックボックスを `[x]` に変更し、新たな未完了タスクを追記する

---

## EXP_SUMMARY.md の運用ルール

**重要：AIへの指示**

### ユーザーが LB スコアを報告したとき
1. **即座に `EXP/EXP_SUMMARY.md` を読む**
2. 以下を更新する：
   - 該当実験に LB スコアを追記
   - LB-CV ギャップを更新
   - 改善・悪化の考察を追記
   - モデル比較テーブルを更新
3. 新ベストなら Competition Status セクションも更新

### ユーザーが新しいアイディアを求めたとき
1. **まず `EXP/EXP_SUMMARY.md` を読む**（何を試したか、何が効いたか）
2. 試行済みの手法を再提案しない
3. 失敗した実験の教訓を踏まえて提案する

### ドキュメント基準
- **成功した実験**：CV・LB・fold スコア、主な考察、前実験との比較
- **失敗・中断した実験**：中断理由（遅すぎる・CV 改善なしなど）、得られた知見

## 開発ワークフロー

| 環境 | 用途 |
|------|------|
| **ローカル** | コード編集、EXP_SUMMARY.md 更新、実験管理 |
| **Google Colab** | 学習スクリプトの実行（GPU が必要） |
| **Kaggle Notebooks** | 推論・サブミット（≤9h, インターネット不可） |

**Claude Code はローカルで学習スクリプトを実行しない。**

### Google Drive との連携（Colab GPU 使用時）
- モデル重みや結果は Google Drive に保存する
- `.ipynb` ファイルは Google Drive 上で管理し、Colab とローカル JupyterLab 両方から開ける
- ローカルからは `gdrive/` 経由でアクセスできる

## コーディング方針

- **ロジックは .py に書く**。ノートブックはパラメータを渡して呼び出すだけにする
- `train.ipynb` は実験ごとに複製しない。`child-exp{N}.yaml` でパラメータを切り替える
- 実験結果は `data/outputs/<YYYY-MM-DD_exp-name>/` に保存する

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
- 実験パラメータの変更は `child-exp{N}.yaml` を対象にする
- 新規実験の作成を依頼するときは「EXP{N} の下に child-exp を作って〜」と指定する
