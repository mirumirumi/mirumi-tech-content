---
title: naturalWidth/Height で要素のレンダリングサイズではなく画像の実寸サイズを取得する
tags : [JavaScript, DOM]
publish: false
---

HTML 上から画像を指定してサイズを取得するとき、

```js
const imgSize = {
    width: document.getElementById("photo").width, 
    height: document.getElementById("photo").height,
}
```

とやりがちですが、対象が画像の場合はビュー上のレンダリングサイズになってしまいます。

![image-size-width-height](../images/image-size-width-height.png)

画像の実寸サイズを取得したい場合には `naturalWidth`/`naturalHeight` というプロパティが使える模様。  
参考：[MDN](https://developer.mozilla.org/en-US/docs/Web/API/HTMLImageElement/naturalWidth)

```js
const imgSize = {
    width: document.getElementById("photo").naturalWidth,
    height: document.getElementById("photo").naturalHeight,
}
```

![image-size-naturalWidth-naturalHeight](../images/image-size-naturalWidth-naturalHeight.png)

サイトアイコンの 312px が取得できていることがわかります。
