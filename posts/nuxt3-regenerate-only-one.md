---
title: Nuxt 3 で更新のあった記事だけ re-generate してピンポイントにデプロイし直す
tags : [Nuxt 3, CI/CD, GitHub Actions]
---

現時点での Nuxt 3 には「指定したページだけを generate する」という機能はなく、ブログを SSG で作っている場合は記事を更新するたびに全サイト再生成というハメになってしまいイマイチです。

かなりパワープレイなワークアラウンドですが、一応やりたいことは実現できたので記録しておきます。以前どこかで Discussions だか issues だかで似たような要望を見た気もするので、そのうち純正機能が出てくるとは思います。

## やりたいこと

記事の更新（もしくは新規追加）をフックとして nuxt にコマンドを打ちたいので、当然 CI で外から入力する形になり、`nuxi xxx` に該当するコマンドだけでなんとかしないといけません。

[https://nuxt.com/docs/api/commands/generate]

いまのところ `nuxi generate` にそのようなオプションはなく、3.0 以降で増えた他のコマンドを眺めてみてもやりたいことはできそうにありません。

そんなわけで、

- CI の中でだけ Nuxt のコンフィグ自体をいじってしまう
- いじった状態で `nuxi generate` する

ことによって問題の解決を計ってみます。

## やること

全体の流れは以下のようになります：

1. 生成したいページのルートを受け取る
2. コンフィグを書き換える
3. nuxi generate する
4. 生成したページのみ再デプロイする

CI/CD のツールは GitHub Actions を想定とします。

### 1. 生成したいページのルートを受け取る

まずワークフローを外から起動するときに、生成したいページの URL パスを受け取る必要があります。

```yaml
on:
  workflow_dispatch:
    inputs:
      slug:
        description: The slug to generate (no `/` as prefix required)
        required: true
        type: string
```

GitHub Actions のワークフローは外から HTTP リクエストで起動させることができますが、その設定のために `workflow_dispatch` が必要です。ここはググればいっぱい出てくるので割愛します。

その引数にあたるものを設定できるのが `inputs` で、ここに URL パスを置きます。自分は `/` なしの記事スラッグをそのまま受け取る状態にしています。

あとは下記のように、以降のステップで使えるように環境変数にでもセットしておきます。

```yaml
- name: Set a slug to generate
  run: |
    echo ROUTE_TO_GENERATE=${{ github.event.inputs.slug }} >> $GITHUB_ENV
```

（ちなみに上記ワークフロー全体を公開リポジトリに置いていますので下記からご覧になれます）

[https://github.com/mirumirumi/mirumi-me/blob/main/.github/workflows/generate-only-specified-post.yaml]

### 2. コンフィグを書き換える

Nuxt 3 にはない単体ルート生成をどうやるのかというと、

- SSG の前に必ず生成をしてほしいページを指定できる `nitro.prerender` を使う
- サイト内のリンクをクロールして自動で生成ページを検知していく機能をオフにする

の 2 つを組み合わせることで実現します。

nuxt.config.ts 的にいうと、

```ts
export default defineNuxtConfig({
  nitro: { 
    prerender: { 
        crawlLinks: false,
        routes: ["/slug"],
    },
  },
})
```

というところが該当します。Nuxt 内のドキュメントに記載はなく（特に `crawlLinks`）、Nitro のリファレンスを見に行かないと気づけません。

上記の記載状態だと「`/slug` ページだけを生成してください」の状態になっており、おそらく僕らが意図しているものです。

注意点として<strong> `/pages` ディレクトリに用意している固定のページ類の生成を止める方法は基本的にない</strong>のでそこは留意してください。つまり実際に generate されるのは「`/pages` ディレクトリにあるもの＋`/slug` ページ」となります。

で、これを CI からいじくり回すために…

```ts
export default defineNuxtConfig({
  // ...

  // CAUTION!: The following comment are used by CI to re-generate specified post
  // ### nitro: { prerender: { crawlLinks: false, routes: [###] } },
})

```

こんなものを最後に書いてコミットしておきます。

そしたらこれを GitHub Actions 内から書き換えます。

```yaml
- name: Re-generate a specified post
  run: |
    sed -irz "s/\/\/ ### //g" nuxt.config.ts
    sed -irz "s/\[###\]/['\/${{ env.ROUTE_TO_GENERATE }}']/g" nuxt.config.ts
```

sed コマンドの挙動が怖くてわざわざ 2 行に分けて書いているのですが、とにかくこれでさっきの「`/slug` ページだけを生成してください」の指定に変換できます。

### 3. nuxi generate する

あとは generate してお好みで該当部分だけデプロイするなりするだけです。
