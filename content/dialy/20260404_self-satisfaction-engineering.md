+++
title = "自己満エンジニアリング"
date = "2026-04-04"
description = "横道逸れまくり自己満エンジニアリング"
[taxonomies]
tags = ["dialy"]
+++

横道エンジニアリングの流れ

- AI開発してみる (claudeで遊ぶ)
- トークン足りないので下流開発だけならローカルAIでできないか画策
	- aider + ollama(qwen-3.5:9b)試す
- mac(m3 16g)じゃ弱いので自宅のあまり使ってないゲーミングPCのAI化を画策
- 連携サーバとしてとして昔の自宅鯖Proxmox再利用を画策
- LXCコンテナ (almalinux-10) 上のdockerで連携基盤構築を画策
- 出先からも接続したくてtailscaleコンテナを立てる
- tailscale経由で他の機能も外部から接続したくて `--accept-node` を設定
	- dockerネットワーク、lxcホスト、PVEホストを公開
- チャット基盤でAI連携したくてmattermost-teamコンテナ立てる
	- 併せてpostgres18コンテナも立てる
- tailscale内で名前解決したくてCoreDNSコンテナ立てる
- tailscaleで広告ブロックプロキシ立てたくなりprivoxyコンテナ立てる
- 各端末(Macbook、スマホ等含む)からの通信をtailscale経由にしたくなり exit-node を設定
- httpsも対応させたくてprivaxy試すも断念 (透過型にできなかったため)
- 広告ブロック狙いでadguardhomeコンテナを立てる
- DNSをCoreDNSからadguardhomeに移管
- 開発環境用に別LXCコンテナを立てることを画策🚧途中
- tailscaleへの自宅LAN全公開に抵抗を感じ、PVEのSDN機能を使い始める
	- Zone, VNET, Subnet を設定して SNAT でインターネット接続
- 環境再現性が気になり始め PVE の自動構築を調査、pulumi 構築を試み始める🚧途中

- 別でやってた RDS Postres Express Configuration の PoC アプリのデプロイに詰まる
- aws cli でのセッショントークン取得が煩雑に感じ始め aws-vault 導入
- aws-vault のコマンドが煩雑で zsh-aws-vault 導入
- 久々に触った dotfiles の gitignore ホワイトリスト管理が煩雑に感じ始めて chezmoi へ移行
- ターミナルアプリどれが良いか悩み alacritty から ghostty へ移行
- ghostty での claude の作業完了通知を設定
- 久々に neovim 触ったら色々エラー出てるので修復
- AI待ち時間中のメモを zola で書こうとしつつ、obsidian との連携を画策
	- github actions で構築
- zola のディレクトリ階層化を設定
- zola の mermaid 図についてショートコードが煩雑になりコードブロックを使用可能に設定

TODO
- tailscale リバースプロキシ設定
- mattermost と aider/ollama の連携プログラム作成
- mattermost と claude の連携プログラム作成
- 