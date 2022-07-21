---
title: Electron の Windows 用ビルド (NSIS) でインストーラーを高解像度 DPI 対応する方法
tags : [Electron]
---

[electron-builder](https://github.com/electron-userland/electron-builder) を使っている場合の話です。

Electron で Windows 用の NSIS ビルドを行うと、デフォルトで表示されるインストーラー画面の解像度は著しく低いです。ユーザーには一瞬しか見られない画面とはいえ、アプリケーションを使う前のファーストインプレッションとしては絶対に改善したいです。

electron-builder のリファレンスを見る限りそのような記述は見つけられなかったのですが、NSIS の設定として一般的に取り扱われる要素として設定が可能でした。

「installer.nsh」という名前でファイルをつくり、これを `build` ディレクトリに置きます。ここでいう `build` ディレクトリは、electron-builder のビルド用アセットを置いている場所です。特に指定をしていない限りこの名前で使用しているでしょう。ない場合はプロジェクトのルートに mkdir します。

中身の記述はこうです：

```plain
ManifestDPIAware true
```

この 1 行を書いて保存しておくだけで、ビルド時に勝手に反応し対応がなされます。

<p><img src="https://mirumi.me/tech/wp-content/uploads/electron-builder-installer.png" alt="electron-builder-installer" class="win11_ss" /></p>

できあがったインストーラー画面はちゃんとクッキリしており、アプリアイコンもジャギーさから解放されました🎉
