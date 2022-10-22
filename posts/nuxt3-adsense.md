---
title: Nuxt 3 で Google AdSense を使う
tags : [Nuxt 3]
---

Nuxt 3 で Google AdSsense を動かせるようになるための情報はまだ少ないです。

AdSense から提供されたコードをそのまま使うだけではほぼ 100% エラーになり、かつ Nuxt 2 時代に使えていたプラグイン類もうまく動かないことが多いのでだいぶ苦戦しました。

Nuxt 3 の issue スレッドでも同じ悩みを抱えている人がいるのは散見されますが、まだ GA 前でかつ AdSense はまだまだ限られたユースケースなのか、困っている人数はあんまり多くない模様…。もっと情報が集まるといいなと思ってます。

ちなみに下記がそのスレッドなのですが、この記事に書いた最終的な結果を自分も投稿しています。コードの見てくれがあまり綺麗じゃないですが誰かの役に立てると嬉しいです。

[https://github.com/nuxt/framework/discussions/5586]

## やること

最終的なコードはこうなりました。

```html
<template>
  <Head>
    <Script defer crossorigin="anonymous" src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-xxxxxxxxxxxxxxxxx"></Script>
  </Head>
  <div class="ad">
    <div ref="adins"></div>
    <div ref="adpush"></div>
  </div>
</template>

<script setup lang="ts">
const adins = ref()
const adpush = ref()

onMounted(() => {
  const ins = document.createElement("ins") as HTMLElement
  ins.classList.value = "adsbygoogle"
  ins.style.display = "inline-block"
  ins.setAttribute("data-ad-client", "ca-pub-xxxxxxxxxxxxxxxxx")
  ins.setAttribute("data-ad-slot", "xxxxxxxxxx")
  ins.setAttribute("data-ad-format", "rectangle")
  adins.value.appendChild(ins)
  
  const src = document.createElement("script") as HTMLScriptElement
  src.text = "(adsbygoogle = window.adsbygoogle || []).push({});"
  adpush.value.appendChild(src)
})
</script>
```

遭遇することになるエラーは主に

- adsbygoogle.push() error: All ins elements in the … with class=adsbygoogle already have ads in them.
- adsbygoogle is not defined

などかと思われます。

前提として、

- 「https://pagead2.googlesyndication.com〜」のメインスクリプトはページで 1 度だけ読み込まれていればよい
    - async で問題ないので読み込みタイミングを気にする必要はない
- 広告タグになる `<ins>` の部分と `(adsbygoogle = window.adsbygoogle || []).push({});` は広告ユニットごとの数ぶん存在していないとダメで、かつ両者の数値（実行回数）が一致している必要がある

これらの仕様と Nuxt 3 の SSG(SSR) の挙動が絶妙に相性が悪く、「どう考えてもこのコンポーネントは一度しか呼ばれていないのになぜだ…？」みたいなことになります。丸 3 日くらいハマりました…。

見た目はよくないですが上記のコードで以降問題なく動いています。

注意点として、<strong>パッと見は広告が正常に表示されているように見えても「ページの新規表示か Nuxt として内部リンク遷移してきた状態か」で結果が大きく変わる</strong>ので注意してください。ここには深い理解が必要そうで今回はもう諦めてしまいましたが、この次の項で書く各パターンを試していく際にはページ表示の方法をそれぞれテストしていました。

### こうなるに至った経緯

とりあえず動いていればいいという方は読まなくてもいいかとは思います。

試したパターンは、下記それぞれの要素を組み合わせたものという感じになります。

- 「https://pagead2.googlesyndication.com〜」のメインスクリプトの読み込み方
    - `Head` タグを使うかどうか
    - async/defer/none のどれか（これはどれでも問題ないことがわかっているので async か defer を選ぶことをおすすめします）
    - 広告を表示したいコンポーネントに書くか app.vue に書くか
- `<ins>` タグの置き方
- `(adsbygoogle = window.adsbygoogle || []).push({});`　のスクリプトの表現の仕方
- 各要素の実行順序、コンポーネントのどこに置くかなど
- `<ClientOnly>` を使うかどうか

もっと高度なアプローチにとって全然違う実現方法を誰かが考えてくれるか、Nuxt 3 がもっと広まって便利なライブラリが登場するかなどするまではこれでいくつもりです。

## おまけ：AdSense が勝手に親コンテナに height: auto !important; を与えやがる件

今回の件とは関係はないのですが、問題を切り分けるときにわかったことがあるので記録に残しておきます。

AdSense は広告ユニットを配置した場所によって、自分が所属する親コンテナから body までのすべての経路に `height: auto !important;` を強制的に付与するという最悪な仕様があります。

昔から何度も見たことがありつつもどういうときに発生するかずっと謎に思っていたのだけど、どうやらこれは<strong>スクロール可能なコンテナ等に配置した場合に発生する</strong>ようです。

本ページがまさにそうなっているのですが、右側のサイドバーはスクロールに追従するようになっていて（`position: sticky;`）、この CSS が原因でスタイルの強制付与が起きていました。

ただ CSS が変わるだけならまだしも、このサイトはユーザーが現在読んでいる場所に応じて目次の自動ハイライトをする機能があるのですが、その実装のせいで「永遠に下にスクロールできない状態」になってしまっていました。目次ハイライトはやや実装がよくない点もあったものの、これはどうしても直したかったわけです。

これはかっこいい方法を見つけられました。

```js
const targetHeight = ref("calc(100vh - 13px * 2)")  // CSS にも v-bind() バインドする

const observer = new MutationObserver(() => {
  sidebar.value.style.height = targetHeight.value
})
observer.observe(sidebar.value, {
  attributes: true,
  attributeFilter: ["style"],
})
```

こんな感じで、AdSense にスタイルを書き換えられたら自動で検知してさらに上書きします。「やられたらやり返す」です。

ちなみに昔の AdSense はスクロール追従領域に広告ユニットを配置するのは重い規約違反扱いでしたが（一発 BAN レベルだったと思う、PV 数が多いサイトは特別扱いで許可されていた）、数年前に開放されたのでいまは問題ありません。
