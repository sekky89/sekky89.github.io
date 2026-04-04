+++
title = "Obsidian から Zola (GitHub Pages) への自動同期"
date = "2026-04-04"
description = "Obsidian の public ディレクトリを GitHub Actions で Zola サイトに自動同期する仕組み"
[taxonomies]
tags = ["obsidian", "zola", "github-actions"]
+++

Obsidian で書いた記事を GitHub Pages (Zola) に自動公開できないかな？と gemini と claude に相談して作ってみた。

結果として、GitHub Actions のピタゴラスイッチになった。

## やりたいこと

- Obsidian の `public/` ディレクトリに記事を書く（全部公開せず、特定ディレクトリだけを対象にしたい）
- push するだけで GitHub Pages に反映される
- ローカルにスクリプトを置きたくない

## 構成

リポジトリが2つある。

| リポジトリ | 役割 |
|-----------|------|
| `obsidian` (private) | Obsidian Vault。`public/` に公開用記事を置く |
| `sekky89.github.io` (private) | Zola サイト。`content/` が公開対象 |

## フロー

{% mermaid() %}
flowchart TD
    A[Obsidian で public/ に記事を書く] --> B[obsidian リポに git push]
    B --> C{GitHub Actions<br>sync-to-pages.yml}
    C -->|"paths: public/** で発火"| D[obsidian リポを checkout]
    D --> E[sekky89.github.io を checkout<br>PAT で認証]
    E --> F["rsync --delete<br>public/ → content/"]
    F --> G{差分あり?}
    G -->|Yes| H[commit & push to<br>sekky89.github.io]
    G -->|No| I[終了]
    H --> J{GitHub Actions<br>gh-pages.yml}
    J -->|on push で発火| K[zola build]
    K --> L[gh-pages ブランチに deploy]
    L --> M[GitHub Pages で公開]

    style A fill:#7c3aed,color:#fff
    style M fill:#059669,color:#fff
    style I fill:#6b7280,color:#fff
{% end %}

## obsidian リポ側の GitHub Actions

`public/` 配下に変更があった場合のみ発火する。sekky89.github.io をチェックアウトして `rsync --delete` で同期し、commit & push する。

```yaml: .github/workflows/sync-to-pages.yml
name: Sync public to GitHub Pages

on:
  push:
    branches:
      - main
    paths:
      - 'public/**'

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout obsidian repo
        uses: actions/checkout@v4
        with:
          path: obsidian

      - name: Checkout pages repo
        uses: actions/checkout@v4
        with:
          repository: sekky89/sekky89.github.io
          token: ${{ secrets.PAGES_REPO_TOKEN }}
          path: pages

      - name: Sync public to content
        run: |
          rsync -av --delete obsidian/public/ pages/content/

      - name: Commit and push
        working-directory: pages
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add -A
          if git diff --cached --quiet; then
            echo "No changes to commit"
          else
            git commit -m "Sync content from obsidian/public"
            git push
          fi
```

`rsync --delete` なので、`public/` から消した記事は `content/` からも消える。

## PAT の設定

`GITHUB_TOKEN` は自リポジトリにしかアクセスできないため、別リポジトリへの push には Personal Access Token (PAT) が必要。

1. GitHub → Settings → Developer settings → Fine-grained Personal Access Tokens で作成
2. `sekky89.github.io` リポの **Contents: Read and write** 権限を付与
3. obsidian リポの Settings → Secrets に `PAGES_REPO_TOKEN` として登録

## 所感

Obsidian で書いて push するだけで公開される。ローカルに同期スクリプトを置く方式も検討したが、GitHub Actions に閉じた方がマシン依存がなくて良い。
