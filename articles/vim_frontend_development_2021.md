---
title: "Neovimでのフロントエンド開発環境 2021"
emoji: "⚡"
type: "tech"
topics: ["Vim", "Neovim", "LSP", "TypeScript", "フロントエンド"]
published: false
---

:::message
この記事は[Vim Advent Calendar 2021-2](https://qiita.com/advent-calendar/2021/vim)の17日目の記事です。
:::

最近はフロントエンドエンジニア(主にReact)をしているYano ([@yuki_ycino](https://twitter.com/yuki_ycino)) といいます。
この記事では自分が開発に使っているNeovim周りの環境と、その大まかな構成について書いていきます。

関係ないですが最近Among Usというゲームをやっていて、金曜日にamong us engineerというDiscordサーバでよくプレイしています。
[一番下](#おまけ)にこっそり招待リンクを書いておくので興味のあるエンジニアの方は覗いてみてください。

## はじめに

自分は現在フロントエンド開発のほぼ全てをNeovimで行っています。
個人的にはVSCodeと遜色なく(むしろ効率よく)開発できていると思うのですが、VSCodeではシンプルな設定で開発を始められるのに対し、Vimはどうしても設定のハードルが高いです。

具体的には現在5000行弱のVimの設定ファイルと100個前後のプラグインを導入して開発しています。
ただ、それを他の方に勧めるのは無茶なので、この記事ではほぼ最小構成に近いTypeScript開発環境を紹介していきたいと思います。
逆に特殊な編集機能を提供するプラグインや、Vimの操作自体を改善するようなプラグインについてはこの記事では紹介しません。

この記事では主にLSPによる開発サポート及び現代の開発における必須プラグイン(LSPプラグイン・ファジーファインダー・補完プラグイン・ファイラ)の紹介をメインに進めています。
最近はVimによる開発環境やプラグインの選択肢が多いために情報が錯綜していると思っており、そのあたりの整理も軽くできたらなと思っています。

類似したタイプのプラグインでも導入の敷居・知識不足・個人的な判断で紹介を割愛しているものもあります。
ある程度バイアスがかかっていると思うのですがご了承ください。

後半ではTypeScriptを書くためのLSP環境 + αだけを整備した小さめのvimrcファイルを公開しておきます。
そのvimrcをベースに設定を追加していくことで好みのVim環境を整えることができると思います。

最後に、[更にVimを使いこなすために](#更にvimを使いこなすために)どのようすればよいかを書いているので、もしよければそこまで読んでいただけると幸いです。

(参考程度に、自分のvimrcのURLも貼っておきます https://github.com/yuki-yano/dotfiles/blob/main/.vimrc)

## VimとNeovim

まず、分かりづらいVimとNeovimの違いについて説明します。

### Neovim (*)

NeovimはVimをForkしてできたOSSです。
近年はVimと袂を分かってきており、それぞれ独自の機能を導入するようになっています。
Vimとの違いとしては、提供されているAPIがプラグイン開発者フレンドリーになっており、高機能なプラグインが比較的作りやすいです。
また、Luaのランタイムが組み込まれており、Vim scriptの実行速度が遅い問題についてLuaを使うことで解決するというアプローチを取っています。
TypeScriptでの開発における大きな違いとしてはbuilt-inでLSPが使えるようになっています。

### Vim

NeovimのFork元で昔から開発が続けられているのがVimです。
どこの環境にも比較的入れやすいのがメリットかと思います。
ただ、提供されている機能やAPIの兼ね合いなどでNeovimに比べてサードパーティのプラグイン実装の自由度が相対的に低いなどの懸念点があると思っています。

### 結論

以上の比較から、今回のLSPを用いたフロントエンド開発ではNeovimを採用します。

## プラグインマネージャ

国内で最もメジャーなプラグインマネージャとしてはvim-plugとdein.vimが上げられると思います。
それぞれ一長一短なので解説します。

他にも[packer.nvim](https://github.com/wbthomason/packer.nvim)など、Neovimに特化したプラグインマネージャなどもいくつかあるのですが今回は割愛します。

### [vim-plug](https://github.com/junegunn/vim-plug) (*)

非常にシンプルなプラグインマネージャです。
プラグインの導入を始めて行う場合はvim-plugを選んでおけばとりあえず間違いないです。
dein.vimと比べての欠点としてはチューニングを行うことが難しいというものがあり、Vimの起動速度をチューニングしたいなどといった場合はdein.vimを採用することになると思います。

### [dein.vim](https://github.com/Shougo/dein.vim)

あのDark powered pluginを開発しているShougoさん製のDark powered plugin managerです。
強力なチューニングができるのでVimの起動速度の最適化を目指す場合などは非常に有用なのですが、初心者が使うにはハードルがかなり高いので個人的にはお勧めしづらいです。

自分の場合はLazy Loadを用いる事で(プラグイン数が100を越えていたので)1200ms前後かかっていた起動時間を200ms強まで短縮できたのですが、遅延ロードの設定が複雑でキャッシュの取り扱いではまっている人も定期的に見るので、よっぽど自信がある方以外はvim-plugから試してみるのがいいかと思います。

### 結論

以上の比較から、この記事で紹介する設定ではvim-plugを採用します。

## LSPプラグイン

TypeScript開発をする上で必須になるのがLSPです。
(Neo)vimにはいくつかの有名なLSPプラグインがあるので、それらについて解説します。

### [coc.nvim](https://github.com/neoclide/coc.nvim) (*)

LSP連携・補完機能・VSCodeからPortingした各種extensionsによる設定の簡易さを全て兼ね備えているAll in Oneのプラグインです。
もう少し分かりやすく言うと、(多少大げさですが)coc.nvimを導入するとVimからVSCodeの機能の大半が使えるようになります。
LSと連携もcoc extensionsという仕組みを使い、コマンド1つで構築できます。環境を整備するには最も楽なプラグインだと思っています。
(例えばTypeScript開発のためにはcoc-tsserverというcoc extensionsのインストールが必要で、後述のvimrcでは自動的にインストールされるよう設定しています)

また、coc.nvim自体がTypeScriptで実装されている都合上VSCodeのextensionsをforkしてVim用にしているものも多く、VSCodeの資産を使い回す事ができるのは大きなメリットです。

気になる点としてNodeに依存しているというものがあるのですが、フロントエンド開発においてはNodeはほぼ確実に導入されているので基本的に問題ないと思っています。

### [vim-lsp](https://github.com/prabirshrestha/vim-lsp)

vim-lspはVim scriptで実装されたLSPプラグインです。
Pure Vim scriptで実装されているの外部依存がないなどのメリットがあります。
また、[vim-lsp-settings](https://github.com/mattn/vim-lsp-settings)という環境整備のためのプラグインが手厚く管理されており、LSの設定にはあまり困らないようになっています。

coc.nvimとの大きな違いとして、All in Oneではないため補完にはVim標準のomnifuncを使うか、より高度な自動補完をするなら補完プラグインを別途導入する必要があります。
[asyncomplete.vim](https://github.com/prabirshrestha/asyncomplete.vim), [ddc.vim](https://github.com/Shougo/ddc.vim), [nvim-cmp](https://github.com/hrsh7th/nvim-cmp) などがあるのですが、このあたりから選定して補完設定を行う必要があります。

### nvim_lsp

nvim_lspはNeovim built-inのLSPクライアントです。built-inなので信頼度は最も高いかもしれません。
nvim_lspについては自分は使ったことがないのですが、設定が大変だという話をよく聞きます。([nvim-lsp-installer](https://github.com/williamboman/nvim-lsp-installer)というプラグインはあるのですが、それにしても設定が難しいという声を多く聞きます)
周辺のエコシステムは徐々に整ってきているように見えるので、自分もそろそろ検証したいと思っています。

現状ではこの3つの中では設定が手間なのも含め、最も人柱に近いLSPクライアントだと思っているので個人的にはあまり推奨していません。

### 結論

自分が使っているのもあるのですが、LSPプラグインは導入・設定がEasyなcoc.nvimを採用します。

## ファジーファインダー

ファジーファインダーはフロントエンド開発とは直接関係はないのですが、近年の開発においてファジーファインダーはほぼ必須になってきていると感じています。
具体的な機能としてはファイルのファジー検索・Grep結果からのファジーマッチなどです。
一部のファジーファインダーはLSPとも連携しており、それらの機能を使うことで開発効率を上げることができます。

現在はfzf.vimのシェアが最も高く、次点でNeovimユーザにtelescope.nvimが使われているかと思います。

### [fzf.vim](https://github.com/junegunn/fzf.vim)

[fzf](https://github.com/junegunn/fzf)というCLIのファジーファインダーをVim上で動作させるプラグインです。
fzfがGo製なので非常に高速に動作し、CLIとしてのfzfが広く使われていることもありVimプラグインとしても非常に有名です。
リソースもVimで作業するための最低限のものが揃っています。

欠点としては、CLIツールをVimのterminal経由で動作させるという設計のために柔軟性がやや低く、いろいろな処理を挟みたくなった場合に非常に複雑な実装をしないといけないということがあります。
また、それに伴いLSPとの連携などは基本的にできません。

### [telescope.nvim](https://github.com/nvim-telescope/telescope.nvim)

Lua製のNeovim用ファジーファインダープラグインです。
開発が活発で、Neovimユーザの中ではシェアがかなり高そうです。

プラグインでリソースを拡張できるので、ユーザが様々なリソースを開発しています。
LSPとの連携も可能ですが、リソース用のプラグインを導入する必要があります。

fzfはWindowsでの動作がかなり怪しい(手元で検証できていない)ので、Windowsで環境構築をする際は今回書いている中だとtelescope.nvimを採用するのがいいかもしれません。

### [fzf-preview.vim](https://github.com/yuki-yano/fzf-preview.vim) (*)

知名度・シェアでいうと上記2つより劣るのですが、自分が開発フローの中でヘビーに使っているのでここで紹介させてもらいます。
自作のTypeScript製のファジーファインダーでNodeに依存しており、内部的にfzfを使用しています。
All in One系のプラグインになっており、基本的に組み込みでリソースを提供していて拡張性はあまりありません。
その代わりに非常に少ない設定で多くのリソースとfzf.vimよりリッチなUIが使えるようにしています。

また、提供している機能についてもfzf.vimより豊富です。
例を挙げると[gina.vim](https://github.com/lambdalisue/gina.vim)などと連携してgitのインタラクティブな操作するなどができます。
LSPについてはcoc.nvim・vim-lspのリソースに対応しているのが特徴で、特にcoc.nvimと組み合わせると非常に便利です。

fzf-preview.vim及び他のファジーファインダーの情報は主に以下の記事にまとめられています。

https://zenn.dev/yano/articles/vim_with_fzf_preview_is_best_experience
https://zenn.dev/yutakatay/articles/vim-fuzzy-finder

### 結論

正直ポジショントークな部分もあるのですが、今回はlspプラグインにcoc.nvimを採用するのと、自作のためバグフィックスやサポートが十分できるという理由でfzf-preview.vimを採用します。

## 補完プラグイン

### [coc.nvim](https://github.com/neoclide/coc.nvim) (*)

LSPプラグインのところでも解説しましたが、coc.nvimはAll in Oneのプラグインとなっており、標準で補完機能が付いています。
coc extensionsを導入することでLSPを使った補完にもデフォルトで対応しているので、coc.nvimを導入した場合は補完プラグインの設定は不要です。

### [ddc.vim](https://github.com/Shougo/ddc.vim)

ddc.vimは[denops.vim](https://github.com/vim-denops/denops.vim)製の補完プラグインです。
デフォルトでは設定は存在せず、ユーザが周辺のプラグインを選んだ上で全ての設定を自分で書く必要があります。
例えばcoc.nvimでは補完候補のマッチ・ソートアルゴリズムなどが本体に組み込まれていて変更不可能なのに対して、ddc.vimはそのあたりについても全て自分で設定する必要があります。

非常に強力なプラグインなのですが、使いこなすまでの敷居がかなり高いです。

### 結論

ddc.vimの設定難易度の高さ、またcoc.nvimを導入する前提で進めているので補完プラグインとしてもcoc.nvimを採用します。

## ファイラ

VSCodeやIntelliJ系ではサイドバーにファイラが表示されていると思うのですが、Vimでもそれがないと不便なことが多いのでファイラを導入します。
ファイラの詳細な比較は以下を参考にしてみてください。

https://zenn.dev/lambdalisue/articles/3deb92360546d526381f

### [fern.vim](https://github.com/lambdalisue/fern.vim) (*)

fern.vimはPure Vim scriptで実装され、さまざまな機能が提供されているファイラプラグインです。
外部依存なく高速に動作し、個人的にはファイラとして必要な機能が網羅されているので最近はfern.vimを常用しています。

### [defx.nvim](https://github.com/Shougo/defx.nvim)

defx.nvimはPythonで実装されたNeovimのRemote Pluginです。
fern.vimとの大きな違いとしてはPythonに依存している点と、デフォルトではmappingが一切行われていないという点で、上級者向けのプラグインとなっています。
その代わりに拡張性は非常に高いファイラプラグインです。

### [coc-explorer](https://github.com/weirongxu/coc-explorer)

coc extensionsで実装されたファイラプラグインです。
coc.nvimを使っていると導入が楽なのと、Floating Windowでの情報の表示などが優秀なのですが、他に特筆すべき機能がないのであまり積極的に採用しなくていいかと思っています。
また、以前試した時は動作が重かったと記憶しています。

### 結論

上記を総合してfern.vimを採用します。

## ColorSchemeとHighlight

VimのデフォルトのColorSchemeは(個人的に)見た目があまり良くないため、人気があり継続して更新されているColorSchemeを適用します。

### [gruvbox-material](https://github.com/sainnhe/gruvbox-material) (*)

非常に人気があり、さまざまなプラグインにも対応しているColorSchemeです。
後述のtreesitterにも対応しているため、標準のColorSchemeを使うよりもいい感じにハイライトされます。

### [nvim-treesitter](https://github.com/nvim-treesitter/nvim-treesitter) (*)

構文解析をし、Syntax highlightなどを提供してくれるNeovim用プラグインです。
Vimは標準ではSyntax highlightを正規表現で実装しているので、それと比較して高精度かつ豊富なハイライトが行われます。

また、treesitterの拡張プラグインを導入することでSyntax highlight以外にも様々な機能を使うことができるようになります。

## 最終的なvimrc

### 準備

- https://github.com/junegunn/vim-plug/wiki/tutorial に書いているようにvim-plugを導入してください
- 以下のvimrcを配置してからNeovim(可能であればHEAD)を起動し、 `:PlugInstall` を実行してプラグインをインストールします(初回起動のインストール前にエラーが表示されると思いますが無視してください)
  - 可能であればfzf-preview.vimが内部的に使用する[bat](https://github.com/sharkdp/bat)や[ripgrep](https://github.com/BurntSushi/ripgrep)を先に導入しておいてください

### .vimrc

```vim
" Install Plugin
call plug#begin('~/.vim/plugged')

Plug 'vim-jp/vimdoc-ja'
Plug 'junegunn/fzf', {'dir': '~/.fzf_bin', 'do': './install --all'}
Plug 'neoclide/coc.nvim', {'branch': 'release'}
Plug 'lambdalisue/fern.vim'
Plug 'lambdalisue/gina.vim'
Plug 'nvim-treesitter/nvim-treesitter'
Plug 'sainnhe/gruvbox-material'

call plug#end()

" set options
set termguicolors
set number

" map prefix
let g:mapleader = "\<Space>"
nnoremap <Leader> <Nop>
xnoremap <Leader> <Nop>
nnoremap [dev]    <Nop>
xnoremap [dev]    <Nop>
nmap     m        [dev]
xmap     m        [dev]
nnoremap [ff]     <Nop>
xnoremap [ff]     <Nop>
nmap     z        [ff]
xmap     z        [ff]

"" coc.nvim
let g:coc_global_extensions = ['coc-tsserver', 'coc-eslint8', 'coc-prettier', 'coc-git', 'coc-fzf-preview', 'coc-lists']

inoremap <silent> <expr> <C-Space> coc#refresh()
nnoremap <silent> K       :<C-u>call <SID>show_documentation()<CR>
nmap     <silent> [dev]rn <Plug>(coc-rename)
nmap     <silent> [dev]a  <Plug>(coc-codeaction-selected)iw

function! s:coc_typescript_settings() abort
  nnoremap <silent> <buffer> [dev]f :<C-u>CocCommand eslint.executeAutofix<CR>:CocCommand prettier.formatFile<CR>
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

nnoremap <silent> <C-p>  :<C-u>CocCommand fzf-preview.FromResources buffer project_mru project<CR>
nnoremap <silent> [ff]s  :<C-u>CocCommand fzf-preview.GitStatus<CR>
nnoremap <silent> [ff]gg :<C-u>CocCommand fzf-preview.GitActions<CR>
nnoremap <silent> [ff]b  :<C-u>CocCommand fzf-preview.Buffers<CR>
nnoremap          [ff]f  :<C-u>CocCommand fzf-preview.ProjectGrep --add-fzf-arg=--exact --add-fzf-arg=--no-sort<Space>
xnoremap          [ff]f  "sy:CocCommand fzf-preview.ProjectGrep --add-fzf-arg=--exact --add-fzf-arg=--no-sort<Space>-F<Space>"<C-r>=substitute(substitute(@s, '\n', '', 'g'), '/', '\\/', 'g')<CR>"

nnoremap <silent> [ff]q  :<C-u>CocCommand fzf-preview.CocCurrentDiagnostics<CR>
nnoremap <silent> [ff]rf :<C-u>CocCommand fzf-preview.CocReferences<CR>
nnoremap <silent> [ff]d  :<C-u>CocCommand fzf-preview.CocDefinition<CR>
nnoremap <silent> [ff]t  :<C-u>CocCommand fzf-preview.CocTypeDefinition<CR>
nnoremap <silent> [ff]o  :<C-u>CocCommand fzf-preview.CocOutline --add-fzf-arg=--exact --add-fzf-arg=--no-sort<CR>

"" fern
nnoremap <silent> <Leader>e :<C-u>Fern . -drawer<CR>
nnoremap <silent> <Leader>E :<C-u>Fern . -drawer -reveal=%<CR>

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

## 実践

設定した全ての機能ではないですが、メインの機能について解説します。

### Completion

補完はInsert Modeで文字を入力すれば自動的に補完候補が表示され、候補を選択できます。
また、横に型やdocの情報が表示され、確定時に自動importも行われます。
![completion](https://user-images.githubusercontent.com/5423775/146121464-58f182be-028e-4706-a191-8378076f0ab4.png)

### Diagnostics

エラーや警告の表示については自動で表示され、該当箇所にカーソルを合わせるとエラー内容が出力されます。
また、fzf-preview.vimを使うことで( `zq` を押下)ファイル内のエラー・警告一覧をリスト表示できます。
![diagnostics](https://user-images.githubusercontent.com/5423775/146121488-709cd5a9-9a02-49ed-b48a-de5fa82f7a0b.png)
![diagnostics_fzf](https://user-images.githubusercontent.com/5423775/146122564-163b855c-15be-42c9-95fc-ee0cf5cd0075.png)

### Hover

型情報を確認したい変数などで `K` を押下することで型情報がHoverで表示されます。
![hover](https://user-images.githubusercontent.com/5423775/146121578-e3866599-9c9c-449a-9a81-294998e5dc42.png)

### Format

`mf` を押下することで現在開いているファイルにeslint fixとprettierが実行されます。

### Rename

変数名などの上で `mrn` を押下するとLSPの機能を使って変数名などの一括Renameを行う事ができます。

### Code Action

エラーや警告が発生している場合に、その上で `ma` を押下するとLSPを使った自動修正可能な候補一覧が表示され、選択したものを実行できます。
![code_action](https://user-images.githubusercontent.com/5423775/146127391-a1b45d83-2c82-45e4-87a2-6d1a8136218b.png)

### Fuzzy File Search

`<C-p>` を押下することでプロジェクト内のファイルを一覧表示し、インタラクティブに検索してファイルを開くことができます。
![open_file](https://user-images.githubusercontent.com/5423775/146122500-acba5d00-d863-44ab-9b02-9edfb8220ca0.png)

### Interactive Grep

`zf` を押下してから任意のワード(ripgrepの引数)を入力して確定することで、grep結果を一覧表示してからその中で更に絞り込んでファイルの該当行を開くことができます。
![grep](https://user-images.githubusercontent.com/5423775/146122715-d84d9dbb-6ddd-469b-bee4-b475b924ff47.png)

### Git status and commit

`zs` を押下することでgit statusを表示し、差分を確認しつつAdd, Reset, Commitなどが実行できます。
![git](https://user-images.githubusercontent.com/5423775/146122742-8fe6d156-1d5e-46b3-a084-f3a11faf76ae.png)

### References

変数や型の上で `zrf` を押下するとその変数や型を参照している箇所を一覧表示し、選択した箇所を開くことができます。
![references](https://user-images.githubusercontent.com/5423775/146122848-ba28a00d-bdce-4b59-b152-e8bfc78860c6.png)

### Outline

`zo` を押下することで現在開いているファイルのoutlineを表示し、選択した箇所にジャンプできます。
![outline](https://user-images.githubusercontent.com/5423775/146122937-359bcb43-2989-4075-be13-53f0369140e1.png)

### ファイラ

`<Space>e` を押下することでファイラを開くことができます。
また、 `<Space>E` で開いているファイルのディレクトリを展開してファイラを開きます。

ファイラの操作については `:h fern` でヘルプを引くか、ファイラのバッファで `?` を押下することで機能一覧を表示できます。
![fern](https://user-images.githubusercontent.com/5423775/146125344-653eabb1-06d0-4bfe-9649-f6d70052784e.png)

## 更にVimを使いこなすために

もし本格的にVimを使い込みたい場合には以下の3つに挑戦してみることでVimへの知識を深めることができると思います。

### [vim-jp Slack](https://vim-jp.org/docs/chat.html)へ参加する

[エンジニアの楽園 vim-jp](https://blog.tomoya.dev/posts/vim-jp-is-a-paradise-for-engineers/)で紹介されているvim-jpコミュニティのSlackです。
詳細は上の記事を見ていただければ分かると思うのですが日々500-1500前後の発言が流れており、コミュニケーションが非常に活発です。

Vimについて分からないことなどがあればすぐに返答が返ってくる環境なので、Vimについて学んでいきたいユーザにとてもおすすめです。

### [doc](https://github.com/vim-jp/vimdoc-ja)を読む(vim-jp/vimdoc-ja)

`vim-jp/vimdoc-ja` プラグインを入れることでVimから日本語のhelpが読めるようになります。(前述のvimrcに含まれています)
また、上記のvimrcの設定だとvimファイルの中で `K` を押下することでカーソル上のキーワードのhelpを引くことができます。
分からない単語が出てきた場合も基本的に `:h {word}` でhelpが表示されます。

Vimのhelpは非常に充実しているので、分からないことがあればすぐに調べることができて非常に便利です。

### [実践Vim](https://tatsu-zine.com/books/practical-vim)を読む

Vimの使い方や考え方についてとてもいい内容が書かれている本です。
個人的にはVim関連の書籍では最も読む価値があり、バイブルだと思っています。(vim-jp Slackでもバイブルと言われています)

余裕があれば実践Vimを読むことでVimをより高度に使えるようになると思います。

## 最後に

この記事ではNeovimでのフロントエンド開発環境構築について紹介しました。

フロントエンドエンジニアは基本的にはVSCodeを使っていると思うのですが、Vimでのテキスト編集は慣れると非常に快適なので、この記事を機会に一度使ってみて頂けると幸いです。
(VSCodeのVimプラグインはVimユーザからするといろいろ厳しい点が多いです…)
特にtextobjとoperatorなどが使えるようになると他のエディタに移れなくなるほど快適なので、そのあたりについても時間があれば記事を書きたいと思っています。

量は少ないですが、プラグインの解説記事なども書いているので興味がある方は読んでみてください。
https://zenn.dev/yano/articles/vim_plugin_top_10

もしVimについて知りたいことや質問があればvim-jp Slackや[Twitter](https://twitter.com/yuki_ycino)で連絡を頂ければ答えられると思うので、是非お願いします。
それではより良いVimライフをお送りください！

## おまけ

[among us engineerサーバの招待URL](https://discord.gg/kbDyjUvp)
