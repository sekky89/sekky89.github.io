+++
title = "neovim (alacritty) をあれこれ弄りつつ透明化する（見た目編）"
date = "2026-02-15"
description = "エディタの見た目は大事"
[taxonomies]
tags = ["editor", "neovim"]
+++

最近 VSCode が重くなることが多くストレス感じるため幾つか移行候補を考えつつ試してみている。エンジニアとしての育ちが eclipse で、 vim 系エディタは最近触り始めたが思ったより快適なので Neovim はかなり良いと思う。今のところ、 zed と neovim がかなり良い感じ。

ターミナルは [alacritty](https://alacritty.org/) を使用する。warp、wezterm も触ったがリッチすぎてターミナルアプリとしては TooMuch なので alacritty にした。

というわけでざっくり見た目設定。あちこち None を設定しているのは透明化設定。

```lua: init.lua
vim.cmd.colorscheme('catppuccin')

vim.api.nvim_set_hl(0, 'WinBar', { bg = 'none' })
vim.api.nvim_set_hl(0, 'WinBarNC', { bg = 'none' })
vim.api.nvim_set_hl(0, 'NeoTreeTabInactive', { bg = 'none' })
vim.api.nvim_set_hl(0, 'NeoTreeTabSeparatorInactive', { bg = 'none' })
vim.api.nvim_set_hl(0, 'StatusLine', { bg = 'none' })
vim.api.nvim_set_hl(0, 'StatusLineNC', { bg = 'none' })
vim.api.nvim_set_hl(0, 'StatusLineFill', { bg = 'none' })
vim.api.nvim_set_hl(0, 'BufferLineFill', { bg = 'none' })
```

プラグインマネージャは lazy を使用。`lua/plugins/colorscheme.lua` には以下を仕込んでいる。`catppuccin` の設定上も各所に透明化を仕込んでいる。

```lua: lua/plugins/colorscheme.lua
  {
    'https://github.com/catppuccin/nvim.git',
    name = 'catppuccin',
    priority = 1000,
    opts = {
      flavour = 'mocha',
      transparent_background = true,
      float = { transparent = true, solid = true },
      auto_integrations = true,
    },
  },
```

そのまま使用するとターミナルアプリの透明化設定がそのまま適用される。  
`alacritty.toml` はこんな感じ。blur はオフにしつつ opacity で良い感じに裏側の文字が邪魔にならない程度にうっすら透ける程度に設定。  
ターミナルテーマまで catppuccin かよって感じだけど、個人的には背景色を黒にできれば割と何でもいい。

```
[general]
import = ["catppuccin.toml"]
# import = ["gruvbox_material_hard_dark"]
# import = ["gruvbox_dark"]

[window]
dimensions = { columns = 180, lines = 58 }
position = { x = 0, y = 1000 }
padding = { x = 8, y = 4 }
decorations = "Buttonless"
opacity = 0.8
blur = false

[scrolling]
history = 100000

[font]
normal = { family = 'HackGen35 Console NF', style = "Regular" }
bold = { family = "HackGen35 Console NF", style = "Bold" }
italic = { family = "HackGen35 Console NF", style = "Italic" }
bold_italic = { family = "HackGen35 Console NF", style = "Bold Italic" }
size = 12
offset = { x = 0, y = 2 }
glyph_offset = { x = 0, y = 1 }

[colors]

[colors.primary]
background = "#000000"

[selection]
save_to_clipboard = true

[mouse]
hide_when_typing = true

[debug]
```
