---
title: WSL2 Ubuntu で Electron 起動時に共有ライブラリがないとか色々怒られる
tags : [WSL2, Electron]
---
どうやら Linux としてもともと共有されていることが前提らしいライブラリを `/node_modules/electron/dist/electron` が多く必要としているらしく、それが OS によっては全然なかったりするらしい。

WSL2 で主にこの報告が多いみたいだけど、ディストリビューションは Ubuntu 以外にも CentOS (たぶん7) であったり、そもそも WSL じゃなかったりもするみたい。

自分の環境は以下。

- WSL2
- Ubuntu-20.04
- Windows 10 [Version 10.0.19044.1387]

僕の場合は下記をインストールしたら必要なものが全て揃いました。

```shell
sudo apt install -y libnss3-dev libatk1.0-0 libatk-bridge2.0-0 libgdk-pixbuf2.0-0 libgtk-3-0 libgbm-dev
```

GUI のレンダリングや Chromium 関係の依存ライブラリ…っぽいです。
