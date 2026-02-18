# Zenn CLI

Zenn の投稿コンテンツをローカルで管理・プレビューするためのリポジトリです。

## セットアップ

### 前提条件

- Node.js 14 以上
- [Zenn と GitHub リポジトリの連携](https://zenn.dev/zenn/articles/connect-to-github)が完了していること

### インストール

```bash
npm install
```

### CLI のアップデート

```bash
npm install zenn-cli@latest
```

## 使い方

### 記事の作成

```bash
npx zenn new:article
```

オプションでスラッグやタイトルを指定可能：

```bash
npx zenn new:article --slug my-article --title "記事タイトル" --type tech --emoji ✨
```

### プレビュー

```bash
npx zenn preview
```

`http://localhost:8000` でプレビューが開きます。ポート変更は `--port 3000` で指定できます。

### 記事の公開

1. Front Matter の `published` を `true` に設定
2. 変更をコミットして GitHub に push
3. 連携ブランチへの push をトリガーにデプロイが実行される

予約投稿をする場合は `published_at` を指定：

```yaml
published: true
published_at: 2050-06-12 09:03
```

### 本の作成

```bash
npx zenn new:book
```

## ディレクトリ構成

```
.
├── articles/          # 記事の Markdown ファイル
│   └── <slug>.md
├── books/             # 本のディレクトリ
│   └── <book-slug>/
│       ├── config.yaml
│       ├── cover.png
│       └── <chapter>.md
└── package.json
```

### 記事の Front Matter

```yaml
---
title: "記事タイトル"
emoji: "😸"
type: "tech"            # tech: 技術記事 / idea: アイデア
topics: ["Go", "AWS"]   # タグ（最大5つ）
published: true
published_at: "2024-07-07 01:37"  # 予約投稿（任意）
publication_name: "stafes_blog"   # Publication（任意）
---
```

### 本の config.yaml

```yaml
title: "本のタイトル"
summary: "概要"
topics: ["Go"]          # タグ（最大5つ）
published: true
price: 0                # 0（無料）または 200〜5000（100円単位）
toc_depth: 0            # 目次の見出し深度（0〜3）
chapters:               # チャプターの順序
  - intro
  - setup
  - usage
```

## 画像の挿入

- [Zenn のダッシュボード](https://zenn.dev/dashboard/uploader)からアップロード
- GitHub リポジトリ内に配置（`/images` ディレクトリなど）
- 外部サービス（Gyazo 等）を利用

## 注意事項

- 記事・本の削除は[ダッシュボード](https://zenn.dev/dashboard)からのみ可能（ローカルファイルの削除では公開済みコンテンツは消えない）
- コミットメッセージに `[ci skip]` を含めると Zenn への同期をスキップできる
- slug は `a-z0-9`、ハイフン、アンダースコアで 12〜50 文字

## 参考リンク

- [📘 CLI の使い方](https://zenn.dev/zenn/articles/zenn-cli-guide)
- [📦 CLI のインストール](https://zenn.dev/zenn/articles/install-zenn-cli)
- [🔗 GitHub 連携](https://zenn.dev/zenn/articles/connect-to-github)