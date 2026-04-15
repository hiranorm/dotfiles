# Blog Workspace — Claude Instructions

## 概要

このワークスペースは「ゆるディープ」ブログの更新作業用ディレクトリ。

- **ブログ名**: ゆるディープ（ゆるふわディープラーニングブログ）
- **著者**: ひらノルム
- **リポジトリ**: `yurudeep/`（GitHub: https://github.com/hiranorm/yurudeep）
- **デプロイ**: GitHubへのpushで自動的にNetlifyへデプロイされる


## リポジトリ構成

```
yurudeep/
├── src/
│   ├── content/
│   │   └── posts/           # 記事ファイル
│   │       ├── deeplearning/
│   │       │   └── YYYY/YYYYMMDD.md
│   │       ├── automation/
│   │       │   └── YYYY/YYYYMMDD.md
│   │       └── other/
│   │           └── YYYY/YYYYMMDD.md
│   ├── components/
│   ├── layouts/
│   ├── pages/
│   └── config.ts            # サイト設定
├── public/
│   ├── blogimg/             # 記事用画像
│   └── favicon/
├── dist/                    # ビルド成果物（Netlifyにデプロイされる）
├── package.json
└── astro.config.mjs
```

## 技術スタック

- **フレームワーク**: Astro
- **テーマ**: Fuwari
- **パッケージマネージャ**: pnpm
- **言語設定**: 日本語

## 記事のフォーマット

ファイル名: `src/content/posts/{カテゴリ}/{YYYY}/{YYYYMMDD}.md`

```markdown
---
title: 記事タイトル
published: YYYY-MM-DD
description: 記事の説明文（一文程度）
image: （画像URL またはパス）
tags: [タグ1, タグ2]
category: deeplearning  # または automation / other
draft: false
---

:::tip
* ポイント1
* ポイント2
:::

## 本文...
```

## ワークフロー

### ローカル開発
```bash
cd yurudeep
pnpm dev      # 開発サーバー起動
```

### ビルド & デプロイ
```bash
cd yurudeep
pnpm build    # dist/ にビルド
git add .
git commit -m "記事追加 or 更新内容"
git push      # GitHub pushで自動的にNetlifyへデプロイ
```

### 新規記事作成
```bash
cd yurudeep
pnpm new-post   # 対話形式で新記事ファイルを生成
```

## カテゴリ一覧

| ディレクトリ | 内容 |
|---|---|
| `deeplearning` | 機械学習・AI・kaggle等の技術記事 |
| `automation` | 自動化・ツール・スクリプト系 |
| `other` | その他（雑記・小説・趣味等） |

## Google Driveネタ管理

`gdrive/` はGoogle Driveをローカルにマウントしたディレクトリへのシンボリックリンク（またはパス）。

- **ネタ置き場**: `gdrive/my-thinking/` — 記事のネタになる思考整理ノート
- **使用済み**: `gdrive/my-thinking/old/` — 記事化が完了したネタはここへ移動
- ネタを記事にし終わったら、元ファイルを `old/` へ移動するのがルール

## 追加ルール

詳細なルールは `.claude/rules/` 以下のファイルを参照すること。

- `.claude/rules/reference.md` — 調査資料をまとめる際の引用スタイル（番号付き引用形式）

### Claudeとの共同編集表示

記事をClaudeと共同で執筆・編集した場合、冒頭の `:::tip` 要約ブロックの前に以下のブロックを挿入すること。

```markdown
:::tip この記事について
Claude（Anthropic）との共同編集により作成されました。
:::
```

## 注意事項

- `dist/` はビルド成果物のため直接編集しない
- 画像は `public/blogimg/` 以下に配置
- `node_modules/` は変更しない
