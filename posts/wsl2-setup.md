---
title: WSL 2 を初期化してからセットアップでやったこととその順番
tags : [WSL 2]
---

久しぶりにメインマシンを一新したので必然的に WSL 2 も最初からセットアップすることになった。いい機会なので WSL 2 内のベース環境を整える手順をメモっておく。

Windows 11 から WSL 2 関係の立ち位置や処遇もだいぶ（いい方向に）変わっているが、入れ方が多少変わるだけで中身のセットアップには何も関係はないのでその辺は省いている。

ディストリビューションは [Ubuntu 22.04.2 LTS](https://www.microsoft.com/store/productid/9PN20MSR04DW?ocid=pdpshare) です。

1. `sudo apt update && sudo apt upgrade`
2. `sudo apt install build-essential openssl libssl-dev pkg-config`
    - ビルド環境のセッティング
    - build-essential は gcc とか含んでいる
    - `sudo apt install unzip` もやっておくとよい
3. Python 系
    - 自分の用途的には pyenv だけあればよい
    - なんやかんややる前に必ず仮想環境シリーズを入れる、壊れた Python 環境ほどどうしようもないものはない
4. docker いれる
    - Docker Desktop のこと
5. Terraform いれる
    - tfenv いれてから
6. その他各プログラミング言語シリーズ
    - rustup いれる
    - node いれる
        - n いれてから
7. AWS CLI いれる
8. SAM いれる
9. <span class="keyboard-key">Tab</span> キーとかでポーンと鳴らないようにする
    - `/etc/inputrc` にある `# set bell-style none` となっている部分のコメントアウトを解除する
10. 自前エイリアスの登録
    - `ls` 系（もとから入っているのがあるので注意）
    - `cd ../` 系
11. Git 系の設定
    - `~/.gitconfig` とか
    - .gitignore_global とか
        - GUI の Git クライアントを使っている場合は Windows 側で管理する必要があるので注意（そのクライアントがどの Git を向いているかにもよるけど）
