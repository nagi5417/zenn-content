---
title: "ビルドの裏側をNext.jsで理解する"
emoji: "🏗️"
type: "tech"
topics: ["frontend", "nextjs", "react", "webpack", "javascript"]
published: true
---

## はじめに

フロントエンド開発を始めると「ビルド」という言葉に必ず出会います。
`npm run build` を実行したことはあっても、その裏で何が起きているか気になったことはないでしょうか。

この記事では、以下のことが分かります。

- ビルドとは何か（なぜ必要なのか）
- ビルドで何が変換・最適化されるのか
- Next.js のビルドで実際に何が起きるのか

対象読者: フロントエンド開発を始めたばかりの方、ビルド機能の中身が気になっている方

## ビルドとは何か

ビルドとは、「人間が書いたコード」を「ブラウザが実行できる形」に変換するプロセスのことです。


## ビルドで何が行われるのか
ビルド機能では、以下の処理が行われています。

### トランスパイル（変換）

ブラウザは `TypeScript` や `TSX` をそのまま解釈できません。そのため、ビルド時に `JavaScript` へ変換するトランスパイルという処理が行われます。
例を以下に記載します。

```tsx
// 書いたコード（JSX）
const Hello = () => <h1>Hello, World!</h1>

// ビルド後（ブラウザが理解できるJS）
const Hello = () => React.createElement("h1", null, "Hello, World!")
```

### バンドル（まとめる）

開発時に複数のファイルに分けて書いたコードを、まとめて配信用のファイルに整理する処理です。`webpack` や `Vite` などのバンドラーがこの役割を担います。

```js
// バンドル前（2つのファイルに分かれている）
// utils.js
export const add = (a, b) => a + b

// main.js
import { add } from './utils'
console.log(add(1, 2))

// バンドル後（1つのファイルにまとまる）
const add = (a, b) => a + b
console.log(add(1, 2))
```

### ミニファイ（圧縮）

人間が読みやすいように入れていた空白や改行、コメントなどを削除してファイルサイズを最適化します。

```js
// ミニファイ前
const greeting = "Hello, World!";
// 挨拶を表示する
console.log(greeting);

// ミニファイ後
const greeting="Hello, World!";console.log(greeting);
```

## Next.js のビルドを覗いてみる

### `npm run build` の出力を読む

`npm run build` を実行すると、次のような出力が表示されます。

```
Route (app)                              Size     First Load JS
┌ ○ /                                    5.21 kB         90.3 kB
├ ○ /_not-found                          871 B           85.9 kB
└ ○ /about                              2.11 kB          87.2 kB
+ First Load JS shared by all            85.0 kB
  ├ chunks/framework-xxx.js              45.3 kB
  └ chunks/main-xxx.js                   39.7 kB

○  (Static)   prerendered as static content
```

「Size」は各ページのファイルサイズ、「First Load JS」はそのページを表示するのに必要な `JavaScript` の合計サイズです。

### `.next` ディレクトリの構造

ビルドが完了すると、プロジェクトのルートに `.next` ディレクトリが生成されます。

```
.next/
├── static/     # CSS・JS・画像などの静的ファイル
├── server/     # サーバーサイドで実行されるコード
└── cache/      # ビルドキャッシュ（再ビルドを高速化するために使われる）
```

:::message
`.next` ディレクトリの内容は `npm run build` のたびに再生成されます。`.gitignore` に追加しておくのが一般的です。
:::

## ビルドの重要性

初回デプロイ時にビルドを忘れると、本番環境には配信できるファイルが存在しないためサービスが動きません。

また、コードを修正した後にビルドを実行し忘れた場合、古いビルド結果がそのまま使われ続けます。修正が反映されない原因として、ビルド漏れは意外と多いです。

ビルド後の動作確認も大切です。開発環境では動いていても、本番向けのビルドで問題が発覚するケースがあります。

## まとめ

- ビルドとは「開発者が書いたコード」を「ブラウザが実行できる形」に変換するプロセス
- ビルドにはトランスパイル・バンドル・ミニファイの3つの主な処理が含まれる
- Next.js では `npm run build` でこれらが自動的に実行され、`.next` ディレクトリに成果物が生成される
- ビルドを忘れると古いコードがそのまま本番で動き続けるため、デプロイ前の実行は必須
