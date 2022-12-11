---
title: elm-pages + tailwindcss
descriptoin: 
tags:
  - プログラミング
  - Elm
  - tailwindcss
---

## 概要

## 環境

### PC

Macbook Air (Apple M2)

### npm packages

- `elm-pages:2.1.10`
- `tailwindcss:3.2.4`
- `concurrently:7.6.0`

## elm-pages

[Getting Started - elm-pages](https://elm-pages.com/docs/getting-started) に従ってプロジェクトを生成する。

```
$ npx elm-pages init my-project
$ cd my-project
```

## tailwindcss

[Getting Started > Installation - tailwindcss](https://tailwindcss.com/docs/installation) の Tailwind CLI に従って tailwindcss をインストールする。

```
$ npm install -D tailwindcss@latest
```

`tailwind.config.js` を生成するために以下のコマンドを実行する。

```
$ npx tailwindcss init
```

生成された `tailwind.config.json` を次のように書き換える。

```javascript
module.exports = {
  content: ["./src/**/*.elm"],
  theme: {
    extend: {},
  },
  plugins: [],
};
```

`./public` 配下で管理する `css` でも tailwindcss を使いたい場合は、そちらも `content` の配列にパスを追加する。

次に `npm run build`、`npm run start` で tailwindcss のクラスを処理するための設定を `package.json` に追加する。
`elm-pages build`、`elm-pages dev` と平行して `tailwindcss` コマンドを実行するために [concurrently](https://www.npmjs.com/package/concurrently) をインストールする。

```
$ npm install -D concurrently@latest
```

最後に `package.json` の `scripts` に `tailwind:build`、`tailwind:watch` を追加し、`start`、`build` を修正することで `./src/**/*.elm` ファイルで tailwindcss のクラスが使えるようになる。

```json
{
  "name": "elm-pages-app",
  "scripts": {
    "postinstall": "elm-tooling install",
    "start": "concurrently \"elm-pages dev\" \"npm run tailwind:watch\"",
    "build": "npm run tailwind:build && elm-pages build",
    "tailwind:build": "tailwindcss --input tailwind.css --output ./public/style.css",
    "tailwind:watch": "tailwindcss --input tailwind.css --output ./public/style.css --watch"
  },
  "devDependencies": {
    "elm-optimize-level-2": "0.2.3",
    "elm-pages": "2.1.10",
    "elm-review": "^2.8.5",
    "elm-tooling": "^1.4.0",
    "concurrently": "^7.6.0",
    "tailwindcss": "^3.2.4"
  },
  "dependencies": {
  }
}
```

