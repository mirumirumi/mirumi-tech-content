---
title: Vue.js の状態管理ライブラリ Pinia の使い方まとめ
tags : [Vue.js, Pinia]
---

この記事は公式チュートリアルの内容、開発中に実際に困ったポイント、その解決方法などを中心にしつつ自分へのメモを主目的としてまとめていくものです。随時追記していきます。

ちなみにもう僕は Pinia 最高派です 🍍

[https://pinia.vuejs.org/]

## Vuex と Pinia

2021 年終盤、これまで Vue 公式で推奨されていた状態管理ライブラリが [Vuex](https://vuex.vuejs.org/ja/) から [Pinia](https://pinia.vuejs.org/) に変わりました。

例えば Vetur -> Volar は当時でも言われて久しい感じでしたけれど、Vuex に対する Pinia という名前はだいぶぽっと出だったので混乱した人も多いと思われます。

これまで Vue 3 に対して公式にサポートされていた状態管理ライブラリは Vuex 4 というものですが、これの次世代バージョン、つまり Vuex 5 の検討内容が Pinia の根底にあるものです。

…というのは実はやや語弊があり、実際には「新しい状態管理の形を模索していたときに作っていたものがほとんど Vuex 5 の内容を含んでいた」ということらしく（これが Pinia のこと）、そのシンプルな API が評価され Vue 公式のツールチェインに取り込まれ RFC も通過した、という経緯のようです。

Vuex 4 の主な課題は

- TypeScript の恩恵を受けるのが難しい
- ストアのモジュール分割の手間の多さ
- Vue 3 で完全にデフォルトになったと思われる Composition API とイマイチ相性がよくない
- 全体的に冗長な記述が多い

などでした。Pinia の特徴のほとんどはこれらの改善そのものであると思って差し支えありません。それと、この他の重要な変更点として `mutations` が廃止されたことが挙げられます（後述）。

Composition API の利用が前提になっているため、Vue 2 でも Pinia の利用は可能なものの `@vue/composition-api` か `Vue ^3.2.0-0` が備わっている場合のみに限定されています。

## 個人的に Pinia が素晴らしいと感じているところ

細かい話に入る前に感想ベースの要約を書いておきます：

- 記述が少ない
    - 全てにおいて見た目がスッキリした、ストアを使う心理的ハードルがかなり減った
    - 特に今までの `store.commit("untarakantara", value)` がしんどかったのでこれがなくなったのが本当に素晴らしい（これは mutation がなくなったということ自体の話ではなくコードの 見てくれ としての話）
- 型が効く
    - 安心感が違います
- 分割ラク
    - そもそも「分割」という考え方ですらない、「状態管理の状態管理」に悩まされるという現象が格段に減りました
- mutation じゃなくても編集できる
    - 最も多くの議論があった場所ですが、少なくとも僕はこの変更に賛成している側です

## 目的ごとの使い方まとめ

[公式チュートリアル](https://pinia.vuejs.org/introduction.html)（この記事執筆時点で日本語ページはなし）の内容をベースに自分の解釈がいくらか混ざっている感じでまとめています。

頑張って読み込めばだいたいのことは書いてあるのだけど、「こういうときどうする？」といういわゆるクックブック的なドキュメントはまだ整備が浅いように思えます。今後に期待しましょう。

### ストアの定義

```ts
// store/counter.ts

import { defineStore } from "pinia"

export const useStore = defineStore("counter", {
  state: () => ({
    count: 0,
  }),
  getters: {
    tenfold() {
      return this.count * 10
    },
  },
  actions: {
    increment() {
      this.count++
    },
  },
})
```

これが「ストア」の 1 つの単位です。

pinia モジュールからは `defineStore` だけをインポートし、エクスポートするのは `useStore` のみです。これは後でコンポーネント側で使います。

`defineStore` の第一引数にある counter というのが気になると思いますが、これはストアを複数使うユースケース（今まで分割と呼んでいたもの）以外では特に必要ありません。main とか store とかでもいいと思います。

それ以外の部分は Vuex 時代に慣れ親しんだものと全くといっていいほど変わっていないのがわかるでしょう。mutations だけはなくなっていますけど。

タイプセーフに関しても基本的にはこれだけで型推論が機能するゆえに必要十分なケースが多いですが、細かく指定をしたい場合には以前のように interface を使うことができます：

```ts
interface State {
  counter: number | string,
}

export const useStore = defineStore("counter", {
  state: (): State => ({
    counter: "0",
  }),
})
```

この場合は「useStore では初期値のセットをしている」と考えるとわかりやすいと思います。

### コンポーネントからストアを参照する

コンポーネント側です。

```html
<script setup lang="ts">
import { useStore } from "@/store/store"

const store = useStore()

// state の参照
console.log(store.counter)  // 0

// state の変更
store.counter = 10
console.log(store.counter)  // 10

// getters の使用
console.log(store.tenfold)  // 100
</script>
```

store.ts 内でエクスポートした `useStore` 関数をインポートし、適当な変数にそれを格納します。このコンポーネントファイル内ではこの変数が全てのストアとしての役割を担います（もちろん別のストアを定義している場合はそれぞれまた定義します）。

実際にエディタ上で書いてみるとわかるのですが、この時点で既に型の安全確保がなされています。変なものを入れようとするとちゃんと怒ってもらえるということ。

型以外で特に注目すべきは、state の書き換えをまるで普通のオブジェクトプロパティかのように扱えているところです。

```ts
// state の変更
store.counter = 10
console.log(store.counter) // 10
```

しかも getters の簡潔さも素晴らしいです。

```ts
// getters の使用
console.log(store.tenfold) // 100
```

たぶんシンプルすぎて Vuex 時代に store.**getters**.tenfold とかやっていたことなんてもうみなさんも忘れていたのではないでしょうか。間に挟まる種別指定がなくなり 1 ブロックの記述が少し減るだけでソースコード全体も格段に読みやすくなると感じます（もちろんこの手の話は常に明示するメリットとの天秤ではあります）。

actions は後ろで別に書いています。

<div class="box-common box-info">
<p>ストアによる状態管理というのは、ちょっとやそっとのことでは容易にデータを書き換えられない「変更のハードルの高さ」によってある程度の管理強度が担保されているといえます。</p>
<p>mutation がなくなることについて記述のクリアさだけで言えば議論の余地はないものの、この「定められた方法以外で自由に状態を書き換えられてしまっていいのか」という部分はやはり大きな争点になっていたようです。</p>
<p>Pinia の issues でも同様の問い合わせを見つけることが可能で、例えば下記のスレッドはこの議論の概要を知るのにかなりよい例と思われます：</p>

[https://github.com/vuejs/pinia/issues/58]

<p>冒頭で「少なくとも僕は賛成派です」と書いたのは、「mutation がほとんど形骸化した儀式のような状態になっていたことを鑑みると実質的な対応としてこれはよい判断だと思った」という感じです。</p>
</div>

### ストアの中で各変数を参照する

getters の中で state を参照する：

```js
export const useStore = defineStore("counter", {
  state: () => ({
    count: 0,
  }),
  getters: {
    tenfold() {
      return this.count * 10
    },
  },
})
```

getters の第一引数には state が入ってくるのでそれを使うこともできます。

```js
export const useStore = defineStore("counter", {
  state: () => ({
    count: 0,
  }),
  getters: {
    tenfold(state) {
      return state.count * 10
    },
  },
})
```

getter から他の getter を参照する：

```js
export const useStore = defineStore("counter", {
  state: () => ({
    count: 0,
  }),
  getters: {
    tenfold() {
      return this.count * 10
    },
    tenfoldToString() {
      return this.tenfold.toString()
    },
  },
})
```

### ストアを分割する（目的ごとにストアを作る）

Pinia ではストアを定義する段階でそもそも「かたまり」を分けられるので、これを呼び出す useStore もわけてエクスポートしておくだけで簡単にストアの分割が実現できます。

```js
// カウンター用のストア
export const useCounterStore = defineStore("counter", {
  state: () => ({
    count: 0,
  }),
  getters: {
    tenfold() {
      return this.count * 10
    },
  },
  actions: {
    increment() {
      this.count++
    },
  },
})

// ユーザーのセッション情報用のストア
export const useSessionStore = defineStore("session", {
  state: () => ({
    userId: "abcd-efgh",
    token : "xxxxxxxxxxxxx",
  }),
  getters: {
    isLoggedIn() {
      return this.token !== undefined
    },
  },
})
```

id（defineStore の第一引数）もそれぞれのストアで変更するのを忘れないようにします。あとはそれぞれの useStore（今回の例では「useCounterStore」と「useSessionStore」）をただインポートすればコンポーネント側で普通に使えます。

```js
import { useCounterStore, useSessionStore } from "@/store/store"

const storeCounter = useCounterStore()
const storeSession = useSessionStore()

console.log(storeCounter.counter)  // 0
console.log(storeSession.userId)   // "abcd-efgh"
```

一般的には  `src` ディレクトリ内に `store` などの名前でさらにディレクトリを切っているケースがほとんどでしょうから、ここでファイル名 1 つに対し 1 つのストアを定義して管理するのもよさそうです。数個程度ならファイル 1 つでもいいのかな。

<div class="box-common box-info">
<p>わざわざファイルを分けて複数のストアを定義する必要がない「Nested stores」という機能もあるのですが、メリット含め詳細をよく理解できていないため末尾の「よくわかっていないところ」で紹介するに留めています。</p>
</div>

ちなみに id は下記のように書くこともできます。こちらの方が目に入りやすいので id の変更忘れなどが減りそうな気がしますね。

```js
// store/counter.js

import { defineStore } from "pinia"

export const useStore = defineStore({
  id: "counter",
  state: () => ({
    count: 0,
  }),
  getters: {
    tenfold() {
      return this.count * 10
    },
  },
  actions: {
    increment() {
      this.count++
    },
  },
})
```

### 複数の state を一挙に変更する

ある一定のまとまりを持った state 群を同時に書き換えたいとき、Vuex では mutations（commit）を使って処理を定義できましたが、Pinia では次のように書きます：

```js
store.$patch({
  usesrId: undefined,
  token  : undefined,
})
```

配列への要素の追加など、上記の書き方では現実的に難しいシーンに対応するタイプもあります：

```js
store.$patch((state) => {
  state.items.push({ name: "book", quantity: 1 })
})
```

これらユースケースの存在を思い出したとき初めて commit のちゃんとした(？)有用性に気づいた気がしましたが、安心してください。これは actions が適任なのです。後述します。

### ストアをリセットする

ストアのすべての state を初期値に戻します：

```js
store.$reset()
```

### ストアごとまるまる置き換える

リセットにプラスして次なる値セットの初期化まで行いたいときなどにまるごと置き換えが可能です：

```js
store.$state = {
  name : "みるみ",
  site : "https://mirumi.me/tech",
}
```

### actions（ストア内で非同期処理を行う）

actions は「非同期処理専用のもの」と思ってしまいがちですが、位置づけ的にはコンポーネント内の methods に近く、つまりビジネスロジックを書くための場所であるということになっています。

例えば先ほどの `$patch` による一括更新なども、毎回同じ形で使うものならば actions で定義しておくと安心感があります：

```js
export const useStore = defineStore("session", {
  state: () => ({
    userId: "",
    token1: "",
    token2: "",
    token3: "",
  }),
  actions: {
    initSession() {
      this.userId = ""
      this.token1 = ""
      this.token2 = ""
      this.token3 = ""
    },
  },
})
```

<span style="font-size: 0.8em;">（この例だと他に対象の state がないので `store.$reset()` とかやったほうがいいということになっちゃいますけどね）</span>

そしてこの考え方の上に「ここには非同期処理も書けるよ」というのが乗っていると思っておくとよさそうです。

```js
actions: {
  async signin(id, password) {
    try {
      this.auth = await api.post({ id, password })  // API リクエストなど
    } catch (error) {
      showTooltip(error)
      return
    }
  },
},
```

なお、Promise が返却されるなら記法として async/await などに限定されることはありません。

### ストアの状態を監視する

ストアの状態監視です。

```js
store.$subscribe(() => {
  // you want to do after the store is updated
})
```

公式ドキュメントに記載がされてからもまだ日が浅く正直よくわかっていないところが多いのですが、とりあえずこれでストアの変更を検知することはできます。

第二引数に state が入ってくるのでそれを使うことも可能です（第一引数は mutation という独自オブジェクトなのですが中身を見る限り特に用事がなく実用性が不明です…）。

```js
store.$subscribe((mutation, state) => {
  console.log(state)  // 変更後の state オブジェクト全体
})
```

もうひとつ `watch` を使うものもあります。

```js
watch(
  pinia.state,
  (state) => {
    // persist the whole state to the local storage whenever it changes
    localStorage.setItem('piniaState', JSON.stringify(state))
  },
  { deep: true }
)

```

「 `watch`  よりも `$subscribe` を使う利点は、サブスクリプションがパッチの後に一度だけ起動することです」ということで、おそらくパフォーマンス上の理由から後者に優位性があるのではという浅い理解をしています。

Vue コンポーネントのように変更前後の値をそれぞれ取得できたらとても嬉しいなと思うのですが、今のところそのような機能は両者ともないようです。

### コンポーネントの外（main.tsなど）でストアを使う

ストアを使いたいシーンがすべて Vue コンポーネントの内側であるとは限りません。最もよくあるケースとしては `main.ts` や Vue Router で使う `router.ts` （ `index.ts` ）などでしょう。

しかし何か面倒な手順が必要なわけではなく、基本的には今までと同じ通り useStore をインポートしてくるだけです。

```js
import { useUserStore } from '@/stores/user'
import { createApp } from 'vue'
import App from './App.vue'

// ❌  fails because it's called before the pinia is created
const userStore = useUserStore()

const pinia = createPinia()
const app = createApp(App)
app.use(pinia)

// ✅ works because the pinia instance is now active
const userStore = useUserStore()
```

ただし見てわかるように、`createPinia()` した App がアプリケーションにバインドされないとそもそもストア自体が機能しないために useStore の定義は `app.use()` よりもあとにする必要はあります。

Vue Router を使う場合も同様の注意が必要です。何も考えずファイル冒頭で import 文を書いてしまうと問題が起きる可能性があるので、それぞれ必要なアクションの中で都度定義するのが望ましいです。

```js
// ❌ Depending on the order of imports this will fail
const store = useStore()

router.beforeEach((to) => {
  // ✅ This will work because the router starts its navigation after
  // the router is installed and pinia will be installed too
  const store = useStore()

  if (to.meta.requiresAuth && !store.isLoggedIn) return '/login'
})
```

ルーターの中でロジックを書くというシーンの多くは「ページ遷移時にログイン状態を確認する」など beforeEach() が使われるときのはずなので、この関数内で定義するのが一番手っ取り早いと思います。

サーバーサイドレンダリングを行う場合は諸々事情が変わります。詳しくは[このページ](https://pinia.vuejs.org/ssr/)。

### Vue.js devtools での使用

具体的に書けることがあまりなく申し訳ないのですが、とりあえず今までの Vuex と同じ使用感で使えることは確認できています。

目玉機能としてタイムトラベル（たぶん値の変更をあとから追跡できるみたいな機能だと思ってます）というのがあるのですが、まだサポートされていない部分が多かったりなど未知数です。

分かり次第追記します。

## よくわかっていないところ

### 状態監視するときの 2 つの記法

上でも書いたように、 `$subscribe` と `watch` の使い分け、および前者の詳しい挙動がよくわかっていません。

ガイドを見る限り `watch` のほうはオプショナル的な位置づけに見えるのですが、そうなると deep オプションは…？とか色々疑問がわきます（ディープコピーまわりは危険度が高いのでそもそも使わないようにしていたゆえに必要性はあまり感じていないですが）。

### 特定の state のみを監視する方法

普通に考えたら「監視対象と全然関係ないときにも `$subscribe` メソッドが毎度動いてしまう」というのは危険極まりないです。

```js
store.$subscribe(() => {
  // ここに書く処理の副作用をとても気にしないといけない
})
```

しかし、監視しているのは上記のようにストア全体ということになっています。

あまりストアを多用した実装を行ったことがなかったのでつい最近気がついたのですが、「こんなはずなくない…？？」となっていてまだ自分が何かを知らないだけの可能性がかなりあります。そういえば Vuex 時代でも watch は雰囲気で使っていたなと…。

### ストアのネスト

上でちらっと触れましたが、ストアをネストできる機能があります。

```js
import { useUserStore } from './user'

export const cartStore = defineStore('cart', {
  getters: {
    // ... other getters
    summary(state) {
      const user = useUserStore()

      return `Hi ${user.name}, you have ${state.list.length} items.`
    },
  },

  actions: {
    purchase() {
      const user = useUserStore()

      return apiPurchase(user.id, this.list)
    },
  },
})
```

概念としても記法としてもわかりづらい上に、どうやら呼び出すたびに毎度 `buildStoreToUse` がキックされるなどパフォーマンス的にもどうなのか、と言っておられる方もいました。

僕は今のところ使うつもりはありません。

### ストアの共有

ストアのネストと似た要領で、他のストアの getters や actions を使えたりする機能のようです。

「そんなことするくらいならストアを分けるのがやめたほうがいいのでは…？」と僕がすぐ思ってしまうくらいレベルが低いものであるはずがないので、おそらくこれもメリットを理解できていないだけと思われます。

### Composition API の外で使う mapXXX シリーズ

今後これを使う可能性は一切ないと思っているので、よくわかっていないけど別に放置でいいかなーというもの。

```js
export default {
  computed: {
    ...mapStores(useCartStore, useUserStore)
  },
}
```

おそらく Vuex 時代でコンポーネントにバインドするために使っていたヘルパー関数シリーズと同種の API と思われます。

## おわりに

あえて最後にこれを書くのですが、**ストアは使わないに越したことはない**です。ストアの使用リスクはグローバル変数を使うことの危険性と本質的に同じだからです。

なぜ純正のフロントエンドフレームワーク単体では状態管理がサポートされていないのかをよく考える必要があるでしょう。これはあくまでも「プラスの付加価値」であり、「本当に困ったときにだけ手を伸ばすべきもの」と僕は思っています。

とはいえ便利なライブラリを知っておくに越したことはないのも事実。もし Vue で状態管理をしたいなら、次からは Pinia はいかがでしょうか🍍
