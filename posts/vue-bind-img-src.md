---
title: Vue で <img :src="@/assets/xxx.jpg"> したいときの解決法（Vite 版）
tags : [Vue.js, Vite]
---

これまでの Vue では、`img` タグ内の `src` 属性に動的な値として「プロジェクトソース内のアセットを指定したインポート」を使うことができませんでした。

これの回避策として `require()`  を使うなどがありましたが、これも現在の環境では動かない、もしくはエディタやリンターなどに厳しく怒られるようになったりしていて、どうするのが一番いいのかわりと謎な状態が続いていました（少なくとも僕の中では…）。

最近 Vue オンリーのプロジェクトでも Vite を使うようにしたのですが、そのとき Vite のリファレンスを眺めていたら「Importing Asset as URL」という機能を見つけ、これを使ったら簡単に解決できたので紹介します（前とはタイトルや説明内容が変わっていたけど同じものを指しているようです）。

[https://ja.vitejs.dev/guide/assets.html#importing-asset-as-url]

つまるところ、こうすればいいだけです。

```html
<template>
  <div class="image">
    <img :src="mikan">
  </div>
</template>

<script setup lang="ts">
import mikan from "@/assets/images/mikan.png"
</script>
```

本番ビルドではちゃんとファイルハッシュのサフィックスが付くものに自動で置き換わります。

ちなみに、この記事を書くにあたって最新の Vite のリファレンスの該当部分を読んでみたところ、Webpack にも似たような機能はあったらしいです（`file-loader`）。だったら前もこれでできたのか…？

とにかく、今後画像はもちろん一般的な静的アセットのローディングで困ることはなさそうです。
