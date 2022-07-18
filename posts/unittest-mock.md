---
title: pytest/unittest で「オブジェクト指定しづらい関数」をモックに置き換える
tags : [pytest, Python, unittest]
---

Python のユニットテストにおいて関数のモック化をあとから考えることになってしまったとき、既存の設計のままでそこそこなテストを書く方法があることを知ったので残しておきます。

要は**インポート系の問題やクラス化されていない関数指定などの問題を回避できる**よ、という話です。

## あてはまる状況とやりたいこと

例えばこんなディレクトリ構成で、

```bash
root
 └─ connection.py 
 └─ cake.py
```

cake.py はこんなシーンだったとする。

```python
import connection

def make_cake():
  dish = []
  if has_strawberry(my_fridge):
    dish.append(my_fridge["berry"])
  return dish

def has_strawberry(fridge):
  conn = connect(fridge)
  return "berry" in conn
```

`make_cake()` の単体テストを考えているとします。

冷蔵庫(fridge) がハイテクすぎるので、中にイチゴがあるか確認するためにはネットワークコネクションを握る必要があります。そしてこのとき使う `connect` メソッドは環境依存があるためにローカルの開発環境では失敗するとしましょう。

こうなると

- `has_strawberry()`
- `connect()`

のどちらかをモックにする必要がありますが、今回は `has_strawberry()` をダミー化します。 `make_cake()` の単体テストなので、他との依存はなるべく疎にするべきです。

こうなったときに取り得る選択肢は

- `unittest.mock` の `Mock()` および `MagicMock()`
- `monkeypatch` の `setattr()`

などですが、**これらはオブジェクトとしてモック対象を識別できるようになっていないとモックとして指定ができない**のが普通です。関数名を文字列で直接指定したりできないってことですね。

そんなわけで、

- クラスとクラスメソッドの関係になっていなかった！
- プロダクションコードを含むファイルをモジュールごとインポートできればいいけど、色々事情があってそれは無理！

みたいなときが想定シーンということになります。

## unittest.mock モジュールの patch を使う

標準パッケージの unittest 内にある mock モジュール、その中の patch メソッドを使います。

- 公式ドキュメント：[unittest.mock --- patch()](https://docs.python.org/ja/3/library/unittest.mock.html#unittest.mock.patch)
- unittest.mock は Python 3.3 以降で追加

```python
>>> from unittest.mock import patch
>>> patch
<function patch at 0x00000242CB7DF288>
```

メソッドと言ってますが実際はデコレータとして使います。  
<span style="font-size: 0.8em; color: #999999;">※ `patch.object()` のように普通のメソッドとしても使えますがそれは今回の趣旨に相対する方の使い方なので割愛</span>

```python
from .. import cake
from unittest.mock import patch

@patch("cake.has_strawberry")
def test_make_cake():
  result = make_cake()
  assert result == ["berry"]
```

```bash
root
 └─ connection.py
 └─ cake.py
 └─ tests
      └─ test_cake.py
```

そう、この例で分かるように**モックに置き換えたい****関数の指定を文字列で指定できるので実際にインポートなどが発生しない**点が魅力です<span style="font-size: 0.8em;">（今回はミニマムな例の都合上 `cake` をインポートしちゃってますが）</span>。

とはいえ、一応はオブジェクト指定（ドット繋ぎ）の表現がされていることは前提です。

上記を

```python
@patch("has_strawberry")  # <- "cake." をなくした
def test_make_cake():
  result = make_cake()
  assert result == ["berry"]
```

のようにしてもモック対象として識別はされない。

今回はテスト対象コードを含むモジュールの中にモック対象も存在していたので恩恵があまりないけど、他のなんやかんやと色々事情があるときにはとても助かる。 `monkeypatch` の `setattr()` より感覚的にも使いやすいです。

### モック関数の使い方

`@patch` でデコレータをつけると、デコレートされたテスト関数の引数には自動的にモック化されたオブジェクトが渡されてきます。

```python
@patch("cake.has_strawberry")
def test_make_cake(mock_func):
  mock_func.return_value = True
  result = make_cake()
  assert result == ["berry"]
```

モックオブジェクトのメンバとして

- `return_value`
- `side_effect`

などがあるので、これを指定することで任意の返り値を制御できます。他に何があるかは [ここ](https://docs.python.org/ja/3/library/unittest.mock.html#unittest.mock.Mock) を参照。

デコレータの第二引数以降で指定することも可能です。

```python
@patch("cake.has_strawberry", return_value=True)
def test_make_cake():
  result = make_cake()
  assert result == ["berry"]
```

### for 文の中で呼ばれるから毎回違う値を返したいんですけど…

テスト対象コードがこんなんに変わったとしましょう。

```python
def make_cake():
  dish = []
  for i in range(len(my_strawberry)):
    if has_strawberry(my_fridge):
      dish.append(my_fridge["berry"])
    else:
      if i == 3:
        print("アホ")
  return dish
```

つまり**指定の値は返してほしいんだけど、条件網羅のために繰り返し回数によって返却値を変えたい**という場合です。

これは `side_effect` をリストにすることで解決できます。

```python
@patch("cake.has_strawberry")
def test_make_cake(mock_func):
  mock_func.side_effect = [True, True, True, False]
  result = make_cake()
  assert result == ["berry"]
```

このような指定にすると、モック対象である `has_strawberry()` が呼ばれるたびに返り値がリストの各要素になってくれるということ。

### 引数に応じて返り値変えられん？

これは `patch` の話というよりどちらかというと `MagicMock` の話だけど、ちゃんとできます。

```python
@patch("cake.has_strawberry")
def test_make_cake(mock_func):
  def hoge(num):
    return num + 3
  mock_func.side_effect = hoge
  result = make_cake()
  assert result == ["berry"]
```

`side_effect` には関数も指定できるので、モックとして引数で受け取ってきた関数の返り値をその別の関数にすれば OK 。今回の例だと本当の引数は冷蔵庫のはずだけども。

ちなみに `Mock()` と `MagicMock()` はほぼ同じものだそうで、公式は「今は特に気にせず `MagicMock()` を使えばいいよ」と[言っています](https://docs.python.org/ja/3/library/unittest.mock-examples.html#:~:text=%E3%81%93%E3%81%AE%E4%BE%8B%E3%81%AE%E3%82%88%E3%81%86%E3%81%AA%E5%A0%B4%E5%90%88%E3%80%81%E3%81%9F%E3%81%84%E3%81%A6%E3%81%84%20Mock%20%E3%81%A8%20MagicMock%20%E3%81%AF%E4%BA%92%E6%8F%9B%E3%81%A7%E3%81%99%E3%80%82%20MagicMock%20%E3%81%AE%E6%96%B9%E3%81%8C%E5%BC%B7%E5%8A%9B%E3%81%AA%E3%81%AE%E3%81%A7%E3%80%81%E3%83%87%E3%83%95%E3%82%A9%E3%83%AB%E3%83%88%E3%81%A7%E3%81%AF%E3%81%93%E3%81%A1%E3%82%89%E3%82%92%E4%BD%BF%E3%81%86%E3%81%A8%E3%81%84%E3%81%84%E3%81%A7%E3%81%97%E3%82%87%E3%81%86%E3%80%82)。

### 例外の送出もしたいなあ！

当然返り値に例外を送出させたいときもあるはず。

シンプルにこれで OK 。

```python
mock_func.side_effect = Exption()
```

もちろん前述のリスト化と併用できます。
