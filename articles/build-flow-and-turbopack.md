---
title: "フロントエンドのビルドはどう動く？仕組みとTurbopackをわかりやすく解説"
emoji: "⚡"
type: "tech"
topics: ["webpack", "turbopack", "nextjs", "frontend", "build"]
published: true
---

## はじめに

「`npm run build` を実行するだけで本番用のファイルが生成される」——フロントエンド開発では当たり前の操作ですが、その裏側でどのような処理が行われているか、正確に説明できますか？

この記事では以下を解説します。

- フロントエンドのビルドとは何か
- ビルドの実行フロー（変換・バンドル・最適化）
- 従来のWebpackの仕組みと課題
- 次世代バンドラー Turbopackとは何か、何が変わるのか

対象読者は「ビルドツールを使っているが仕組みはよく知らない」という方です。

## ビルドとは何か

ビルドとは、**開発用のソースコード（TypeScript、JSX、CSSなど）を、ブラウザが実行できる形式に変換する処理**です。

現代のフロントエンド開発では、ブラウザがそのままでは解釈できないものを多用します。

- TypeScript → JavaScriptへのトランスパイル
- JSX → `React.createElement()` への変換
- CSS Modules / Tailwind → 通常のCSSへの変換
- `import` 文 → ブラウザ向けのバンドル

これらの変換をまとめて行うのがビルドツールの役割です。

## ビルドの実行フロー

ビルドは大きく3つのフェーズで構成されます。

### 1. 解析（Parse）・変換（Transform）

まずソースファイルを**AST（抽象構文木）**に変換し、各種トランスパイルを行います。

```text
TypeScript (.tsx) → AST解析 → JavaScript (ES5/ES2020など) に変換
JSX → React.createElement() に変換
CSS Modules → スコープ付きクラス名に変換
```

この変換を担うのが **Babel**（または高速な代替として **SWC**）などのトランスパイラです。

### 2. バンドル（Bundle）

複数の `import` で結ばれたファイルを、**1つ（または少数）のJSファイルにまとめる**処理です。

```text
index.tsx
  └─ import Button from './Button'
       └─ import styles from './Button.module.css'
```

依存関係グラフを解析し、必要なファイルをすべて含んだバンドルを生成します。

:::message
バンドルすることでHTTPリクエスト数を削減し、ページの読み込み速度を改善できます。
:::

### 3. 最適化（Optimize）

最後に本番向けの最適化を行います。

- **Tree Shaking**: 使われていないコードを除去する
- **Minify（圧縮）**: 変数名の短縮、空白削除などでファイルサイズを削減する
- **Code Splitting**: 初期ロードに不要なコードを分割し、遅延読み込みする
- **ハッシュ付きファイル名**: `main.a3f2c1.js` のようにキャッシュ制御を最適化する

## Webpackの仕組みと課題

長らくフロントエンドの標準だったのが **Webpack** です。

### Webpackの動作

Webpackはエントリーポイントから始まり、すべての依存ファイルを再帰的に解析してバンドルを生成します。

```mermaid
graph LR
  A[index.tsx] --> B[解析]
  B --> C[依存グラフ構築]
  C --> D[変換・バンドル]
  D --> E[最適化]
  E --> F[dist/main.js]
```

### Webpackの課題

Webpackは非常に高機能ですが、大規模プロジェクトで課題が顕在化します。

| 課題 | 詳細 |
|------|------|
| 起動の遅さ | 初回起動時にすべてのファイルを解析するため、数十秒かかることも |
| HMRの遅延 | ファイル変更時にも関連モジュールを再バンドルするため、更新が遅い |
| メモリ消費 | 大規模プロジェクトでのメモリ使用量が高い |

## Turbopackとは

**Turbopack**はVercelが開発した次世代のJavaScriptバンドラーです。Next.js 13以降で段階的に採用されています。

### Turbopackの特徴

#### 1. Rustで実装されている

WebpackはJavaScriptで実装されていますが、TurbopackはRustで書かれています。Rustは実行速度が非常に高速なため、ビルド処理そのものが高速になります。

#### 2. 差分ビルド（Incremental Computation）

Turbopackの最大の特徴は**変更されたファイルだけを再ビルドする**仕組みです。

```text
ファイルA を変更
 ↓
Aに依存するモジュールだけを再計算
 ↓
変更のないBやCは前回の結果をキャッシュから再利用
```

:::message
Webpackは変更があると関連するすべてのファイルを再バンドルしますが、Turbopackはキャッシュを活用して最小限の再計算にとどめます。
:::

#### 3. 並列処理

Rustのマルチスレッド機能を活用し、複数のファイルを並列で処理します。

### WebpackとTurbopackの比較

| 項目 | Webpack | Turbopack |
|------|---------|-----------|
| 実装言語 | JavaScript | Rust |
| 初回起動 | 遅い（全ファイル解析） | 速い（必要なものだけ） |
| HMR（ホットリロード） | 遅延が生じることがある | ミリ秒単位で更新 |
| 成熟度 | 非常に高い（エコシステムが豊富） | 安定版（Next.js 15でデフォルト化） |
| 設定の柔軟性 | 非常に高い | Next.js向けに特化 |

### Next.jsでTurbopackを使う

Next.js 14以降では、`--turbopack`フラグを付けることでTurbopackを利用できます。

```bash
# --turbopack フラグを指定（Next.js 14以降）
next dev --turbopack
```

Next.js 15からは `next dev` でTurbopackがデフォルトになっています。

:::message alert
`next build`（本番ビルド）でのTurbopack対応はNext.js 15時点でも実験的段階です。本番ビルドには引き続きWebpackが使用されます。
:::

## まとめ

- ビルドとは「ソースコードをブラウザが実行できる形に変換する処理」
- フローは**解析→バンドル→最適化**の3ステップ
- Webpackは高機能だが、大規模プロジェクトでは起動・HMRの遅さが課題
- TurbopackはRust製で差分ビルドを採用し、開発体験を大幅に改善
- Next.js 15からは開発サーバーでTurbopackがデフォルト
