---
title: Windows PC から削除していい不要ファイルの一覧 (数十GB~)
tags : [Windows]
---

会社支給のクソ雑魚 PC などを使っている場合、肥大化していくハードディスク使用量との戦いが大きな課題になります。

主にキャッシュ類など削除してよいファイルの中で比較的サイズの大きいものをこの記事にどんどん羅列していきます。

有用な情報と思われるものがある場合はお気軽にコメントくださいませ（ページ上部の GitHub のリンクをご利用ください!）。

## 空き容量を増やすために消していいファイル一覧

<div class="box-common box-info">
<ul>
<li>言うまでもなく一応バックアップしておきましょう<span style="font-size: 0.8em;">（削除対象をコピペしておくだけでも十分）</span></li>
<li>隠しファイルの表示は ON にしておく必要があるのはもちろん、[保護されたオペレーティングシステムファイルを表示しない] も OFF にして探してください（該当項目には記載してあります）</li>
<li>具体的な手順などは外部サイトのリンクを掲載し記事を簡略化しています</li>
</ul>
</div>

### Windows.edb

<pre class="language-no">C:\ProgramData\Microsoft\Search\Data\Applications\Windows</pre>

- 5GB～20GB
- Windowsの「検索」で使われるインデックス生成ファイルなので、これを削除すると当然該当機能に影響はあります。使ってないなら問題なし。
- 削除時は[この記事](https://infoacetech.net/ja/windows/windows-edb文件/)の手順を参考にするとよい

### pagefile.sys

<pre class="language-no">C:\</pre>

- 10GB～30GB
- 仮想メモリのスワッピングファイル
- [保護されたオペレーティングシステムファイルを表示しない] を OFF にしないと見えない
- 削除したい、無効化したい、サイズを減らしたいなどの手順は[こちら](https://itojisan.xyz/settings/25335/)

### hiberfil.sys

<pre class="language-no">C:\</pre>

- 10GB~20GB
- ハイバネーション（休止状態）移行時に作業内容を一時保存するときに使うファイル
- [保護されたオペレーティングシステムファイルを表示しない] を OFF にしないと見えない
- 無効化の手順参考記事は[こちら](https://www.partitionwizard.jp/partitionmagic/delete-hibernation-file-windows-10.html)

### OneNoteのキャッシュ

<pre class="language-no">C:\Users\[Username]\AppData\Local\Microsoft\OneNote\16.0\cache</pre>

- 数MB～数十GB
- cache/ ディレクトリごと手動で削除して特に問題なかった
- 16.0のところはバージョン的なものと思われるが、かなり長い間数字もディレクトリ名も変わっていない（今後は変わる可能性がある）

### Outlookのキャッシュ（.ost）

<pre class="language-no">C:\Users\[Username]\AppData\Local\Microsoft\Outlook\</pre>

- 数MB～数GB
- name@example.com.ost などのファイル
- メールアカウントのデータファイルなので、メールクライアント側で再度設定や再構築が必要になる可能性が高い
- あまりサイズは肥大化しないしすぐに同じくらいになることもあるので削除しなくてもよいかも

### Tempディレクトリ

<pre class="language-no">C:\Users\[Username]\AppData\Local\Temp</pre>

- 数百MB～数GB
- 記事末尾に書いた [ディスクのクリーンアップ] で [一時ファイル] を選択した場合に該当するようだが、全てが消えるわけではないようなので手動で削除する価値はある
- ディレクトリごと消すのはやめておきましょう

## ファイルじゃないけど空き容量確保のためにできる作業

- ゴミ箱を空にする（ゴミ箱に入れても容量としてのカウントは変わらない）
- [ディスクのクリーンアップ] を各ドライブすべてで行う（[システムファイルのクリーンアップ]でやるとなおよい）
