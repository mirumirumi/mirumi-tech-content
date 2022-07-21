---
title: Twitter の API を Python製「TwitterAPI」ライブラリで手軽に利用する
tags : [Python, Twitter]
---

Twitter の API を初めて使おうとしたとき思いのほか情報がなくて驚いたのに加え、Python においては使いやすい手頃なソリューション的なものも全然なさそうだったので記録しておきます。

## 使うもの

[Twitter Developer](https://developer.twitter.com/en) の公式サイトを漁っていたら「各言語によるライブラリ集」的なやつを見つけました。以下。

[https://developer.twitter.com/en/docs/twitter-api/tools-and-libraries]

今回は Python で実装を探していたのですが、 `requests` でベタ書きするだの `OAuth1Session` という OAuth 用のモジュールを使うだの色々見つかるわりには Twitter も標準的な実装とかは紹介してくれていなくて、そんな中公式の推奨？っぽいライブラリはかなり助かります。

選んだのはこれ。

[https://github.com/geduldig/TwitterAPI]

名前そのまんまなんですが、下記のように超わかりやすい最高にシンプルな記述が素敵。

```python
from TwitterAPI import TwitterAPI

api = TwitterAPI(
    consumer_key,
    consumer_secret,
    access_token_key,
    access_token_secret,
)
api.request('statuses/update', {'status':'This is a tweet!'})
```

見てわかる通り、OAuth 認証に使う例の４つのキー類もそのまま扱えて、次の行ではもうツイートのリクエストができちゃいます。まさにこういうものを探していたのだ。

インストールは `pip` でできます。

```bash
pip install TwitterAPI
```

基本的な使い方はこれだけ。

## 画像添付とか

```python
res = api.request(
    "media/upload",
    None,
    {"media": file},
)
media_id = res.json()["media_id"]
```

画像の添付は、 `request` タイプの指定を `media/upload` にしつつ第三引数にキー名を `media` として画像ファイルをあてがいます。

ファイルは普通にファイルオブジェクトやバイナリが許容される模様。

レスポンスの中に `media_id` というのがいるのでそれを取り出しておきます。 `json()` となっているのはこの requsets などと同じく TwitterAPI オブジェクトの独自メソッドなのでうっかり `json.loads(res["media_id"])` とかやらないように注意。

そしたら通常ツイートの実装をベースに画像を添付します。

```python
res = api.request("statuses/update", {
    "status": tweet_text,
    "media_ids": media_id,
})
```

`media_id` を付与するキー名は `media_ids` であることに注意してください。

その他基本的な API（主に v1.1 と呼ばれているやつ）は下記のディレクトリから適宜サンプル実装を拾いましょう。

[https://github.com/geduldig/TwitterAPI/tree/master/examples/v1.1]

最近の主流は v2 かのように思えるのですが、こちらはリファレンスを読んだ感じ自分が必要とすることはなさそうだったのでスルーしています。
