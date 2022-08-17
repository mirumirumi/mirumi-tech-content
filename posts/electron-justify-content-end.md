---
title: Electron では justify-content: end は効かないので flex-end にしよう
tags : [Electron, CSS]
---

Electron のバージョン 13.6.9 で確認しました。

もともと `justify-content: end` はフレックスボックス内のアイテム以外にも適用されるやや広めな仕様であり、その実装はレンダラーに差があると言われていました。

対して `flex-end` はフレックスボックス内のアイテム専用のプロパティ（値）のため、より安全でありなおかつ将来的にもこちらをセットしておくほうが安心だ、とされています。

[https://www.w3.org/TR/css-align-3/#positional-values]

とはいえ Electron は実質 Chromium そのまんまだし、ブラウザ上で開発しているものと単純な CSS で差分が出るとは思いませんでした。

今後は普段の CSS でも `flex-end` を使っていくことを意識します。
