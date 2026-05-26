# Zenn Content - 執筆ガイドライン

## プロジェクト概要

Zenn (zenn.dev) で技術記事を公開するためのコンテンツリポジトリ。
zenn-cli + GitHub連携で記事を管理する。

## ディレクトリ構成

```
articles/        → Zenn記事（slug.md）
books/           → Zenn本（章立て）
images/          → 記事用画像（articles/slug/ に配置）
templates/       → 記事テンプレート
reviews/         → レビュー記録（記事slug/review-XX.md）
```

## 記事フロントマター仕様

```yaml
---
title: "タイトル"        # 必須
emoji: "🔥"             # 必須（1文字）
type: "tech"            # 必須: "tech" or "idea"
topics: ["tag1"]        # 必須: 最大5つ
published: false        # 必須: true/false
published_at: ""        # 任意: YYYY-MM-DD HH:mm
---
```

### slug（ファイル名）の制約

- 半角英小文字 (a-z)、数字 (0-9)、ハイフン (-)、アンダースコア (_)
- 12〜50文字

## 文体ルール

- **です/ます調**で統一する
- 技術用語は正式名称を使う（例: JavaScript, TypeScript, Node.js）
- 一文は60文字以内を目安に短く保つ
- 主語と述語を近づけ、修飾語は被修飾語の直前に置く

## 記事構成パターン

```markdown
## はじめに
- 対象読者の明示
- この記事で得られること

## 本題（複数セクション）
- 1セクション1テーマ
- コード例は「最小で動く」ものを示す
- 適宜 :::message や :::details を活用

## まとめ
- 要点の箇条書き
- 次に学ぶべきこと/関連記事への導線
```

## Zenn固有の記法

```markdown
:::message
補足情報・注意点
:::

:::message alert
警告・注意
:::

:::details タイトル
折りたたみコンテンツ
:::

```mermaid
graph LR
A --> B
```　
```

## レビューワークフロー

1. 記事を書いたら `/review-article` でレビューを依頼（100点満点・4観点で評価）
2. レビュー結果は `reviews/{slug}/review-{連番}.md` に保存される
3. `/fix-article` でレビュー指摘に沿って記事を修正（形式面は自動、内容面は確認後に適用）
4. 再度 `/review-article` で再評価し、改善を重ねる
5. 公開前に `/pre-publish-check` で最終チェック

## Gitフロー

- 記事ごとにブランチを切る: `article/{slug}`
- 公開時に main へマージ（published: true に変更）
- コミットメッセージ: `feat(article): {slug} - {概要}`

## プレビュー

```bash
npx zenn preview
```

localhost:8000 でプレビュー確認可能。
