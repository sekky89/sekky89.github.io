+++
title = "Claude 君と練ったバックエンドアーキテクチャ"
date = "2026-03-14"
description = "全部は生成させない"
[taxonomies]
tags = ["claude", "backend", "go"]
+++

ついに遊び半分で Claude Code (Pro) に課金してしまった。課金してしまったからには遊ばねば。

## ポリシー

個人的なポリシーとして今回は以下のようなルールを設けた。

- なるべく宣言的
- なるべく自動生成ツールを活用
- SOLID 原則
- Clean Architecture 風
- DDD 風

この縛りを設けて、AI開発による不確定部分を減らすこととトークンの節約を試みた。トークンの節約についてはできたのかできてないのかわかってない。

## 技術スタック

結局以下のような構成に。これも Claude に書かせた表。

| カテゴリ             | 技術                     | バージョン |
| -------------------- | ------------------------ | ---------- |
| 言語                 | Go                       | 1.24+      |
| Webフレームワーク    | Gofiber v3               | 最新安定版 |
| データベースドライバ | jackc/pgx v5             | v5.x       |
| DI                   | google/wire              | 最新安定版 |
| ID生成               | lucsky/cuid              | 最新安定版 |
| コード生成           | oapi-codegen, sqlc       | 最新安定版 |
| Linter               | golangci-lint            | 最新安定版 |
| アーキテクチャLinter | go-arch-lint             | 最新安定版 |
| テスト               | 標準testing + mockgen    | —          |
| デプロイ             | AWS Lambda (linux/arm64) | —          |

細かなところで、sqlc-gen-crud や sqldef が抜けてたりするのはご愛嬌。

{% mermaid() %}
block-beta
  columns 4

  RT["AWS Lambda + Web Adapter"]:4

  FIBER["Gofiber v3"]:2 OAP["oapi-codegen"]:1 RED["redocly"]:1

  WIRE["wire"]:2 MOCK["mockgen"]:2

  CUID["lucsky/cuid"]:4

  PGX["pgx v5"]:1 SQLC["sqlc"]:1 CRUD["sqlc-gen-crud"]:1 SDEF["sqldef"]:1

  DB[("PostgreSQL（Aurora DSQL）")]:4

  LINT["golangci-lint"]:2 ARCH["go-arch-lint"]:1 TEST["標準 testing"]:1

  style RT fill:#4a90d9,color:#fff
  style FIBER fill:#7bc96f,color:#fff
  style OAP fill:#7bc96f,color:#fff
  style RED fill:#7bc96f,color:#fff
  style WIRE fill:#f5a623,color:#fff
  style MOCK fill:#f5a623,color:#fff
  style CUID fill:#d0021b,color:#fff
  style PGX fill:#9b59b6,color:#fff
  style SQLC fill:#9b59b6,color:#fff
  style CRUD fill:#9b59b6,color:#fff
  style SDEF fill:#9b59b6,color:#fff
  style DB fill:#34495e,color:#fff
  style LINT fill:#95a5a6,color:#fff
  style ARCH fill:#95a5a6,color:#fff
  style TEST fill:#95a5a6,color:#fff
{% end %}

Claude 君に作らせたいい加減な図。

## 選定経緯

### Go

今時の Web アプリケーションならもう Go かなと。パフォーマンス極めるなら rust かもしれないけど今回は Go にしてみた。シングルバイナリゆえのコールドスタートの速さ、デプロイの容易さが良い。

### fiber, oapi-codegen, redocly, AWS Lambda Web Adapter

gin が主流かもしれないが express インスパイアで生まれた fiber を採用。早いらしい。実測はしてない。ミドルウェアが充実しており、Go 標準 Http ハンドラの Adapter も提供されている。

OpenAPI 定義からの自動生成には oapi-codegen を採用。各サーバ用の実装を生成できるが、fiber は対応していない。代わりに Strict Server Interface の生成ができるようになっている。これは標準 Http ハンドラとして生成されるので Adapter を噛ませれば fiber にも組み込める。

OpenAPI 定義そのもののドキュメント生成兼 linter として redocly を採用。Claude 君には OpenAPI 定義変えたら redocly チェックしてエラー解消してねと指示しておいた。

