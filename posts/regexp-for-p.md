---
title: 改行コード入りの文字列群を <p> タグの段落フォーマットに変換する正規表現
tags : [正規表現]
---

こうする。

```ts
const formatContent = (content: string): string => {
  return content
    .replace(/(([^\r\n]+(\r?\n)?)+)/gim, "<p>$1</p>")
    .replaceAll(/\r?\n([^<])/gim, "<br />$1")
}
```

これは外部ユーザーによって投稿されたコメントを整形するために書いたのだけど、その中でリンク文字列を自動的に a タグ変換したかったので下記も追加した。

```ts
const formatContent = (content: string): string => {
  return content
    .replace(/(([^\r\n]+(\r?\n)?)+)/gim, "<p>$1</p>")
    .replaceAll(/\r?\n([^<])/gim, "<br />$1")
    // 下記を追加
    .replaceAll(
      /((<p>)|<br \/>)?(https?:\/\/[\w\/:;%#\$&\?\(\)~\.=\+\-]+)(\r?\n)?((<\/p>)|<br \/>)/gim,
      '$1<a href="$3" rel="nofollow ugc">$3</a>$5'
    )
}
```
