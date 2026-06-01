---
title: "package.jsonって何？Node.jsプロジェクトの設定ファイルを理解する"
emoji: "📦"
type: "tech"
topics: ["nodejs", "npm", "javascript", "packagejson", "frontend"]
published: true
---

## はじめに

Node.jsプロジェクトを触ると必ず目にする `package.json`。
でも「なんとなく存在は知ってるけど、中身の意味は正直よくわからない」という方も多いのではないでしょうか。

この記事では、`package.json` が何者で、どんな役割を持っているのかを丁寧に解説します。

**対象読者**

- Node.jsやnpmを使い始めたばかりの方
- `package.json` を見たことはあるけど理解が曖昧な方

**この記事で得られること**

- `package.json` の役割と基本構造の理解
- 主要なフィールドの意味と使い方
- `dependencies` と `devDependencies` の違い

## package.jsonとは

`package.json` は、Node.jsプロジェクトの **設定ファイル兼メタデータファイル** です。
プロジェクトのルートディレクトリに置かれ、npm（またはyarn・pnpm）がこのファイルを読み取って動作します。

大きく分けて3つの役割があります。

1. **プロジェクトの情報を記述する**（名前・バージョン・作者など）
2. **依存パッケージを管理する**（`npm install` で何をインストールするか）
3. **スクリプトを定義する**（`npm run dev` などのコマンドを登録）

## 基本的な構造

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "description": "サンプルアプリ",
  "scripts": {
    "dev": "node index.js",
    "build": "webpack"
  },
  "dependencies": {
    "express": "^4.18.2"
  },
  "devDependencies": {
    "eslint": "^8.0.0"
  }
}
```

## 主要フィールドの解説

### name・version

プロジェクトの名前とバージョンです。
npm に公開しない場合でも、プロジェクトを識別するために記述します。

```json
{
  "name": "my-app",
  "version": "1.0.0"
}
```

バージョンは **セマンティックバージョニング**（`メジャー.マイナー.パッチ`）に従います。

### scripts

`npm run <スクリプト名>` で実行できるコマンドを登録します。

```json
{
  "scripts": {
    "dev": "node server.js",
    "build": "tsc",
    "lint": "eslint src/**/*.ts"
  }
}
```

`npm run dev` と打つだけで `node server.js` が実行される仕組みです。

### dependencies

本番環境でも必要なパッケージを記述します。
`npm install <パッケージ名>` でインストールすると自動的にここに追記されます。

```json
{
  "dependencies": {
    "express": "^4.18.2",
    "axios": "^1.4.0"
  }
}
```

### devDependencies

開発時にのみ必要なパッケージを記述します。
`npm install <パッケージ名> --save-dev` でインストールします。

```json
{
  "devDependencies": {
    "typescript": "^5.0.0",
    "eslint": "^8.0.0"
  }
}
```

:::message
**dependenciesとdevDependenciesの使い分け**

- `dependencies`: アプリの動作に必要なもの（例: Express, React）
- `devDependencies`: 開発・ビルド時にだけ必要なもの（例: TypeScript, ESLint）

本番環境では `npm install --omit=dev` とすると `devDependencies` はインストールされません。
:::

:::message alert
**よくあるミス: devDependencies に入れるべきものを dependencies に書いてしまう**

TypeScript や ESLint は本番実行には不要です。
`dependencies` に入れると本番バンドルやデプロイ時に余分なパッケージがインストールされます。
:::

### バージョン指定の記号

`^` や `~` は、許容する更新範囲を表す記号です。

| 記号 | 意味 | 例 |
|------|------|-----|
| `^` | マイナー・パッチの更新を許容 | `^4.18.2` → `4.19.0` や `4.18.5` は OK（`5.0.0` は NG）|
| `~` | パッチの更新のみ許容 | `~4.18.2` → `4.18.x` なら OK |
| なし | バージョンを固定 | `4.18.2` のみ |

## package.jsonはどう作るか

`npm init` コマンドで対話形式に作成できます。

```bash
npm init
```

質問に答えていくと `package.json` が生成されます。
デフォルト値でよければ `npm init -y` で一気に作成することも可能です。

:::message
**チームでの環境統一に役立てる**

`package.json` と `package-lock.json` を合わせてバージョン管理すれば、`npm install` 一発でチーム全員が同じバージョンの依存パッケージをインストールできます。
:::

## まとめ

- `package.json` はNode.jsプロジェクトの設定ファイルで、プロジェクト情報・依存管理・スクリプト定義の3役を担う
- `dependencies` は本番用、`devDependencies` は開発用のパッケージを管理する
- `scripts` にコマンドを登録すると `npm run <名前>` で呼び出せる
- バージョン指定の `^` `~` は許容する更新範囲を示す

次のステップとして、`package-lock.json` や `node_modules` の役割を理解すると、npm の動作についてより深く学べます。
