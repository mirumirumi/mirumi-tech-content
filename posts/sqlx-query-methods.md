---
title: SQLx (Rust) の query(), query_as(), query!, query_as! の違い
tags : [Rust, SQLx]
---

Rust の [SQLx](https://github.com/launchbadge/sqlx) でクエリしたいとき、パッと見の選択肢がいくつもあってどれを使うべきかの判断がとても難しい。実際に書いてある程度慣れてきても何度もわからなくなったりするので、このメモはそのまま需要があるかもと思った。とりあえず未加工で公開してみます。

事前情報として読んでおくべきページは、

- [概要説明](https://github.com/launchbadge/sqlx#querying)
- マクロシリーズの説明
    - [query!](https://docs.rs/sqlx/latest/sqlx/macro.query.html)
    - [query_as!](https://docs.rs/sqlx/latest/sqlx/macro.query_as.html)
- [オフラインモードの話](https://github.com/launchbadge/sqlx/blob/main/sqlx-cli/README.md#enable-building-in-offline-mode-with-query)
- [よくある質問](https://github.com/launchbadge/sqlx/blob/main/FAQ.md)

です。あと例によって [examples](https://github.com/launchbadge/sqlx/tree/main/examples) もいっぱいあります。

- `query_with`, `query_as_with` シリーズは使いづらいのでまず検討候補から外してよさそう
- ほとんど `query()` か `query!` でいける
    - `query()` は commit 系、`query!` は select 系という使い分け
    - commit して返り値を受けたいときは `query!` を使うとよい（examples にもある）
    - 込み入った型の定義や select 系クエリ後そのままその型の実装を使いたいようなシーンでは独自型を定義できている分 `query_as!` を使うとちょうどハマる
    - `query_as()` は唯一わかりやすい使い道が思いついてない…
- マクロじゃない 2 つはいわゆる普通な感じ
    - 変数のバインドは 1 つずつ書く必要がある（行数が増える）
    - `query_as()` は返り値の型を指定できる
        - `query()` でも map を使ったコントロールは可能だが少し読みづらくなる（主観）
- マクロの 2 つはコンパイル時チェックがある
    - これがオフラインモードの話（別にオフラインモードじゃなくてもいいけど）
    - そのうちの 2 つの違いは明確に型を指定するか匿名型かのみと思う
