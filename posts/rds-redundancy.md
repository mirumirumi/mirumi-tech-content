---
title: Amazon RDS の冗長化に関する各要素のまとめ
tags : [AWS, RDS]
---

RDS の冗長化に関する設定項目は種類が多い上に各要素同士が関連し合っており、さらには DB のエンジンタイプやインスタンスタイプのサーバーレス可否などによっても事情が変動し、総じて理解が難しいです。なによりまとまった俯瞰的な情報というものがないのが困ります。

というわけでこの記事ではこれまでに得た情報をメモ的に羅列し、できそうならわかりやすいまとめ作成を試みるものです。

:::info
いまのところ実際に手を動かして正しく理解できていなさそうなものも混じっているため、メモ的な羅列に留めてあります。
:::

## 登場する要素

### 標準エンジンタイプ

RDS の基本的な DB エンジン互換性タイプについての項目。一部 MySQL と PostgreSQL しか対応されていないものもあります（マルチAZ DB クラスター）。

#### マルチAZ DB インスタンス

- 2 つの AZ にプライマリインスタンスとスタンバイレプリカを置く
- スタンパイレプリカへは同期レプリケーション
- 読み取りスケーラビリティ向上のためのものとは別（リードはできない）
- 自動フェイルオーバーあり

#### リードレプリカ

- 同じリージョンまたは異なるリージョンに最大 5 台のリードレプリカを置ける
    - 同じリージョンのときは異なる AZ へ配置される
- プライマリ以外には非同期レプリケーションされる
- リードもできる（＝スケーラビリティ向上としての側面もある）
- 自動フェイルオーバーなし
    - 手動でプライマリインスタンスに昇格させることはできる
    - エンドポイントは変わってしまうのでアプリケーション側などにも何らかの仕組みが必要
- マルチAZ DB インスタンスと併用できる（計 3 台になる）
    - リードレプリカをマルチAZ DB インスタンスとすることもできて、この場合はリードレプリカをプライマリに昇格させたあとも楽

#### マルチAZ DB クラスター

- プライマリインスタンス（ライター）とは別の AZ 2 つにリーダーインスタンスを配置する
- 自動フェイルオーバーあり
- わかりやすいし可用性アップとスケーラビリティアップの両方取れる
- ただし制限が多い（下記は一部です、詳細は[こちら](https://docs.aws.amazon.com/ja_jp/AmazonRDS/latest/UserGuide/multi-az-db-clusters-concepts.html#multi-az-db-clusters-concepts.Limitations)）
    - MySQL と PostgreSQL のみ
    - RDS Proxy が使えない
    - リードレプリカは作れない
    - リザーブドインスタンスは使えない

### RDS Proxy

これまで書いてきた内容とはちょっとだけ毛色が違いますが、これも DB 本体への負荷を下げるものという意味で一応書いておきます。

- RDS へのコネクションをプールしていい感じに使い回してくれるやつ
    - Lambda から RDS を使う場合は相性がいい
- エンドポイントを RDS Proxy 用のものに変えるだけで簡単に導入できる
- DB 側は何も変わらないので冗長性アップのためのものではない
- Aurora Serverless (v1) では使えなかったが v2 から対応した
- デメリット：
    - 「固定」の発生可能性（昔は「ピン留め」とか言われていた）
    - 単純にお金がかかる（RDS Proxy を使った結果 DB 側のスケールダウンやインスタンス削減などによって合計の運用費が下がることはある）

### Aurora 系

MySQL と PostgreSQL に限っては「AWS が RDS 用にチューニングした」とされている、より最適化されたエンジンタイプが選択できます。この 2 つは Aurora XXX と呼ばれ、上記までの通常のエンジンタイプとは区別されます。

で、この Aurora 系はこれまで書いてきた各冗長化に関する要素への扱いがだいぶ異なっているので別枠でまとめています。

#### Aurora の基本的な仕組みが提供する可用性

- プライマリ＋最大 15 台のリードレプリカ的なものを作れる
    - 公式的にはリードレプリカとは別物で、「リーダーインスタンス」などと呼ばれている
    - 「複数の AZ にまたがる」ことにはなっているが、これがどう決まるのかがまだ分かっていない（たぶん指定はできないはず）
- Aurora の基本的な仕組みとして、インスタンスとストレージが分離されていることをまず理解する
    - データは 3 つの AZ で各 2 つずつのストレージに保管されており（計 6 個）、自動的にレプリケーションされる（共有分散ストレージボリューム）
    - さらに S3 にバックアップされるのでこの時点で耐久性だけでいえば 99.999999999% になる
- 自動フェイルオーバーあり
    - 0 ~ 15 までで振った優先度に応じて選ばれる

#### Aurora Global Database

- Aurora クラスター自体をマルチリージョンするもの
    - プライマリクラスターの他に最大 5 つのセカンダリクラスターを作成可能
    - 「リージョンまるごと落ちたとしても 1 分で復旧できるよ」としている
- レプリケーションあり（同期？非同期？）
- フェイルオーバーは自動？
- セカンダリクラスターはリーダブルなのでスケーラビリティアップにもなる

#### サーバーレスタイプについて

- v2 でつくるプロビジョンドクラスターというのは、上記で書いた普通の Aurora クラスターと同じもの（v1 のサーバーレスクラスターはブラックボックスの別物）
- ちなみに v2 なら他の普通のインスタンスサイズと混在してクラスターを作れる

## 個人的まとめ

- Aurora Serverless v2 を使う
- ライターとリーダーで 1 つずつくらいまではマルチAZ する
    - ワークロード的に無理がありそうならスケールアップするかインスタンス増やすかその都度考える
- RDS Proxy は基本使わない
    - 色々細かい問題に遭遇したことがあるのでパフォーマンスで困っていない限りは使わない方針
- これで安くて早くて速くて楽ちん

かな！