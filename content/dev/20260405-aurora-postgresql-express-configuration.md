+++
title = "Aurora PostgreSQL Express Configuration 試してみた"
date = "2026-04-05T"
description = "起動は速い"
[taxonomies]
tags = ["aurora", "postgress", "serverless"]
+++

数十秒で立ち上がると噂の Express Configuration で遊んでみた。

Aurora というと VPC 必須、作成が遅い、削除も遅いで CDK がイライラタイムになることで有名。

VPC 必須に吊られて Lambda も ENI アタッチで起動が遅くなるおまけつきで更にストレスが溜まることで有名。

## 結論

細かい制約は公式や先人の記事の方が見やすいので気になった点だけ

- DB作成は本当に早い
- DB削除は普通に遅い（先人の記事にも記載の通り）
- 設定次第で起動は遅い
	- Serverless v2 で Min ACU を 0 にすると起動に20秒弱くらい。これは Serverless v2 の仕様なのでどうしようもない
- セキュリティ要件でVPC必須にならなければ普通に運用に乗せても良さそう

## 気になる点
- IAM認証
	- 多少オーバーヘッドありそう。
- RDS Proxy は使えない
	- Lambda の Concurrency が増えると接続不良起きそう
	- セオリー通りにバラの Lambda を大量に作ってるワークロードと相性悪そう
	- Lambda-lith と相性良さそう

## 使ってみて

Lambda-lith で作った API と組み合わせて使ってみた。前述の通り、DB 停止からだと 20 秒くらい。これは仕方ない。

2回目以降のアクセスは単純な GET で 100ms 切ったので UX ヨシ。

## 参考
以下記事参考にさせていただきました。
https://zenn.dev/unilorn/articles/72b6918c2ccb2e
https://zenn.dev/ncdc/articles/7e9c8dfa47fd5e
https://dev.classmethod.jp/articles/aurora-postgresql-express-configuration-serverless-database-creation-in-seconds/