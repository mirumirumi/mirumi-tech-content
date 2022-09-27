---
title: Lambda + Python で ffmpeg を動かす (Pydub)
tags : [Lambda, Python, ffmpeg, Pydub]
---

どうしても Lambda で ffmpeg を動かさないといけないことがあって、こねくり回したら一応なんとかなったのでメモっておく。

## 使うもの

- Lambda Layer
    - 依存関係類は全部レイヤーにつっこむ
- ffmpeg
    - 動かしたい目標
    - 今回はビルドしたバイナリをそのまんま詰め込む
- ffprobe
    - やりたいことによってはこいつも必要になることがある、自分は [Pydub](https://github.com/jiaaro/pydub) で音声ファイルの変換をしたかったから ffmpeg が必要になった、その際この ffprobe というのもセットで扱った
    - こちらもビルドバイナリを使う

## バイナリ

ffmpeg は[これ](https://johnvansickle.com/ffmpeg/)、ffprobe は[これ](https://ffbinaries.com/downloads)を使う。

もちろん Lambda の実行ランタイムに合わせたものを選びます。特に指定しない場合やデフォルトの CloudFormation テンプレートを使う場合は `x86_64` で、つまりはそれぞれ「ffmpeg-git-amd64-static.tar.xz」や「linux-64」を選ぶことになります。

```yaml
MyFunction:
  Type: AWS::Serverless::Function
  Properties:
    FunctionName: my-function
    Handler: app.lambda_handler
    Runtime: python3.9
    Architectures:
      - x86_64  # <-- ここ
```

## レイヤーをつくる

あとはこのバイナリを `bin/` ディレクトリに入れた状態でレイヤーをつくればよい。

ディレクトリ構成は

```bash
layers/
├── ffmpeg_layer/
│   └── bin/
│       └── ffmpeg
└── ffprobe_layer/
    └── bin/
        └── ffprobe
```

となります。

レイヤーの定義は、

```yaml
FfmpegLayer:
  Type: AWS::Serverless::LayerVersion
  Properties:
    ContentUri: "project_root/layers/ffmpeg_layer"
    CompatibleRuntimes:
      - python3.8
      - python3.9
FfprobeLayer:
  Type: AWS::Serverless::LayerVersion
  Properties:
    ContentUri: "project_root/layers/ffprobe_layer"
    CompatibleRuntimes:
      - python3.8
      - python3.9
```

という感じになれば OK。

あとは Lambda を起動すればランタイムからバイナリを実行できるようになっているはずです。
