---
title: "node_modulesには何が入っている？依存関係の仕組みを理解する"
emoji: "📦"
type: "tech"
topics: ["nodejs", "npm", "javascript", "packagejson", "frontend"]
published: true
---

## はじめに

Node.jsでプロジェクトを作ると、必ず現れる `node_modules` フォルダ。
`npm install` を実行した直後から何千ものファイルが詰め込まれ、容量も数十〜数百MBになることがあります。

この記事では、`node_modules` の中に何が格納されているのかを整理します。
対象読者は、Node.jsやnpmを使い始めたばかりの方や、「なんとなく動いているけど仕組みがわからない」と感じている方です。

この記事を読むと、以下のことが理解できます。

- `node_modules` の内部構造
- パッケージがどのように解決されるか
- なぜ `node_modules` が巨大になるのか

## node_modulesとは何か

`node_modules` は、**インストールしたパッケージ（ライブラリ）が格納されるディレクトリ**です。

`npm install` を実行すると、`package.json` の `dependencies` / `devDependencies` に記載されたパッケージがすべてここに展開されます。

```bash
npm install
# → node_modules/ にパッケージが展開される
```

## 格納されているもの

### 1. パッケージ本体

インストールしたパッケージのファイル一式（JavaScriptファイルや設定ファイルなど）が格納されます。

```text
node_modules/
  react/          ← reactパッケージ本体
    index.js
    package.json
    ...
  lodash/         ← lodashパッケージ本体
    ...
```

### 2. .binディレクトリ

`node_modules/.bin/` には、CLIツールの実行ファイルへのシンボリックリンクが格納されます。

```bash
ls node_modules/.bin/
# tsc, eslint, prettier, ... など
```

`npx tsc` や `npm run` でスクリプトを実行するとき、この `.bin/` ディレクトリが参照されます。

### 3. 間接依存（依存の依存）

自分でインストールしたパッケージだけでなく、**そのパッケージが必要とするパッケージ（間接依存）** も格納されます。

たとえば `react` をインストールすると、`react` が依存している `loose-envify` なども一緒にインストールされます。

```text
node_modules/
  react/
  loose-envify/    ← reactが依存しているパッケージ
  js-tokens/       ← loose-envifyが依存しているパッケージ
```

これが `node_modules` が巨大になる主な理由です。

## package.jsonとの関係

`package.json` には、プロジェクトが直接依存するパッケージとそのバージョン範囲が記載されています。

```json
{
  "dependencies": {
    "react": "^18.0.0"
  },
  "devDependencies": {
    "typescript": "^5.0.0"
  }
}
```

| 種類              | 用途                                   | 例                         |
| ----------------- | -------------------------------------- | -------------------------- |
| `dependencies`    | 本番環境でも必要なパッケージ           | react, axios               |
| `devDependencies` | 開発時のみ必要なパッケージ             | typescript, eslint         |

`npm install` はこの記述をもとに `node_modules` を構築します。

## package-lock.jsonの役割

`package.json` にはバージョン範囲（`^18.0.0` など）しか書かれていません。
実際にインストールされたバージョンは `package-lock.json` に記録されます。

```json
{
  "react": {
    "version": "18.3.1",
    "resolved": "https://registry.npmjs.org/react/-/react-18.3.1.tgz",
    ...
  }
}
```

`package-lock.json` があることで、チームの誰がインストールしても同じバージョンが使われます。

:::message
`package-lock.json` は必ずGit管理に含めましょう。これを共有することで、環境差異によるバグを防げます。
:::

## requireはどこを探索するか

Node.jsで `require('react')` と書いたとき、どのファイルが読み込まれるのでしょうか。

Node.jsは以下の順番で `node_modules` を探索します。

1. 現在のファイルと同じディレクトリの `node_modules/`
2. 親ディレクトリの `node_modules/`
3. さらに親ディレクトリの `node_modules/`（ルートまで繰り返す）

```text
src/
  components/
    Button.tsx     ← ここで require('react') すると…
  node_modules/    ← ① src/components/node_modules/ を探す（なければ）
node_modules/      ← ② src/node_modules/ を探す（なければ）
                   ← ③ ルートの node_modules/ を探す → ここで見つかる
```

通常はルートの `node_modules/` にパッケージが置かれるため、ここで解決されます。
この仕組みを知っておくと、モノレポなど複数の `node_modules/` が存在する環境でのトラブル解決に役立ちます。

## なぜnode_modulesは巨大になるのか

前述した間接依存に加え、もう1つ理由があります。

**フラット化されたディレクトリ構造**

npm v3以降、`node_modules` は**フラット構造**を採用しています。
依存ツリーをなるべく1階層に展開することで重複インストールを減らしますが、その分ルートの `node_modules/` に大量のパッケージが並ぶことになります。

## .gitignoreでの扱い

`node_modules` は `package.json` さえあれば `npm install` で再現できます。
そのため、Gitの管理対象から除外するのが一般的です。

```gitignore
# .gitignore
node_modules/
```

:::message alert
`node_modules` をGitにコミットしてしまうと、リポジトリが非常に重くなります。必ず `.gitignore` に追加してください。
:::

## まとめ

- `node_modules` には、インストールしたパッケージ本体と間接依存パッケージが格納される
- `.bin/` ディレクトリにはCLIコマンドの実行ファイルが置かれる
- `package.json` がバージョン範囲を定義し、`package-lock.json` が実際のバージョンを固定する
- `node_modules` は再現可能なので、`.gitignore` で管理対象から除外する
