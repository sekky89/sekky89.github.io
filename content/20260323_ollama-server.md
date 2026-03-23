+++
title = "ゲーミングPCをOllamaサーバにする構想"
date = "2026-03-23"
description = "遊休ゲーミングPCと常設Proxmoxサーバで自宅AI開発環境を構築する"
[taxonomies]
tags = ["llm", "ollama", "selfhost", "mattermost", "proxmox", "tailscale"]
+++

最近ゲームしなくなった自宅のゲーミングPC（Ryzen 9800X3D / 64GB RAM / RTX 3080）がただの文鎮と化しているので、Ollamaサーバにして開発用に使えないかという構想メモ。

## モチベーション

- Claude Code で設計・検討した内容の実装作業を、ローカル LLM + aider に任せたい
- Claude は思考・設計が得意、実装の手を動かす部分はローカルで十分なケースがある
- 設計は Claude、実装は aider（ローカルLLM）という分業でAPI費用を抑えられる
- RTX 3080 (VRAM 10GB) が遊んでるのはもったいない

## 全体構成

```
                        Tailscale ネットワーク
                     (CoreDNS で内部名前解決)
                               |
        +----------------------+----------------------+
        |                      |                      |
  [MacBook]           [ゲーミングPC]          [常設サーバ]
   Claude Code         Windows                Ryzen 5700G / 64GB
   aider               Ollama + qwen3.5       Proxmox
                        (GPU: RTX 3080)          |
                                            [AlmaLinux VM]
                                              Docker
                                                ├ Mattermost (Team Edition)
                                                ├ CoreDNS
                                                ├ aider Bot
                                                └ (その他コンテナ)
```

### 役割分担

| マシン | 役割 | 特徴 |
|--------|------|------|
| MacBook | 設計・指示 | Claude Code で設計、aider で直接実装も可 |
| ゲーミングPC | GPU推論 | Ollama 専用。必要な時だけ WoL で起動 |
| 常設サーバ | インフラ基盤 | 常時起動。Mattermost, DNS, Bot 等を集約 |

ゲーミングPC に全部載せない理由:
- ゲーミングPC は電気食うので常時起動したくない（WoL で必要な時だけ）
- Mattermost や Bot は常時起動が前提のサービス
- 常設サーバ（Ryzen 5700G）は省電力で24/365回せる

## 人間介入フロー

メッセージングツールを挟む最大の利点は「人間がゲートになれる」こと。

```
Claude Code → Mattermost #design に設計投稿
                      ↓
                スマホに通知
                      ↓
            人間がレビュー・承認
           （リアクション or コメント）
                      ↓
          aider Bot が承認を検知
       Ollama (ゲーミングPC) に推論リクエスト
                      ↓
           aider + qwen3.5 で実装
                      ↓
         Mattermost #review に diff 投稿
                      ↓
                スマホに通知
                      ↓
             人間が確認して判断
```

出先からスマホで設計をレビューして、OKならリアクション一つで実装が走る。

## なぜ Mattermost か

セルフホスト可能な候補:

| ツール | Bot API | Docker | スマホアプリ |
|--------|---------|--------|-------------|
| **Mattermost** | Webhook + Bot SDK | 公式イメージ | iOS / Android |
| Rocket.Chat | REST API + Bot | 公式イメージ | iOS / Android |
| Zulip | Bot framework | 公式イメージ | iOS / Android |
| Matrix (Element) | Bot SDK | Synapse | iOS / Android |

正直どれでも実現できるが、Mattermost は Webhook の作りやすさと Bot SDK のドキュメントの充実度で選定。

### Edition 選定

| | Team Edition (MIT OSS) | Enterprise Edition Entry モード |
|---|---|---|
| 費用 | 無料 | 無料（ライセンスキー不要） |
| メッセージ履歴 | **制限なし** | サーバー全体で10,000件 |
| プッシュ通知 | TPNS（SLAなし） | 月10,000件 |
| Bot / Webhook | 使える | 使える |
| スマホアプリ | 使える | 使える |
| SSO | **なし（2025年以降削除）** | あり |

**Team Edition を採用。** Enterprise Entry はメッセージ10,000件制限があり、Bot の diff 出力を流し続けるとすぐ到達する。SSO がないが個人1人なので問題なし。