Lambda で動かすことを前提に選定したが、個別ハンドラではなく API サーバ丸ごと作って上げる方式を採用。ローカルでの起動が容易など可搬性が良く、リソース増えすぎないので CDK にも優しい。そのままでは Lambda 上で動作しないが、そこは AWS Lambda Web Adapter をレイヤーないしはコンテナに組み入れることでアプリに手を加えることなく対応できる。（数msのレイテンシは犠牲になるが・・・）

### sqlc, pgx, sqlc-gen-crud, sqldef

DB 周りは RDB を前提に選定。sqldef で DDL を管理し、そこから sqlc と migration を自動生成。sqlc-gen-crud で標準 CRUD を自動生成。平仄が気になる部分は自動生成で捩じ伏せる。クエリが複雑化したり動的クエリが必要となったら goqu と組み合わせるのも良さそう。

### go-arch-lint

Clean Architecture になっているよねチェッカー。依存方向を定義でき、違反を検出してくれる。SOLID にしろって指示していてもできたコードは依存パッケージの違反がそこそこあったので導入。機械的な仕組みでエラー検出してくれるツールは信頼できる。

### wire

DI にはコンパイル時に静的 DI できる wire を選択。最悪なくとも良いがあると楽という程度の DI なのが良い。

なぜメンテが停止している wire を？と思われるかもしれないが、メンテが停止しているというより現時点では「完成」だと思う。go のエコシステムが大きく変わらない限り今の wire でずっと使える気がする。

他にも Dynamic な DI に対応したライブラリには fx や do などあるが、実装が DI ライブラリに依存してしまう。これが気に入らなかった。その点 wire はプロバイダの定義は必要なものの、コンパイル時に静的 DI するのでランタイムエラーに繋がりにくく、実装も DI ライブラリに依存しない。

プレーンな go のインタフェースで各層の実装ができるので mockgen 等でのテストモック自動生成との親和性も良い。

### その他

ロガーは fiber 内臓ミドルウェアで出力。パフォーマンス上げたければ zap にしても良い。

Clean Architecture だと presentor や usecase interactor も定義するものだが、今回はシンプルなので省略。大規模になってきたり、再利用性を高めたい場合は追加すれば良いと考える。

usecase の interface 作っても良かったが、go の思想的に interface は利用する側が定義するものとされているため今回はそちらを優先。  
最初から再利用性を考慮して interface を定義して、mockgen でモック作って並行でテスト作っても良いと考えている。

## 開発体験

機能・非機能要件を与え、domain model と repository interface 定義、usecase の定義、OpenAPI の定義、DB テーブルの定義を進めた上でそれぞれレビュー。
その上で、各自動生成ツールで吐いたコードを与えた要件に合わせて繋ぎ合わせる実装をさせるイメージ。実装も当然レビューはする。

{% mermaid() %}
flowchart LR
  subgraph 手書き["🤖 AIが書く / 人がレビュー"]
    OA[OpenAPI 定義<br/>openapi.yaml]
    DDL[DDL<br/>schema.sql]
    QR[クエリ定義<br/>queries/*.sql]
    DM[ドメイン層<br/>entity / repository IF<br/>/ service]
    UC[ユースケース層<br/>usecase]
    WR[DI 定義<br/>wire.go]
  end

  subgraph 自動生成["⚙️ ツールが生成するもの"]
    HC[ハンドラ IF<br/>oapi-codegen]
    MD[モデル<br/>sqlc]
    CR[CRUD<br/>sqlc-gen-crud]
    QC[クエリコード<br/>sqlc]
    WG[DI ワイヤリング<br/>wire]
    MK[テストモック<br/>mockgen]
  end

  OA --> HC
  DDL --> MD
  DDL --> CR
  QR --> QC
  DM --> MK
  WR --> WG
  DDL -.->|sqldef| DB[(実 DB)]

  subgraph 検証["🔍 機械的チェック"]
    RL[redocly]
    GL[golangci-lint]
    AL[go-arch-lint]
  end

  OA --> RL
  HC & DM & CR & QC & WG & MK --> AI{{"🤖 Claude<br/>繋ぎ合わせ実装"}}
  DM & UC --> AI
  AI --> GL
  AI --> AL
{% end %}

自動生成ツールは設定と定義が正しければ生成物を疑う余地はないので、全てをAIに生成させることによる生成コードの不確定さを排除できる。あとはレビュー観点のうち機械的にチェックできるものは極力 linter で排除することで人目による確認の負荷を減らすことができていると考える。
