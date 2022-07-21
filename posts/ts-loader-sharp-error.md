---
title: ts-loader で Cannot find module Error になるのはパス内に # があったから
tags : [ts-loader]
---

ts-loader を使ってビルドをする際、何度環境を確認しても

```shell
ERROR in ../../#dev/repo-name/index.ts
Module build failed (from ../../#dev/repo-name/node_modules/ts-loader/index.js):
Error: Cannot find module '/home/Username/#dev/repo-name/node_modules/ts-loader/index.js'
Require stack:
- /home/Username/#dev/repo-name/node_modules/loader-runner/lib/loadLoader.js
- /home/Username/#dev/repo-name/node_modules/loader-runner/lib/LoaderRunner.js
- /home/Username/#dev/repo-name/node_modules/webpack/lib/NormalModule.js
- /home/Username/#dev/repo-name/node_modules/webpack/lib/NormalModuleFactory.js
- /home/Username/#dev/repo-name/node_modules/webpack/lib/Compiler.js
- /home/Username/#dev/repo-name/node_modules/webpack/lib/webpack.js
- /home/Username/#dev/repo-name/node_modules/webpack/lib/index.js
- /home/Username/#dev/repo-name/node_modules/webpack-cli/lib/webpack-cli.js
- /home/Username/#dev/repo-name/node_modules/webpack-cli/lib/bootstrap.js
- /home/Username/#dev/repo-name/node_modules/webpack-cli/bin/cli.js
- /home/Username/#dev/repo-name/node_modules/webpack/bin/webpack.js
    at Function.Module._resolveFilename (node:internal/modules/cjs/loader:927:15)
    at Function.Module._load (node:internal/modules/cjs/loader:772:27)
    at Module.require (node:internal/modules/cjs/loader:999:19)
    at require (/home/Username/#dev/repo-name/node_modules/v8-compile-cache/v8-compile-cache.js:159:20)
    at loadLoader (/home/Username/#dev/repo-name/node_modules/loader-runner/lib/loadLoader.js:19:17)
    at iteratePitchingLoaders (/home/Username/#dev/repo-name/node_modules/loader-runner/lib/LoaderRunner.js:182:2)
    at runLoaders (/home/Username/#dev/repo-name/node_modules/loader-runner/lib/LoaderRunner.js:397:2)
    at NormalModule.doBuild (/home/Username/#dev/repo-name/node_modules/webpack/lib/NormalModule.js:743:3)
    at NormalModule.build (/home/Username/#dev/repo-name/node_modules/webpack/lib/NormalModule.js:892:15)
    at /home/Username/#dev/repo-name/node_modules/webpack/lib/Compilation.js:1311:12
```

というエラーが出てどうにもならない問題がありました。1日半ほど費やしました。

**もしやと思いディレクトリ名に含まれていた `#` を削除したら動いた…！**

もちろんディレクトリ名に通常の文字（ `[a-zA-Z_0-9]` あたり）以外を使ったらいけないかも…なんてのは考えた上でこうしていたし、事実これまで `#` が異変を起こしたことは一度もありませんでした。

「色んな OS が通常通り認識できる文字列種別」なんてのも探してみましたし、局所的なアンラッキーだったなーという感想です。でもこういうことが実際にあるんだからやっぱり変なことはしないべきですね（でもディレクトリ一覧で上の方に持ってきたいとかありません？）。

この件を ts-loader のリポジトリに issue として上げてみましたが、同様の問題が webpack の issue として既に上げられていたというコメントをもらっています。

[https://github.com/TypeStrong/ts-loader/issues/1390]

どなたか参考になれば（いるわけない）幸いです。
