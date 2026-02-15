+++
title = "zed の見た目を良い感じにしつつ透明化する"
date = "2026-02-15"
description = "エディタの見た目は大事"
[taxonomies]
tags = ["editor", "zed"]
+++

エディタの見た目はテンションに直結するので良い感じにしたい。メインのエディタとして使うかは分からないけど使ってみている zed の見た目を良い感じにするメモ。

[参考 GitHub スレッド](https://github.com/zed-industries/zed/issues/5040#issuecomment-2510486980)

個人的には `blurred` でぼかすよりも `transparent` で背景がうっすら透けて見える方が好き。使用テーマにもよると思うが私のテーマはこれで大体透明になった。(なお、テーマは `Catppuccin Mocha` を使用)

本当は [Deepdark Material Theme](https://marketplace.visualstudio.com/items?itemName=Nimda.deepdark-material) を zed に port したいが無かったのと、自力で port しようとしたがうまくいかなかったので。（シンタックスハイライトはこれが一番好き）

```json: settings.json
{
  ..., // その他の設定
  "ui_font_weight": 100.0,
  "ui_font_family": "HackGen35 Console NF",
  "vim_mode": true,
  "icon_theme": "Catppuccin Mocha",
  "theme": "Catppuccin Mocha",
  "telemetry": {
    "metrics": false,
    "diagnostics": false,
  },
  "buffer_font_family": "HackGen35 Console NF",
  "ui_font_size": 14.0,
  "buffer_font_size": 14.0,
  "experimental.theme_overrides": {
    "background.appearance": "transparent",
    "renamed.background": "#FFFFFF20",
    "search.match_background": "#FFFFFF20",
    "ghost_element.background": "#00000010",
    "ghost_element.hover": "#00000099",
    "background": "#000000CC",
    "editor.background": "#00000010",
    "editor.gutter.background": "#00000010",
    "title_bar.background": "#000000DF",
    "toolbar.background": "#00000010",
    "terminal.background": "#00000000",
    "status_bar.background": "#000000DF",
    "tab.active_background": "#000000",
    "tab.inactive_background": "#00000000",
    "tab_bar.background": "#00000010",
    "panel.background": "#00000010",
    "border": "#00000000",
    "border.variant": "#00000000",
    "scrollbar.track.border": "#00000000",
    "scrollbar.track.background": "#00000000",
    "scrollbar.thumb.background": "#00000000",
    "scrollbar.thumb.hover.background": "#00000000",
    "scrollbar.thumb.active.background": "#00000000",
    "scrollbar.thumb.border": "#FFFFFF90",
    "editor.line_highlight": "#00000000",
    "editor.active_line.background": "#00000000",
    "editor.selection.background": "#00000000",
    "editor.selection.foreground": "#00000000",
    "editor.selection.border": "#00000000",
    "editor.selection.inactive.foreground": "#00000000",
    "editor.selection.inactive.border": "#00000000",
    "editor.selection.active.background": "#00000000",
    "editor.selection.active.foreground": "#00000000",
    "editor.selection.active.border": "#00000000",
    "editor.selection.inactive.background": "#00000000",
  },
}
```
