---
title: CloudWatch Logs Insights のクエリチートシート
tags : [AWS, CloudWatch Logs, Cheat Sheet]
---

CloudWatch Logs の「ログのインサイト」でログを見るときの自分用使い方メモです。

## 実際のログ調査例ごとのスニペット

上がってきた不具合に対して「あれを調べればよさそうだな」と検討がついているとき、どうやったら「あれ」をフィルターできるかのシーンごとに集めてみています。

### message 内本体から色々探す

```sh
| filter message.path like /\/user\/deposit/
| filter message.method like /GET/
```

`@message` だとログ本体の全体に対しての検索になりますが、`message.xxx` で特定の項目に対して検索が可能。

文字列の完全一致を見たい場合、ドキュメントにある演算子を見て `=` と書いてしまうとワークしないので注意してください（エラーも出ません）。正しくは `==` です。

```sh
# これは動かない
| filter message.path = "/user/deposit"

# これは動く
| filter message.path == "/user/deposit"

# ちなみにシングルクォーテーションでも OK です
| filter message.path == '/user/deposit'

# ダブルクォーテーションはエスケープできます
| filter message.error == "Error: \"Config\" object"
```

### 正規表現のオプション修飾子（フラグ）をつけたい

```sh
# Exception にも exception にもマッチ
| filter @message like /(?i)Exception/

# Exception でも exception でもないログを探す
| filter @message not like /(?i)Exception/
```

### 構造化ログの途中で配列を含むのをドットチェーンする

```sh
| filter requestParameters.items.0.id="abcde123"
```

## 機能的側面からのスニペット

ログの中身のフィルターの仕方というより、「もうちょっとこういう風に使いたいんだけど…」的なものをこちらに集めています。

### 表示結果の列を制御したい

デフォルトの状態では、クエリをかけた結果のうち一番見たいであろう message はインライン表示されてしまって俯瞰的に見づらいです。

これの解決策として、`display` コマンドで結果リストに好きな列を設けることができます。

```sh
fields @timestamp, @message, @logStream, @log
| display @timestamp, @message, message.user_id
```

これで各行ごとにユーザー ID も一緒に表示されることになります。

### 自動で解析されないログ構造をクエリ内で使用する

CloudWatch Logs Insights は、AWS 内から入ってくるほとんどのログタイプに標準で対応しています。なので `message` 内にある構造化されたログにもそのままアクセスできるわけですが、それに対応していない場合でも手動でパースすることができます。

また、もともと構造化されていないアクセスログ（例えば Apache や nginx など）の場合はまずはこの `parse` を行ってからでないと各種フィルターがだいぶやりづらいことになると思います。

例えば次のような Apache のログがあったとすると、

```jsonc
106.37.25.151 - - [11/Mar/2023:04:07:55 +0900] "POST /posts HTTP/1.1" 200 0 "https://gorira.macho/uhoho" "xxxxx" "-"
```

このようにパースできます：

```sh
| parse @message '* * * [*] "* * *" * "*" "*"' as ip, user, remoteUser, timestamp, method, requestUri, protocol, statusCode, transferredData, referer, ua
```

パースしたあとはそれを fileds として扱えるようになります。

ちなみに名前衝突性の観点からか、`@id` というフィールドは新たに生成することはできないので注意してください。
