+++
title = "自己満エンジニアリング"
date = "2026-04-04"
description = "横道逸れまくり自己満エンジニアリング"
[taxonomies]
tags = ["dialy"]
+++

横道エンジニアリングの流れ

### AI開発してみる (2月中旬〜3月末)

Claude と対話しながらアプリを設計・実装する実験。技術スタックの選定からアーキテクチャ設計、コード生成、レビューまで一通り Claude に壁打ちしながら進めた。

- 技術スタック: Go 1.26 + fiber v3 / React + Vite + shadcn/ui + zustand / CDK + Lambda Web Adapter
- 自動生成重視: sqldef (宣言的マイグレーション)、sqlc + sqlc-gen-crud、oapi-codegen (OpenAPI → Go)、wire (DI)、hey-api (OpenAPI → TS クライアント)、valibot (OpenAPI → バリデーション)
- Go 1.24 の `go tool` ディレクティブでツール管理統一、Docker Compose 上で全自動生成実行
- lint/format: oxlint + oxfmt (eslint/prettier 不使用)、go-arch-lint で依存方向を強制
- Clean Architecture + interface ベース設計、slog による構造化ロギング
- DB比較検討: DSQL vs Neon vs Supabase vs CockroachDB vs Aurora Serverless
- システム間通信: Outbox パターン (イベント駆動、ポーリング不使用)
- 詳細は [バックエンド編](@/dev/20260314_backend-with-claude.md)、[フロントエンド編](@/dev/20260314_frontend-with-claude.md)、[所感](@/dev/20260314_with-claude.md)、[sqldef](@/dev/20260314_sqldef.md)

### ローカルAI環境の構築 (3月上旬〜)

トークン消費を抑えるため、下流の実装作業をローカル AI に委譲する構想。

- aider + ollama (qwen-3.5:9b) を M3 MacBook Air (16G) で試すも性能不足
- 自宅の遊休ゲーミングPC (Ryzen 9800X3D + 64G + RTX3080) を ollama サーバ化する方向へ
- Claude が設計・指示、ollama が実装、人間が介入できるチャット基盤として Mattermost (Team Edition) をセルフホスト
- 詳細は [ゲーミングPCをOllamaサーバにする構想](@/infra/20260323_ollama-server.md)

### 自宅インフラ (3月中旬〜)

ollama サーバの基盤として始めたが、芋づる式に拡大。

- 常設サーバの Proxmox 上に LXC コンテナ (AlmaLinux 10) を立て、Docker で各種サービスを稼働
- tailscale コンテナで外部接続、`--accept-node` で docker ネットワーク・LXC ホスト・PVE ホストを公開
- mattermost-team + postgres18 コンテナ (AI 連携チャット基盤)
- CoreDNS → AdGuard Home に移管 (DNS + 広告ブロック)
- privoxy → privaxy (HTTPS 透過プロキシ断念) → AdGuard Home に統合
- exit-node 設定で全端末の通信を tailscale 経由に
- PVE の SDN 機能 (Zone, VNET, Subnet + SNAT) で自宅 LAN 全公開を回避
- 環境再現性のため pulumi による PVE 自動構築を試行中🚧

### 開発環境整備 (4月4日)

PoC アプリのデプロイ作業をきっかけに、周辺ツールの粗が次々と気になり始める。

- aws-vault + zsh-aws-vault 導入 (MFA セッショントークン管理の自動化)
- dotfiles を gitignore ホワイトリスト管理 → chezmoi に移行
- alacritty → ghostty に移行
- ghostty + hooks + osascript で Claude の作業完了通知を設定
	- macOS のフルディスクアクセス権限周りでハマる
- karabiner-elements の代替検討 (cmd-eikana, eikana 等) → 可搬性重視で karabiner 継続、設定を chezmoi 管理に
- neovim 修復 (プラグイン更新、deprecated 整理、nvim-colorizer 代替切替、tmux 連携修正)
- Obsidian → Zola 自動同期パイプライン構築 ([詳細](@/tools/20260404_obsidian-zola-sync.md))
- zola のディレクトリ階層化、mermaid コードブロック対応

## ふりかえり

本来やりたかったことは AI を活用した開発体験の探求。しかし触るたびに周辺ツールの粗が気になり、芋づる式に改善が連鎖していく典型的なヤクの毛刈りパターンに陥っている。

振り返ると、大きく3つの流れがある:

1. **AI開発の実験** → 技術スタック選定・アーキテクチャ設計・実装を Claude と壁打ち
2. **ローカルAI基盤** → ollama サーバ化 → 自宅インフラの芋づる式拡大
3. **開発環境整備** → aws-vault → chezmoi → ghostty → neovim → obsidian-zola と連鎖

横道に逸れること自体が悪いわけではなく、逸れた先で得た知見がいずれ本線に還元される——と信じたい。

## TODO
- tailscale リバースプロキシ設定
- mattermost と aider/ollama の連携プログラム作成
- mattermost と claude の連携プログラム作成
- PVE pulumi 自動構築の続き
- 開発環境用 LXC コンテナの構築