---
title: Nuxt を使わない Vue 3 だけで各 Vue 系 API や自作コンポーネントを自動インポートする
tags : [Vue.js, Nuxt]
---

自分は SEO (もっというと OGP) さえ除外していいならほとんどのサイトは SPA でいいと思っているくらいなのでネイティブな Vue 3 オンリーで書くのもとても好きなんだけど、Nuxt にある各種自動インポート系の機能だけは単純にあったほうがいい気がして残念に思っていました。

プラグイン的なのないのかなと思って調べてみたらあっさり見つかったので導入方法と使い方をまとめます。

たぶんですが Nuxt 3 で内部に用いられているものと同じ仕組みだと思われます。

## つかうもの

パッケージを 2 つ使用する。

### unplugin-auto-import

[https://github.com/antfu/unplugin-auto-import]

Vue のグローバル API をはじめ、各種外部パッケージ（lodash とか store とかなんでも）から自分で定義した単なるユーティリティ関数まで、何でも好きに登録して自動インポートできるようにしてくれるすごいヤツ。しかも Tree Shaking 対応、もちろん TypeScript でもそのまま使える。

こちらでも `components/` という指定ができそうな部分があるので Vue Component の自動インポートも賄えてしまうのかと思ったのですが、試してみた感じそういうことではなく、module.exports しているものを外部モジュールでオートインポートできるという感じでした（そうじゃないとこの次に書くもうひとつの方の存在意義もなくなってしまうし）。

```ts
// Auto import for module exports under directories
// by default it only scan one level of modules under the directory
dirs: [
  // './hooks',
  // './composables' // only root modules
  // './composables/**', // all nested modules
  // ...
],
```

### unplugin-vue-components

[https://github.com/antfu/unplugin-vue-components]

というわけで、`components/` 配下につくった自分の各コンポーネントの自動インポートはこちらで行います。

Vite, Webpack, Vue CLI, Rollup, esbuild などに対応していて（これは上記の unplugin-auto-import も同じ）、こちらも Tree Shaking されます。

## 導入方法

```bash
npm i unplugin-auto-import unplugin-vue-components --save-dev
```

でインストールしたら、vue.config.js や vite.config.ts などに設定を書きます。それぞれのビルドツールごとの詳細もリポジトリに記載がありますのでそちらをどうぞ。

```js
module.exports = {
  configureWebpack: {
    plugins: [
      require("unplugin-auto-import/webpack")({
        include: [
          /\.vue$/, /\.vue\?vue/,
          /\.[tj]sx?$/,
        ],
        imports: [
          "pinia",
          "vue",
          "vue-router",
          "@vueuse/core",
          {
            "ohmyfetch": [
              "$fetch",
            ],
            "@/router/router": [
              ["default", "router"],
            ],
          }
        ],
        dts: true,
        eslintrc: {
          enabled: true,
        },
      }),
      require("unplugin-vue-components/webpack")({
        dirs: [
          "./src/components",
        ],
        deep: true,
        dts: true,
      }),
    ],
  },
}
```

すべてのパラメータについてはそれぞれの README を参照してもらうとして、上記はいまの自分の設定をほぼそのまま載せているのでこれを使って説明します。

まず unplugin-auto-import のほう。インラインで説明入れます。

```js
require("unplugin-auto-import/webpack")({
  include: [  // 自動インポートが適用されてほしい対象（インポートする対象ではない）
    /\.vue$/, /\.vue\?vue/,
    /\.[tj]sx?$/,  // .ts, .tsx, .js, .jsx
  ],
  imports: [  // インポートする対象
    "pinia",  // インストールしたパッケージはそのまま書ける
    "vue",
    "vue-router",
    "@vueuse/core",  // 同上
    {
      "ohmyfetch": [
        "$fetch",  // 名前付きインポートをしたい場合
      ],
      "@/router/router": [  // プロジェクトのパス alias も使える (@)
        ["default", "router"],  // デフォルトエクスポートの alias as
      ],
    }
  ],
  dts: true,  // インポートした型定義ファイルを生成する（後述）
  eslintrc: {  // VS Code のエラー対応（後述）
    enabled: true,
  },
}),
```

次、unplugin-vue-components のほう。

```js
require("unplugin-vue-components/webpack")({
  dirs: [
    "./src/components",
  ],
  deep: true,
  dts: true,
}),
```

こっちは見たまんまなので特に書くことなかった。

どちらのプラグインも<strong>設定を書いたあと一度ビルド</strong>（`npm run serve` でいい）<strong>しないと適用されない</strong>ことには注意してください。これはすなわち下記の 2 つのファイルがプロジェクトに生成（更新）されるのを待つことを意味します。

- auto-imports.d.ts
- components.d.ts

それぞれ unplugin-auto-import、unplugin-vue-components が生成するファイルですが、ファイル名と場所は設定で変えられます（`dts` の引数に入れる）。TypeScript での使用がメイン想定のためデフォルトで生成までされるはずですが、生成を無効化することもできます。gitignore するかはお好みで。

## つかいかた

もうオートインポートされてます。以上。

## Q&A

### インポートしたコンポーネントが template タグ内でシンタックスハイライトされないんだけど？

試せることは

- 設定で `dts: true` を書く
- tsconfig.json の include に components.d.ts を追加する
- VS Code の設定で editor.semanticHighlighting.enabled が true になっているか確認する

です。

自分は 2 つめが盲点で数日諦めてました。

### unplugin-auto-import のほうで ESLint が「それ宣言されてないよ」って怒ってくるんだけど？

実際には問題なく色々うまくいっていても、VS Code の ESLint がエラーを出してくる場合があるので、

```js
require("unplugin-auto-import/webpack")({
  eslintrc: {
    enabled: true,
  },
}),
```

これを書き、.eslintrc-auto-import.json というファイルが出力されたことが確認できたら、.eslintrc.js に

```js
module.exports = {
  extends: [
    "./.eslintrc-auto-import.json",
  ],
}
```

を追記すれば OK。
