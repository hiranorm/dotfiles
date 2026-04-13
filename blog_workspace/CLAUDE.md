# Blog Workspace — Claude Instructions

## 概要

このワークスペースは「ゆるディープ」ブログの更新作業用ディレクトリ。

- **ブログ名**: ゆるディープ（ゆるふわディープラーニングブログ）
- **著者**: ひらノルム
- **リポジトリ（旧）**: `blog_to_netlify/`（GitLab: `git@gitlab.com:shingetsu-hajime/blog_to_netlify.git`）
- **リポジトリ（新・移行先）**: `yurudeep/`（GitHub: https://github.com/hiranorm/yurudeep）
- **デプロイ**: GitLabへのpushで自動的にNetlifyへデプロイされる（移行後はGitHubから）

## リポジトリ構成

```
blog_to_netlify/
├── .vuepress/
│   ├── config.ts        # VuePressの設定（テーマ・navbar・コメント等）
│   ├── plugins/         # カスタムプラグイン（SEOなど）
│   ├── public/          # 静的ファイル（画像・favicon等）
│   └── styles/          # カスタムCSS
├── blogs/
│   ├── deeplearning/    # 機械学習・AI系記事
│   │   └── YYYY/YYYYMMDD.md
│   ├── automation/      # 自動化・ツール系記事
│   │   └── YYYY/YYYYMMDD.md
│   └── other/           # その他の記事
│       └── YYYY/YYYYMMDD.md
├── docs/
│   ├── guide/           # ガイドページ
│   └── profile/         # 自己紹介・プロフィールページ
├── dist/                # ビルド成果物（Netlifyにデプロイされる）
├── package.json
└── README.md            # トップページのフロントマター（VuePress形式）
```

## 技術スタック

- **フレームワーク**: VuePress 2.0.0-beta.60
- **テーマ**: vuepress-theme-reco 2.0.0-beta.53
- **バンドラー**: Vite
- **コメント**: Valine（LeanCloud）
- **言語設定**: 日本語（`ja-JP`）

## 記事のフォーマット

ファイル名: `blogs/{カテゴリ}/{YYYY}/{YYYYMMDD}.md`

```markdown
---
title: 記事タイトル
date: YYYY-MM-DD
tags:
 - タグ名
categories:
 - deeplearning  # または automation / other
prev: /blogs/{カテゴリ}/{YYYY}/{前の記事}.md
next: /blogs/{カテゴリ}/{YYYY}/{次の記事}.md
---

![トップ画像](/blogimg/...)

:::tip 要約
* ポイント1
* ポイント2
:::

<!-- more -->

::: warning 目次

[[TOC]]
:::

## 本文...
```

## ワークフロー

### ローカル開発
```bash
cd blog_to_netlify
yarn dev      # 開発サーバー起動（localhost:8080）
```

### ビルド & デプロイ
```bash
cd blog_to_netlify
yarn build    # dist/ にビルド
git add .
git commit -m "記事追加 or 更新内容"
git push      # GitLab pushで自動的にNetlifyへデプロイ
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

## 注意事項

- `dist/` はビルド成果物のため直接編集しない
- 記事の `prev`/`next` リンクは手動で設定が必要
- 画像は `.vuepress/public/blogimg/` 以下に配置
- `node_modules/` は変更しない
