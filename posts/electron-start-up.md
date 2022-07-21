---
title: Electron で Windows/Mac のスタートアップ登録をする
tags : [Electron]
---

Electron の豊富な設定項目と丁寧なリファレンスには絶対に存在するかと思われた「スタートアップ登録」ですが、「startup」というワードでは一切ヒットしないのでややわかりづらいです。

実際にはこれが該当します。

[https://www.electronjs.org/ja/docs/latest/api/app#appsetloginitemsettingssettings-macos-windows]

メインプロセスモジュールの `app` が持つメソッドとして `setLoginItemSettings()` があります。

基本形はこうです：

```js
app.setLoginItemSettings({
  openAtLogin: true,
})
```

この他にもプロパティは色々あるものの、基本はこれを設定するだけであとはうまいことやってくれます。Windows/Mac でそれぞれ固有の設定値もあるので詳しくはリファレンスを参照してください。

なお、スタートアップ登録はユーザーが OS からも自由に変更できるために「変更を適用する前に今の設定状態を確認しましょう」というようなことが公式からもガイドされています。

これには `getLoginItemSettings` を使います。そのままですね。

[https://www.electronjs.org/ja/docs/latest/api/app#appgetloginitemsettingsoptions-macos-windows]

同じく `openAtLogin` から真偽値が取得できるのでこれを使いましょう。

```js
const res = app.getLoginItemSettings()
console.log(res.openAtLogin)  // true
```

多くのアプリケーションではスタートアップ登録をチェックボックスやトグルスイッチで実現しているでしょうから、設定画面を開くときに毎度これをチェックしてあらかじめ反映させておくのがよさそうです。

ちなみに2016年（v1.2.7 ）ごろまではこの API はなく、[auto-launch](https://www.npmjs.com/package/auto-launch) という外部パッケージを使うのが一般的だったそうです。たぶん初期リリースは[この回](https://github.com/electron/electron/releases/tag/v1.2.7)。

スタートアップの要望が issue として登録されて以降、Electron 本体側で実装され組み込まれたらしい経緯が [このスレッド](https://github.com/electron-archive/grunt-electron-installer/issues/115) で確認できます。
