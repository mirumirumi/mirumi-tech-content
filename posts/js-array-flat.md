---
title: JS でネストされた配列の全ての要素数を一発で数える：Array.prototype.flat()
tags : [JavaScript]
---

複数層でネストされた配列内にある要素（つまり 葉 にあたる部分）の数を数えたいとき、JavaScript では一発で算出できることをいまさらながら知ったのでメモ。

```js
const array = [
    1,
    2,
    [3, 4],
    5,
    [6, [7, 8], 9, [10]],
]

console.log(array.flat(2).length)  // 10
```

`Array.prototype.flat()` というとても便利なメソッドがありました（[MDN](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/flat)）。いいですね。

引数に「深さ」を指定できて、指定しないと 1 になってしまう（つまり 1 階層目までしかフラット化されない）のでそこは注意。大きい値を指定してもエラーは出ないので、深さが動的でなおかつ書き捨てのスクリプトとかの場合は 100 とかやっちゃってもいいかも。
