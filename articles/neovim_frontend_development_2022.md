---
title: "Neovimでのフロントエンド開発環境 2022"
emoji: "⚡"
type: "tech"
topics: ["Vim", "Neovim", "LSP", "TypeScript", "フロントエンド"]
published: true
---

:::message
この記事は[Vim Advent Calendar 2022-1](https://qiita.com/advent-calendar/2022/vim)の22日目の予定だった記事です。
:::

投稿が予定より大幅に遅れてしまい申し訳ありません。
忙しかったのと記事のボリュームが想定より大きくなってしまい執筆に時間がかかってしまいました。

## はじめに

フロントエンドエンジニア(主にReact)をしているYano ([@yuki_ycino](https://twitter.com/yuki_ycino)) といいます。

去年のAdvent Calendarでは [Neovimでのフロントエンド開発環境 2021](https://zenn.dev/yano/articles/vim_frontend_development_2021) という記事を書きました。
この1年でかなり情勢が変わったので現在の状況について解説する記事となります。

去年に引き続きこの記事では主にLSPによる開発サポート及び現代の開発における必須プラグインの紹介をメインに進めています。
具体的には [coc.nvim](https://github.com/neoclide/coc.nvim) とNeovim built-inのLSP実装であるnvim-lspでの設定の2つについて解説と具体的な設定の紹介をしようと思います。

自分はcoc.nvimを普段使っており、nvim-lspを常用しているわけではないので多少バイアスがかかっていると思うのですがご了承ください。

また、去年よりも具体的な設定に割く量が増えているため、去年書いていた比較についてはいくつか割愛する部分もあります。
そのあたりが気になる方は去年の記事も目を通していただけるとありがたいです。

余談なのですが、自分は現在7000行超のVimの設定ファイルと150個前後のプラグインを導入して開発しています。参考になるか怪しいですがそのvimrcのURLも貼っておきます。
(去年からいつの間にか2000行以上増えていました。)
https://github.com/yuki-yano/dotfiles/blob/main/.vimrc

これは一旦整理してlua化したいなぁと思いながら放置したまま肥大化してしまっています・・・

## Neovimでの開発環境のイメージ

この記事に書いている設定だけではここまで全ての環境は整わないですが、自分はこのような見た目の環境で開発しています。
主にcoc.nvimを使って構築しています。

![my_neovim1](https://user-images.githubusercontent.com/5423775/210470128-9d42c0b1-276b-47b5-895c-68ea6d61934e.png)
![my_neovim2](https://user-images.githubusercontent.com/5423775/210470524-88fb0bc9-be38-4aaa-944f-7830dbf158fe.png)
![my_neovim3](https://user-images.githubusercontent.com/5423775/210470539-1201ec72-06e4-455c-b70e-ba17778f7d13.png)

## coc.nvimとnvim-lsp

前提としてNeovimでフロントエンド開発を行うには大きく分けて2つの方法があります。（他にも方法はあるのですが現在メジャーなものは以下の2つです）
coc.nvimというプラグインを使うか、Neovim built-inのnvim-lspを使って開発環境を整えるというものです。

去年まではnvim-lspはエコシステムが未成熟で勧めるには難しいものだったのですが、この1年でエコシステムが一気に成熟してユーザも増加しました。
周辺プラグインもかなり充実してきており、coc.nvimと比べても遜色ない開発環境を整備することができるようになってきています。

なので、この記事ではcoc.nvimとnvim-lspについてそれぞれの環境の構築例を載せようと思います。
正直最近のヘビーユーザーの流行はnvim-lspに傾いてきているのですが、cocも他にない利点が多くあります。

### coc.nvimとnvim-lspの違い

coc.nvimとnvim-lspの大きな違いとしてはプラグインの思想とコミュニティの気質が上げられます。

#### [coc.nvim](https://github.com/neoclide/coc.nvim)

coc.nvimはこれだけ入れれば簡単にVSCodeに近い開発体験を得られることを目標にしているAll in One系のプラグインです。
各言語のサポートや追加機能も `:CocInstall coc-{extension_name}` というコマンドを実行することで簡単に導入することができます。
Language Server・補完・VSCodeからPortingしたExtension・各種IDEのような表示の設定まで一通で全て行ってくれます。
各設定のコンフリクトなどで悩むことも比較的少ないようおもてなしもしてくれています。
SimpleとEasyがよく比較されますが、coc.nvimは間違いなくEasy側のプラグインです。

また、coc.nvim自体がTypeScriptで実装されておりVSCodeのextensionsをforkしてVim用にしているものも多いです。
そのためVSCodeの資産を使い回す事ができるのは大きなメリットです。
LSとの連携をTypeScriptで書くことができるため、エコシステムも比較的充実しています。

気になる点としてNodeに依存しているというものがあるのですが、フロントエンド開発においてはNodeはほぼ確実に導入されているので基本的に問題ないと思っています。

nvim-lspに対する利点としては機能がVSCode extensionのportingで実装されていることが多いので、LSの領域から外れた機能まで使えるということがあります。
例えばファイルシステムをwatchしてのimportの更新やTypeScriptのライブラリ内のコード実装定義へのジャンプはnvim-lspで現状使うことはできません。

#### nvim-lsp

一方、nvim-lspは小さなAPIをユーザが組み合わせて設定するという思想のbuilt-in機能及びそれを使用したプラグインです。
LSPを使うためのLanguage Serverの設定などもサポートするプラグインで簡略化はされていますがユーザが自分で行う必要があります。
補完を使うには別途プラグインの導入も必要です。
また、built-inのUIは非常に簡素なものが多いため、UIを拡張したり追加機能を提供するプラグインも数多く開発されています。

それらを整備することで好みの環境を構築することができるのがnvim-lspの強みだと思います。
ユーザが細かい動作まで制御可能という点がVimmerの志向にあっているというのもありそうです。

coc.nvimがLSP黎明期から開発されていて最近はかなり開発が安定しているのに対し、nvim-lspはコミュニティのプラグイン開発も含めて現在勢いがかなりあります。

しかし、詳細な設定が可能な代わりに一通り動作する環境を整えるまでのハードルはcoc.nvimに比べるとかなり高いです。
自分もこの記事を書く上で設定を準備する際にcocに比べるとそれなりに手間がかかりました。
Simpleな対価として設定にコストがかかるということは把握した上で手を出した方がよいと思います。（そもそもVim自体が本来そういったものですが）

また、自分が移行を試したときに難しいと感じたのはプラグインの選定で、非常に数が多く品質も玉石混交となっています。
コードの品質や保守されるか怪しいものなども多くありました。nvim-lspを使う際はそのあたりも気にした方がよいと思います。

Neovimのプラグインについてはyutkatさんの記事が非常に参考になります。
https://zenn.dev/yutakatay/articles/neovim-plugins-2022

nvim-lspはNeovim built-inなこともあってほぼLuaで設定を行います。

## coc.nvim, nvim-lsp共通の設定

### ファイラ

VSCodeやIntelliJなどのIDEではサイドバーにファイラが表示されていると思うのですが、Vimでもファイラがないと不便なことが多いので導入します。
Neovimでよく使われているLua製のプラグインでは[nvim-tree.lua](https://github.com/nvim-tree/nvim-tree.lua)が有名なのですが、今回は自分が慣れていないのもあり、vim-jpに作者がいてサポートを受けやすい[fern.vim](https://github.com/lambdalisue/fern.vim)を採用します。

### ColorSchemeとHighlight

VimのデフォルトのColorSchemeは各種プラグインで使うことなども含めると見た目があまりよくないため、様々なプラグインに対応しているColorSchemeを適用します。

#### [gruvbox-material](https://github.com/sainnhe/gruvbox-material)

非常に人気があり、さまざまなプラグインにも対応しているColorSchemeです。
後述のtreesitterや各種プラグインにも対応しているため、標準のColorSchemeを使うよりも綺麗にハイライトされます。

#### [nvim-treesitter](https://github.com/nvim-treesitter/nvim-treesitter)

構文解析をし、Syntax Highlightなどを提供してくれるNeovim用プラグインです。
Vimは標準ではSyntax Highlightを正規表現で実装しているので、それと比較して高精度かつ豊富なハイライトが行われます。

また、treesitterの拡張プラグインを導入することでSyntax Highlight以外にも様々な機能を使うことができるようになります。

## coc.nvimを用いた設定

### プラグインマネージャ

coc.nvimでは [vim-plug](https://github.com/junegunn/vim-plug) を採用します。
他にもいろいろなプラグインマネージャがあるのですがvim-plugが最も安定しているからです。
また、Neovimで現在最も使われている [packer.nvim](https://github.com/wbthomason/packer.nvim) はハマりどころが多く、Breaking Changeも比較的多いため最も安定したものとしてvim-plugを採用します。

### コード補完

coc.nvimが補完機能を実装しているので個別に導入する必要はありません。

### ファジーファインダー

coc.nvimと連携させるファジーファインダーとしてはfzf.vimと拙作のfzf-preview.vimを組み合わせて使います。
各プラグインの特性は以下の通りです。

#### [fzf.vim](https://github.com/junegunn/fzf.vim)

[fzf](https://github.com/junegunn/fzf)というCLIのファジーファインダーをVim上で動作させるプラグインです。
fzfがGo製なので高速に動作し、CLIとしてのfzfが広く使われていることもありVimプラグインとしても非常に有名です。

欠点としては、CLIツールをVimのterminal経由で動作させるという設計のために柔軟性がやや低く、いろいろな処理を挟みたくなった場合に複雑な設定をしないといけないということがあります。
また、それに伴いLSPとの連携などは基本的にできません。
その欠点を補うために拙作のfzf-preview.vimとの連携を行います。

#### [fzf-preview.vim](https://github.com/yuki-yano/fzf-preview.vim)

知名度・シェアでいうと他のプラグインに劣るのですが、coc.nvimと連携させて高度にfzfを使うなら最も手っ取り早いと自負しているので今回採用しました。
自作のTypeScript製のファジーファインダーでNodeに依存しており、内部的にfzfを使用しています。
All in One系のプラグインになっており、基本的に組み込みでリソースを提供していて拡張性はあまりありません。
その代わりに非常に少ない設定でcoc.nvimと連携した多くのリソースが使えるようになっています。

## coc.nvimの設定ファイル

### 準備

- https://github.com/junegunn/vim-plug/wiki/tutorial に書いているようにvim-plugを導入してください
- 以下の設定ファイルを `~/.config/nvim/init.vim` に配置してからNeovim(可能であればHEAD)を起動し、 `:PlugInstall` を実行してプラグインをインストールします(初回起動のインストール前にエラーが表示されると思いますが無視してください)

注意点として、nvim-lspの設定では `~/.config/nvim/init.lua` を配置するのですが、 `init.vim` と `init.lua` を両方配置すると正常に設定が読み込まれなくなってしまうので、片方を使う際はもう片方を削除してください。

### 設定ファイル

```vim
call plug#begin('~/.vim/plugged')

Plug 'vim-jp/vimdoc-ja'
Plug 'junegunn/fzf', {'dir': '~/.fzf_bin', 'do': './install --all'}
Plug 'neoclide/coc.nvim', {'branch': 'release'}
Plug 'lambdalisue/fern.vim'
Plug 'nvim-treesitter/nvim-treesitter'
Plug 'sainnhe/gruvbox-material'

call plug#end()

" set options
set termguicolors
set number
set updatetime=500
set expandtab
set tabstop=2

" map prefix
let g:mapleader = "\<Space>"
nnoremap <Leader>    <Nop>
xnoremap <Leader>    <Nop>
nnoremap <Plug>(lsp) <Nop>
xnoremap <Plug>(lsp) <Nop>
nmap     m           <Plug>(lsp)
xmap     m           <Plug>(lsp)
nnoremap <Plug>(ff)  <Nop>
xnoremap <Plug>(ff)  <Nop>
nmap     ;           <Plug>(ff)
xmap     ;           <Plug>(ff)

"" fern.vim
nnoremap <silent> <Leader>e <Cmd>Fern . -drawer<CR>
nnoremap <silent> <Leader>E <Cmd>Fern . -drawer -reveal=%<CR>

"" coc.nvim
let g:coc_global_extensions = ['coc-tsserver', 'coc-eslint', 'coc-prettier', 'coc-git', 'coc-fzf-preview', 'coc-lists']

inoremap <silent> <expr> <C-Space> coc#refresh()

nnoremap <silent> K             <Cmd>call <SID>show_documentation()<CR>
nmap     <silent> <Plug>(lsp)rn <Plug>(coc-rename)
nmap     <silent> <Plug>(lsp)a  <Plug>(coc-codeaction-cursor)

function! s:coc_typescript_settings() abort
  nnoremap <silent> <buffer> <Plug>(lsp)f :<C-u>CocCommand eslint.executeAutofix<CR>:CocCommand prettier.formatFile<CR>
endfunction

augroup coc_ts
  autocmd!
  autocmd FileType typescript,typescriptreact call <SID>coc_typescript_settings()
augroup END

function! s:show_documentation() abort
  if index(['vim','help'], &filetype) >= 0
    execute 'h ' . expand('<cword>')
  elseif coc#rpc#ready()
    call CocActionAsync('doHover')
  endif
endfunction

"" fzf-preview
let $BAT_THEME                     = 'gruvbox-dark'
let $FZF_PREVIEW_PREVIEW_BAT_THEME = 'gruvbox-dark'

nnoremap <silent> <Plug>(ff)r  <Cmd>CocCommand fzf-preview.ProjectFiles<CR>
nnoremap <silent> <Plug>(ff)s  <Cmd>CocCommand fzf-preview.GitStatus<CR>
nnoremap <silent> <Plug>(ff)gg <Cmd>CocCommand fzf-preview.GitActions<CR>
nnoremap <silent> <Plug>(ff)b  <Cmd>CocCommand fzf-preview.Buffers<CR>
nnoremap          <Plug>(ff)f  :<C-u>CocCommand fzf-preview.ProjectGrep --add-fzf-arg=--exact --add-fzf-arg=--no-sort<Space>

nnoremap <silent> <Plug>(lsp)q  <Cmd>CocCommand fzf-preview.CocCurrentDiagnostics<CR>
nnoremap <silent> <Plug>(lsp)rf <Cmd>CocCommand fzf-preview.CocReferences<CR>
nnoremap <silent> <Plug>(lsp)d  <Cmd>CocCommand fzf-preview.CocDefinition<CR>
nnoremap <silent> <Plug>(lsp)t  <Cmd>CocCommand fzf-preview.CocTypeDefinition<CR>
nnoremap <silent> <Plug>(lsp)o  <Cmd>CocCommand fzf-preview.CocOutline --add-fzf-arg=--exact --add-fzf-arg=--no-sort<CR>

"" treesitter
lua <<EOF
require('nvim-treesitter.configs').setup {
  ensure_installed = {
    "typescript",
    "tsx",
  },
  highlight = {
    enable = true,
  },
}
EOF

"" gruvbox
colorscheme gruvbox-material
```

## nvim-lspを用いた設定

### プラグインマネージャ

nvim-lspでは [lazy.nvim](https://github.com/folke/lazy.nvim) を採用します。
vim-plugではない理由はnvim-lspはluaで設定を書くのでLua製のプラグインマネージャを使いたかったからです。

[packer.nvim](https://github.com/wbthomason/packer.nvim)と迷ったのですが、lazy.nvimの方が仕組み的にハマりにくそうだったので採用することにしました。
lazy.nvimは最近公開されたばかりでpacker.nvimの方が実績はあるので今後変更するかもしれません。

### Language Serverの管理

coc.nvimでは拡張がLanguage Serverの管理をかなり隠蔽してくれるのですがnvim-lspではこれらの設定が表出しやすいです。
今回は以下のプラグインを組み合わせて使うことでTypeScriptの開発環境を整備します。

- Language Serverを管理するプラグインとして[mason.nvim](https://github.com/williamboman/mason.nvim)
- mason.nvimで導入したLanguage Serverの設定を簡易に行うための[mason-lsoconfig.nvim](https://github.com/williamboman/mason-lspconfig.nvim)
- prettierなどのツールをLSとして扱うためのプラグインの[null-ls.nvim](https://github.com/jose-elias-alvarez/null-ls.nvim)
- masonとnull-lsを繋ぎ込むプラグインの[mason-null-ls.nvim](https://github.com/jay-babu/mason-null-ls.nvim)

### コード補完

[nvim-cmp](https://github.com/hrsh7th/nvim-cmp)というプラグインを採用します。
nvim-cmpはLua製の補完プラグインで、nvim-lspとの組み合わせで最もシェアが高いです。

nvim-cmp本体は補完ソースを提供していないため、補完を行うにはcmp-nvim-lspなどのソースプラグインを導入する必要があります。
また、nvim-cmpのUIを[lspkind-nvim](https://github.com/onsails/lspkind.nvim)というプラグインで拡張するなどといったこともできるため今回導入しています。

### ファジーファインダー

Lua製のファジーファインダーの[telescope.nvim](https://github.com/nvim-telescope/telescope.nvim)を採用します。
開発が活発で、Neovimユーザの中ではシェアが最も高いです。

### LSPクライアントのUI

#### [lspsaga.nvim](https://github.com/kkharji/lspsaga.nvim)

lspsaga.nvimというプラグインでnvim-lspの標準のUIを拡張します。
標準のUIは簡素すぎたり一部使いづらいものがあったりするのですが、lspsagaを使うことでモダンなUIを使うことができます。

#### [dressing.nvim](https://github.com/stevearc/dressing.nvim)

dressing.nvimは導入しておくことでNeovim組み込みのUIが呼び出される際に外部ライブラリを使うようにoverrideしてくれるプラグインです。
LSPの機能を呼び出したときの一部の標準UIは使いづらいものがあるため導入しておきます。

## nvim-lspの設定ファイル

### 準備

以下の設定ファイルを `~/.config/nvim/init.lua` に配置してからNeovim(可能であればHEAD)を起動するだけで導入が完了するはずです。

coc.nvimの設定にも書いたのですが、 `init.vim` と `init.lua` を両方配置すると正常に設定が読み込まれなくなってしまうので、片方を使う際はもう片方を削除してください。

### 設定ファイル

```lua
local lazypath = vim.fn.stdpath('data') .. '/lazy/lazy.nvim'
if not vim.loop.fs_stat(lazypath) then
  vim.fn.system({
    'git',
    'clone',
    '--filter=blob:none',
    'https://github.com/folke/lazy.nvim.git',
    '--branch=stable', -- latest stable release
    lazypath,
  })
end
vim.opt.rtp:prepend(lazypath)

require('lazy').setup({
  'vim-jp/vimdoc-ja',
  'neovim/nvim-lspconfig',
  'williamboman/mason.nvim',
  'williamboman/mason-lspconfig',
  'jose-elias-alvarez/null-ls.nvim',
  'jayp0521/mason-null-ls.nvim',
  'stevearc/dressing.nvim',
  'tami5/lspsaga.nvim',
  'ray-x/lsp_signature.nvim',
  'onsails/lspkind-nvim',
  'j-hui/fidget.nvim',
  'hrsh7th/nvim-cmp',
  'hrsh7th/cmp-nvim-lsp',
  'hrsh7th/cmp-buffer',
  'hrsh7th/cmp-path',
  'lambdalisue/fern.vim',
  'nvim-lua/plenary.nvim',
  'nvim-lua/telescope.nvim',
  'nvim-treesitter/nvim-treesitter',
  'sainnhe/gruvbox-material',
})

-- set options
vim.opt.termguicolors = true
vim.opt.number = true
vim.opt.updatetime = 500
vim.opt.expandtab = true
vim.opt.tabstop = 2
vim.opt.shiftwidth = 2

-- map prefix
vim.g.mapleader = ' '
vim.keymap.set({ 'n', 'x' }, '<Space>', '<Nop>')
vim.keymap.set({ 'n', 'x' }, '<Plug>(lsp)', '<Nop>')
vim.keymap.set({ 'n', 'x' }, 'm', '<Plug>(lsp)')
vim.keymap.set({ 'n', 'x' }, '<Plug>(ff)', '<Nop>')
vim.keymap.set({ 'n', 'x' }, ';', '<Plug>(ff)')

-- telescope.nvim
require('telescope').setup({
  defaults = {
    mappings = {
      n = {
        ['<Esc>'] = require('telescope.actions').close,
        ['<C-g>'] = require('telescope.actions').close,
      },
      i = {
        ['<C-g>'] = require('telescope.actions').close,
      },
    },
  },
})
vim.keymap.set({ 'n' }, '<Plug>(ff)r', '<Cmd>Telescope find_files<CR>')
vim.keymap.set({ 'n' }, '<Plug>(ff)s', '<Cmd>Telescope git_status<CR>')
vim.keymap.set({ 'n' }, '<Plug>(ff)b', '<Cmd>Telescope buffers<CR>')
vim.keymap.set({ 'n' }, '<Plug>(ff)f', '<Cmd>Telescope live_grep<CR>')

-- nvim-lsp
local lsp_config = require('lspconfig')
local mason = require('mason')
local mason_lspconfig = require('mason-lspconfig')
local mason_null_ls = require('mason-null-ls')
local null_ls = require('null-ls')

require('dressing').setup()
require('lspsaga').setup()
require('lsp_signature').setup({ hint_enable = false })
require('fidget').setup()

mason.setup()
mason_null_ls.setup({
  ensure_installed = { 'prettier' },
  automatic_installation = true,
})
null_ls.setup({
  sources = { null_ls.builtins.formatting.prettier },
})

mason_lspconfig.setup({
  ensure_installed = {
    'tsserver',
    'eslint',
  },
  automatic_installation = true,
})

mason_lspconfig.setup_handlers({
  function(server_name)
    local opts = {
      capabilities = require('cmp_nvim_lsp').default_capabilities(),
    }

    lsp_config[server_name].setup(opts)
  end,
})

vim.api.nvim_create_autocmd({ 'CursorHold' }, {
  pattern = { '*' },
  callback = function()
    require('lspsaga.diagnostic').show_cursor_diagnostics()
  end,
})

vim.api.nvim_create_autocmd({ 'FileType' }, {
  pattern = { 'typescript', 'typescriptreact', 'typescript.tsx' },
  callback = function()
    vim.keymap.set({ 'n' }, '<Plug>(lsp)f', function()
      vim.cmd([[EslintFixAll]])
      vim.lsp.buf.format({ name = 'null-ls' })
    end)
  end,
})

local function show_documentation()
  local ft = vim.opt.filetype._value
  if ft == 'vim' or ft == 'help' then
    vim.cmd([[execute 'h ' . expand('<cword>') ]])
  else
    require('lspsaga.hover').render_hover_doc()
  end
end

vim.keymap.set({ 'n' }, 'K', show_documentation)
vim.keymap.set({ 'n' }, '<Plug>(lsp)a', require('lspsaga.codeaction').code_action)
vim.keymap.set({ 'n' }, '<Plug>(lsp)rn', require('lspsaga.rename').rename)
vim.keymap.set({ 'n' }, '<Plug>(lsp)q', '<Cmd>Telescope diagnostics<CR>')
vim.keymap.set({ 'n' }, '<Plug>(lsp)n', require('lspsaga.diagnostic').navigate('next'))
vim.keymap.set({ 'n' }, '<Plug>(lsp)p', require('lspsaga.diagnostic').navigate('prev'))
vim.keymap.set({ 'n' }, '<Plug>(lsp)f', vim.lsp.buf.format)
vim.keymap.set({ 'n' }, '<Plug>(lsp)i', '<Cmd>Telescope lsp_implementations<CR>')
vim.keymap.set({ 'n' }, '<Plug>(lsp)t', '<Cmd>Telescope lsp_type_definitions<CR>')
vim.keymap.set({ 'n' }, '<Plug>(lsp)rf', '<Cmd>Telescope lsp_references<CR>')

-- nvim-cmp
local cmp = require('cmp')
local lspkind = require('lspkind')

cmp.setup({
  enabled = true,
  mapping = cmp.mapping.preset.insert({
    ['<C-u>'] = cmp.mapping.scroll_docs(-4),
    ['<C-d>'] = cmp.mapping.scroll_docs(4),
    ['<C-Space>'] = cmp.mapping.complete(),
    ['<CR>'] = cmp.mapping.confirm({ select = true }),
  }),
  window = {
    completion = cmp.config.window.bordered(),
    documentation = cmp.config.window.bordered(),
  },
  sources = cmp.config.sources({
    { name = 'nvim_lsp' },
    { name = 'buffer' },
    { name = 'path' },
  }),
  formatting = {
    fields = { 'abbr', 'kind', 'menu' },
    format = lspkind.cmp_format({
      mode = 'text',
    }),
  },
})

-- fern.vim
vim.keymap.set({ 'n' }, '<Leader>e', '<Cmd>Fern . -drawer<CR>')
vim.keymap.set({ 'n' }, '<Leader>E', '<Cmd>Fern . -drawer -reveal=%<CR>')

-- treesitter
require('nvim-treesitter.configs').setup({
  ensure_installed = {
    'typescript',
    'tsx',
  },
  highlight = {
    enable = true,
  },
})

-- gruvbox
vim.cmd.colorscheme('gruvbox-material')
```

## 設定を終えて

coc.nvimとnvim-lspそれぞれ最小の設定を実際に書いてみました。
比較するとcoc.nvimは約100行、nvim-lspは約200行となりました。
プラグイン数でいうとcoc.nvimが約10個、nvim-lspが約20個です。

最低限記述しないといけない共通の部分などもあるので、実際に必要な設定記述量はcoc.nvimに比べてnvim-lspの方がやはり3-4倍くらいはかかっているのかなと思います。
ただ、その分nvim-lspの方が挙動の把握や制御はしやすいので一長一短です。luaで設定を書けるというのも人によってはメリットです。
これを踏まえた上でどちらを採用するかを決めるといいのではないかと思います。

## 実践

設定した全ての機能ではないですが、coc.nvim, nvim-lspそれぞれのメインの機能について解説します。

### Completion

補完はInsert Modeで文字を入力すれば自動的に補完候補が表示され、候補を選択できます。
また、横に型やdocの情報が表示され、確定時にauto importも行われます。

#### coc.nvim
![completion_coc](https://user-images.githubusercontent.com/5423775/210471358-c0118b2d-0b64-4a9c-9ab5-63dab71b0aa4.png)

#### nvim-lsp
![completion_nvim-lsp](https://user-images.githubusercontent.com/5423775/210471377-46bc6724-6c21-4fdf-8c14-b5151b74227c.png)

### Diagnostics

エラーや警告の表示については自動で表示され、該当箇所にカーソルを合わせるとエラー内容が出力されます。
また、Fuzzy Finderを使うことで( `mq` を押下)ファイル内のエラー・警告一覧をリスト表示できます。

#### coc.nvim
![diagnostics_coc](https://user-images.githubusercontent.com/5423775/210471767-0e16af89-6d7f-4a61-a48f-d75679ea7a57.png)

#### coc.nvim + fuzzy finder
![diagnostics_coc_fzf](https://user-images.githubusercontent.com/5423775/210471976-ca318d4f-2f05-40c3-a1e5-ef1fbed349ff.png)

#### nvim-lsp
![diagnostics_nvim-lsp](https://user-images.githubusercontent.com/5423775/210471777-00e6f138-01df-49d2-a35d-d7f40a79f6c3.png)

### Hover

型情報を確認したい変数などで `K` を押下することで型情報がHoverで表示されます。

#### coc.nvim
![hover_coc](https://user-images.githubusercontent.com/5423775/210472248-29c411e7-bc05-4f2a-84ea-b61c8d46513b.png)

#### nvim-lsp
![hover_nvim-lsp](https://user-images.githubusercontent.com/5423775/210472261-bcaa624a-72ab-442d-b5e4-cd4535b6108b.png)

### Format

`mf` を押下することで現在開いているファイルにeslint fixとprettierが実行されます。

### Rename

変数名などの上で `mrn` を押下するとLSPの機能を使って変数名などの一括Renameを行う事ができます。

### Code Action

エラーや警告が発生している場合に、その上で `ma` を押下するとLSPを使った自動修正可能な候補一覧が表示され、選択したものを実行できます。

#### coc.nvim
![code_action_coc](https://user-images.githubusercontent.com/5423775/210472534-edeae6f0-c7ab-4915-b44d-f562e43ac03a.png)

#### nvim-lsp
![code_action_nvim-lsp](https://user-images.githubusercontent.com/5423775/210472545-a9a714d1-680d-4be4-87d8-237b8baac6b5.png)

### Fuzzy File Search

`;r` を押下することでプロジェクト内のファイルを一覧表示し、インタラクティブに検索してファイルを開くことができます。

#### coc.nvim
![open_file_coc](https://user-images.githubusercontent.com/5423775/210472832-ba5cf4d9-1405-440a-a274-22532c675569.png)

#### nvim-lsp
![open_file_nvim-lsp](https://user-images.githubusercontent.com/5423775/210472844-1662dea4-b443-411d-b5b3-a1515cb1392c.png)

### Interactive Grep

`;f` を押下してから任意のワード(ripgrepの引数)を入力して確定することで、grep結果を一覧表示してからその中で更に絞り込んでファイルの該当行を開くことができます。

#### coc.nvim
![grep_coc](https://user-images.githubusercontent.com/5423775/210472951-e7481cbe-1f49-4fe1-8c9a-183ecaab604e.png)

#### nvim-lsp
![grep_nvim-lsp](https://user-images.githubusercontent.com/5423775/210472960-8fa1d00d-93cc-44a8-a007-9e05c0038dd3.png)

### References

変数や型の上で `mrf` を押下するとその変数や型を参照している箇所を一覧表示し、選択した箇所を開くことができます。

#### coc.nvim
![references_coc](https://user-images.githubusercontent.com/5423775/210473042-27eacd0d-041b-48cd-8f9c-a10ffb088a65.png)

#### nvim-lsp
![references_nvim-lsp](https://user-images.githubusercontent.com/5423775/210473054-9bd86a42-bb47-4034-b732-2f81bc9c290c.png)

### ファイラ

`<Space>e` を押下することでファイラを開くことができます。
また、 `<Space>E` で開いているファイルのディレクトリを展開してファイラを開きます。

ファイラの操作については `:h fern` でヘルプを引くか、ファイラのバッファで `?` を押下することで機能一覧を表示できます。
![fern](https://user-images.githubusercontent.com/5423775/210473364-6a920b99-7bd6-4040-b13b-588372470dc4.png)

## 更にVimを使いこなすために

昨年の記事にも同じ事を書いたのですが、もし本格的にVimを使い込みたい場合には以下の3つに挑戦してみることでVimへの知識を深めることができると思います。

### [vim-jp Slack](https://vim-jp.org/docs/chat.html)へ参加する

[エンジニアの楽園 vim-jp](https://blog.tomoya.dev/posts/vim-jp-is-a-paradise-for-engineers/)で紹介されているvim-jpコミュニティのSlackです。
詳細は上の記事を見ていただければ分かると思うのですが日々500-1500前後の発言が流れており、コミュニケーションが非常に活発です。

Vimについて分からないことなどがあればすぐに返答が返ってくる環境なので、Vimについて学んでいきたいユーザにとてもおすすめです。

### [doc](https://github.com/vim-jp/vimdoc-ja)を読む(vim-jp/vimdoc-ja)

`vim-jp/vimdoc-ja` はvimのヘルプの日本語翻訳プラグインで、導入することでVimから日本語のhelpが読めるようになります。(前述のvimrcに含まれています)
また、上記のvimrcの設定だとvimファイルの中で `K` を押下することでカーソル上のキーワードのhelpを引くことができます。
分からない単語が出てきた場合も基本的に `:h {word}` でhelpが表示されます。

ただ、Neovimのhelpを翻訳したものではないためNeovimの実装/helpと差違があることがあります。
その場合の英語のhelpには `:h {word}@en` でアクセスできるのでご活用ください。

Vimのhelpは非常に充実しているので、分からないことがあればすぐに調べることができて非常に便利です。

### [実践Vim](https://tatsu-zine.com/books/practical-vim)を読む

Vimの使い方や考え方についてとてもいい内容が書かれている本です。
個人的にはVim関連の書籍では最も読む価値があり、バイブルだと思っています。(vim-jp Slackでもバイブルと言われています)

余裕があれば実践Vimを読むことでVimをより高度に使えるようになると思います。

## 最後に

この記事では2022年のNeovimでのフロントエンド開発環境構築について紹介しました。

フロントエンドエンジニアは基本的にはVSCodeを使っていると思うのですが、Vimでのテキスト編集は慣れると非常に快適なので、この記事を機会に一度使ってみて頂けると幸いです。

もしVimについて知りたいことや質問があればvim-jp Slackや[Twitter](https://twitter.com/yuki_ycino)で連絡を頂ければ答えられると思うので、是非お願いします。
それではより良いVimライフをお送りください！
