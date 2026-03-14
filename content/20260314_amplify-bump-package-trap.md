+++
title = "Amplify Gen2 の依存パッケージを無闇に更新してはいけない"
date = "2025-07-16"
description = "バージョン不整合の罠"
[taxonomies]
tags = ["amplifu", "cdk"]
+++

パッケージバージョンを全て最新化するとこのエラーが出る。
CDK のバージョンに amplify が付いていけてないことがある模様。

`Cloud assembly schema version mismatch: Maximum schema version supported is 44.x.x, but found 45.0.0`

今回は以下で対処。最新は 2.204.0 だがエラーが発生するため以下を指定した。

```json: package.json
    "aws-cdk-lib": "2.202.0",
```

足並み揃えてくれんか・・・？

## その他

気が向いたので記事たくさん書いてるけどまたすぐ飽きる気がする。習慣化するのが一番なんだろうけど・・・
