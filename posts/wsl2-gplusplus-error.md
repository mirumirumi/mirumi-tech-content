---
title: WSL2 の CentOS で npm install 中に make: g++: Command not found
tags : [WSL2]
---

標準で開発ツール類が入っていないことが問題のよう。

gcc などは入っていると思っていても色々足りないものがあったりするみたいなので、

```bash
sudo yum groupinstall 'Development Tools'
```

でまとめて入れる。これでだいたい通る。

参考：[https://unix.stackexchange.com/questions/140350/linux-g-command-not-found/140355](https://unix.stackexchange.com/questions/140350/linux-g-command-not-found/140355)
