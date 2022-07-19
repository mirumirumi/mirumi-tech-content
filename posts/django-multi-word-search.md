---
title: Django で「1文字ずつ分解されない複数キーワード検索」をシンプルに実装する
tags : [Python, Django]
---

Django で通常の GET リクエストによるフリーワード検索をしたいとき、意外にも簡素な実装が望めません。  
外部のライブラリなどは使わない自然なフリーワード検索を考えてみます。

## 部分一致検索

まずは「部分一致検索」対応のために `Qオブジェクト` + `queryset.filter` を利用します。  
（本来ならこの時点で Q オブジェクトはまだ必要ありませんが、便宜上同じタイミングで説明しています）

```python
from django.db.models import Q
from .models import Article

class IndexView(generic.ListView):
    model = Article

    def get_queryset(self):
        queryset = Aritcle.objects.all()
        keyword = self.request.GET.get("search-form")
        if keyword:
            queryset = queryset.filter(Q(title__icontains=keyword))
```

これで Article というモデルの title というフィールドに特定のキーワードが含まれるオブジェクトのみを扱えます。

:::info
`__icontains` というプロパティは Q オブジェクトが持つプロパティではありません（そのような記載がなされているサイトがありました）。もともと queryset が持っているルックアップです。  
参考：[Django公式リファレンス](https://docs.djangoproject.com/en/2.2/ref/models/querysets/#id4)
:::

ちなみに i は case-insensitive の i なので、 `__contains` とすれば大文字小文字を区別する厳密な絞り込みが可能です。

## 複数のフィールドにまたがる検索

上記を踏襲しつつ、title 以外のフィールドにも検索範囲を広げてみます。ここでは content としてみましょう。

```python
def get_queryset(self):
    queryset = Aritcle.objects.all()
    keyword = self.request.GET.get("search-form")
    if keyword:
        queryset = queryset.filter(
                    Q(title__icontains=keyword) | 
                    Q(content__icontains=keyword)
                  )
```

これは簡単です。  
Q オブジェクトによって既に OR 検索が可能になっているので、filter メソッドの引数を `|` (パイプ) で繋げていくだけです。

あとはこの状態で複数ワードで区切られたものごとに filter をその都度かけていくのが自然な書き方になりそうですね。

## 区切り文字でキーワードを分けて繰り返すだけ

というわけで、シンプルにキーワードを区切って区切られたワードごとに for 文で filter を適用させてみます。

区切り文字があるときは複数ワード検索できるような対応に変えてみましょう。

多くのユーザーが無意識に複数ワードを入力するときはほぼほぼ全角か半角のスペースで区切るでしょうから、下記のコードもそれに則っています。もちろん適宜好きな区切り文字を追加できます。

:::info
今は分かりやすくするため、一旦 if 節と else 節で分けています。
:::

```python
import re

def get_queryset(self):
    queryset = Aritcle.objects.all()
    keyword = self.request.GET.get("search-form")
    if keyword:
        if re.search("\s", keyword):
            keywords = keyword.split()
            for k in keywords:
                queryset = queryset.filter(
                            Q(title__icontains=k) | 
                            Q(content__icontains=k)
                          )
        else:
            queryset = queryset.filter(
                        Q(title__icontains=keyword) | 
                        Q(content__icontains=keyword)
                      )
```

split() で分割されたキーワードひとつずつに対して filter() が効いています。Q オブジェクトの引数の値をリストの各要素（ `k` ）に置き換えるのを忘れずに。

で、これをまとめます。

```python
def get_queryset(self):
    queryset = Aritcle.objects.all()
    keyword = self.request.GET.get("search-form")
    if keyword:
        keywords = keyword.split()
        for k in keywords:
            queryset = queryset.filter(
                        Q(title__icontains=k) | 
                        Q(content__icontains=k)
                      )
```

区切り文字が含まれていない文字列を `split()` しようが for 文的には問題ないのでこれで大丈夫そうです。  
区切り文字を指定したい場合は `split()` の引数を利用しましょう。
