---
title: "「npm run devのrunって何？」から始めるNode.js入門"
emoji: "☕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nodejs", "javascript"]
published: true
---

## はじめに

**対象読者**: Next.js や React のプロジェクトで `npm` コマンドを使っているが、`Node.js` の基本をあまり意識したことがない方。

Next.jsのプロジェクトを触っていて、ふと「そもそも `Node.js` とはなんだろう」と疑問に思いました。
`npm run dev` は毎日叩いているのに、なぜ `run` が必要なのか説明できませんでした。
`npm install` とは何が違うのかもわかっていない状態でした。

この記事を読むと、以下の2点がわかります。

- `Node.js` の正体（「サーバー」ではなく「実行環境」）
- `npm` コマンドと `npm run` コマンドの違い

## Node.js とは

私は学習前「Node.js＝サーバー」と思っていました。
ですが正しくは **「Node.js＝JavaScriptの実行環境」** です。

本来、JavaScriptはブラウザ上でしか動かせませんでした。
ですが、`Node.js` の登場により、PCやサーバなどブラウザの外でも実行できるようになりました。

```text
【ブラウザ】          【Node.js】
JavaScriptエンジン    JavaScriptエンジン（V8）
   ↓                    ↓
ブラウザ上で実行      PC・サーバ上で実行
（DOM操作など）       （ファイル操作、HTTP通信など）
```

では「サーバー」はどこにあるのかというと、`Node.js` の標準モジュール（`http` モジュールなど）を使って**自分でサーバーを構築できる**という関係です。
サーバー機能が内蔵されているわけではなく、サーバーを作るための道具が揃っている環境だと捉えるとよいかと思います。

たとえば、以下のように数行で簡易サーバーを立てることができます。

```js
const http = require("http");

const server = http.createServer((req, res) => {
  res.writeHead(200, { "Content-Type": "text/plain" });
  res.end("Hello, Node.js!");
});

server.listen(3000);
```

実際に、webpack や Vite などのビルドツールも Node.js 上で動作しています。

:::details Node.js で直接 JavaScript を実行してみる
`node` コマンドを使えば、ブラウザを開かずに JavaScript ファイルを実行できます。

```bash
node index.js
```

`Node.js` が「ブラウザの外でも JavaScript を動かせる環境」であることを、最も手軽に体感できる方法です。
:::

## `npm` と `npm run` の違い

`npm`（Node Package Manager）は `Node.js` のパッケージ管理ツールです。
ライブラリのインストールや依存関係の管理を担っています。

`npm` のコマンドには大きく分けて **ビルトインコマンド** と **`npm run` で実行するユーザー定義スクリプト** の2種類があります。

| 種類 | コマンド例 | 説明 |
|------|-----------|------|
| ビルトインコマンド | `npm install`, `npm init` | `npm` に最初から組み込まれている標準コマンド |
| ユーザー定義スクリプト | `npm run dev`, `npm run lint` | `package.json` の `scripts` に自分で定義するコマンド |

### 最初に戸惑ったこと

実際に、最初は `npm dev` と打ってエラーになった経験があります。
「`npm install` は `run` なしで動くのに、なぜ `dev` は動かないのか？」
→その疑問の答えが、ビルトインコマンドとユーザー定義スクリプトの違いでした。

### ユーザー定義スクリプトの例

`package.json` の `scripts` セクションに記述した内容が `npm run` で実行できます。

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "lint": "eslint . --ext .ts,.tsx"
  }
}
```

```bash
# ビルトインコマンド（npm に組み込み済み）
npm install
npm init

# ユーザー定義スクリプト（package.json の scripts に定義したもの）
npm run dev
npm run lint
```

### 注意: `run` を省略できるスクリプトがある

:::message
`npm start` や `npm test` など一部のスクリプトは `run` を省略して実行できます。
これらは npm が特別扱いしている「ライフサイクルスクリプト」と呼ばれるコマンドです。

```bash
npm start    # npm run start と同じ
npm test     # npm run test と同じ
```

ユーザー定義スクリプトでありながら、ビルトインコマンドのように使えます。
:::

## まとめ

今回は `Node.js` と `npm` の基本を整理しました。

- `Node.js` はサーバーではなく **JavaScriptの実行環境** である
- `npm` のビルトインコマンドと `npm run` のユーザー定義スクリプトは別物である
- ただし `npm start` のように `run` を省略できる例外もある

「なぜ `run` が必要なのかわからない」という状態から一歩前進できました。
`npm run dev` は `package.json` の `scripts` に定義されたコマンドを呼び出しているのだと理解できるようになりました。

次のステップとして、以下のテーマも深掘りしていく予定です。

- `package.json` の各フィールドの役割
- ビルドツール（Vite / webpack）の仕組み

参考リソース:

- [Node.js 公式ドキュメント](https://nodejs.org/ja/learn)
- [npm Docs](https://docs.npmjs.com/)
