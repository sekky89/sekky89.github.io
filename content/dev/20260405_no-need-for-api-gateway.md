+++
title = "API Gateway 不要説"
date = "2026-04-05"
description = "やっぱ不要とは言えない"
[taxonomies]
tags = ["aws", "api gateway", "cloudfront", "lambda"]
+++


CloudFront + Lambda Function URL (OAC) w/Lambda-lith App を試してみた。
併せて、Aurora PostgreSQL Express Configuration も試したけどそれは別記事で。

## 結論
できなくはないし、面白いが運用上はリスクが多いため限定的な用途を除いては採用できない。
ホビー用途なら全部これでも良いけど、セキュリティ気にするならやめた方がいい。


## 得るもの
API Gateway の制約
- Steram とか Server Sent Event とかが使いやすい
- タイムアウトが Lambda 依存になる
API Gateway 分のコスト
- API Gateway ないから・・・

## 失われるもの
レート制限
- Lambda の実装に含めればできるけど、そもそも Lambda 実行させたくない
Authorizer
- 同上、アプリ実行前に検証したい
- Cognito Authorizer との組み合わせもアプリ側実装になる (JWT検証とか)
設定の楽さ
- やろうとすると最低限 CloudFront Function は必須
- その辺の作り込みも含めるとめんどい
セキュリティ
- 認証タイプ NONE : Lambda 実行されてしまう
- 認証タイプ AWS_IAM : CloudFront ではリクエストボディを含めた署名ができないためそのままではエラー。**クライアントで `x-amz-content-sha256` を計算して付加する必要がある。** (ペイロードがでかい場合クライアント負荷が大きくなる。クライアントの時刻同期ずれとかで署名エラー起きそう)
他色々


## 他の方式との比較
### API Gateway (REST API) + Lambda
- 古き良きアーキテクチャ。前述の失われるものがない。
- 無駄に Lambda が実行されることもない。

### CloudFront + API Gateway (HTTP API) + Lambda
- 今の王道はこっちかも。
- CloudFront を経由させることで WAF を適用可能
- SPA もホスティングしている場合、一つの CloudFront + WAF でフロントバック両方守れてお得
- Cognito との連携も JWT Authorizer で容易
- REST API よりは安い
- REST API と同様に、ある程度は API Gateway までで叩き落とせるのでセキュリティも良き。

## 使い所
前述の通り、Server-Sent Event や Response Streaming を使用したい場合など、API Gateway の制約に掛かる場合に限定的な用途で使用するのはあり。
通常は API Gateway (HTTP API) を使用しつつ、一部のエンドポイントだけ Lambda Function URL に流すといった使い方。
29秒以内に終わらないリクエストに使用することもできるが、その用途だと際限なさそうなので素直にポーリングした方が良いような気がする。

セキュリティは十分注意が必要。