## ネットワーク構成

### Tailscale

全マシンを Tailscale で接続。外部にポートを開けない。

- MacBook ↔ ゲーミングPC: aider → Ollama (11434/tcp)
- MacBook ↔ 常設サーバ: Claude Code → Mattermost (8065/tcp)
- 常設サーバ ↔ ゲーミングPC: aider Bot → Ollama (11434/tcp)

### CoreDNS

Tailscale の IP を直打ちするのは辛いので、CoreDNS で内部名前解決する。

```
ollama.home.arpa  → ゲーミングPC の Tailscale IP
mattermost.home.arpa → 常設サーバの Tailscale IP
```

CoreDNS も常設サーバの Docker コンテナとして動かす。Tailscale の MagicDNS でも良いが、自前で管理したい場合は CoreDNS が柔軟。

## セットアップ（予定）

### ゲーミングPC（Windows）

Ollama 専用。GPU推論だけやる。

1. NVIDIA ドライバを最新に
2. Ollama for Windows をインストール
3. Tailscale をインストール
4. Ollama のリッスンアドレスを変更

```powershell
[System.Environment]::SetEnvironmentVariable("OLLAMA_HOST", "0.0.0.0", "Machine")
```

5. qwen3.5 を pull

```bash
ollama pull qwen3.5
```

### 常設サーバ（Proxmox + AlmaLinux VM）

Mattermost, CoreDNS, Bot 等の常駐サービスを集約。

1. Proxmox 上に AlmaLinux VM を作成
2. Docker + Docker Compose をインストール
3. Tailscale をインストール
4. 各コンテナを起動

```yaml
# docker-compose.yml
services:
  mattermost:
    image: mattermost/mattermost-team-edition
    ports:
      - "8065:8065"
    volumes:
      - ./mattermost/data:/mattermost/data
    restart: unless-stopped

  coredns:
    image: coredns/coredns
    ports:
      - "53:53/udp"
      - "53:53/tcp"
    volumes:
      - ./coredns:/etc/coredns
    command: -conf /etc/coredns/Corefile
    restart: unless-stopped

  # aider Bot（要自作）
  # Mattermost の Webhook を監視して
  # Ollama (ゲーミングPC) に推論リクエストを投げる
```

### クライアント側（Mac）

```bash
# 直接 aider を使う場合
export OLLAMA_API_BASE=http://ollama.home.arpa:11434
aider --model ollama/qwen3.5
```

Claude Code からは MCP サーバ経由で Mattermost に投稿する構成も考えられる。

## qwen3.5 について

RTX 3080 の VRAM 10GB で動くサイズが出ているかは要確認。
aider の実装作業者として使う場合、設計ドキュメントへの指示追従性が重要。実際に試してみるしかない。

## 気になっていること

- **ゲーミングPC の電気代**。WoL で必要な時だけ起動したい
  - 常設サーバの aider Bot から WoL パケットを飛ばせば良い
  - 「承認されたら WoL → Ollama 起動確認 → aider 実行 → 完了後シャットダウン」の自動化
- **qwen3.5 の VRAM 要件**。10GB に収まるサイズ・量子化があるか
- **aider との相性**。OpenAI互換APIで繋がるはずだが、モデルごとの癖がある
- **Proxmox の VM リソース配分**。AlmaLinux VM に RAM いくら割り当てるか。Mattermost + CoreDNS + Bot なら 8GB もあれば足りそう

## 段階的に進める

1. まず Ollama + qwen3.5 をゲーミングPC（Windows）に入れて動作確認
2. Tailscale 経由で Mac の aider から繋いでみる
3. ファイルベースの受け渡し（設計.md → aider --read）で分業を試す
4. 常設サーバに AlmaLinux VM を作成、Docker 環境を構築
5. Mattermost Team Edition + CoreDNS をコンテナで立てる
6. aider Bot を開発して Mattermost 連携
7. スマホからレビュー・承認のフローを構築
8. WoL 自動化（承認 → 起動 → 実行 → シャットダウン）

いきなり全部組むと辛いので、小さく始めて段階的に。


---

というアイデアベースの構想を Claude にまとめてもらった。
作ってどの程度仕事してくれるか見てみようかな。
