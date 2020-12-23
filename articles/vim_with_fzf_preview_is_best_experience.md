---
title: "TypeScriptでVimのファジーファインダーを実装して開発体験が最高になっている話"
emoji: "🔍"
type: "tech"
topics: ["Vim", "neovim", "fzf", "TypeScript"]
published: true
---

:::message
この記事は[Vim Advent Calendar 2020](https://qiita.com/advent-calendar/2020/vim)の24日目の記事です。
:::

## はじめに

この記事は自分が開発しているTypeScript製Vimプラグイン、[fzf-preview.vim](https://github.com/yuki-ycino/fzf-preview.vim)についての記事となっています。
以下のような内容について書いています。

- 自分のエディタ(Vim)との付き合い方
- ファジーファインダーを用いた開発体験・開発フロー
- プラグインの機能
- プラグイン開発のモチベーション
- プラグインの導入など

色々と書いていたら結構な量になってしまったので、動作を見たい場合は[fzf-preview.vimをメイン使った作業フロー](#fzf-preview.vimをメイン使った作業フロー)や[Gitとの強力な連携](#gitとの強力な連携)あたりまで飛ぶといいかもしれません。

## 私の開発環境とプラグイン開発

自分は基本的にWebエンジニアとして活動しており、最近はTypeScript(React)での開発がメイン、たまにRuby(Rails)を書いたりしています。
記事のタイトル通りエディタはほぼVim(厳密にはNeovim)を使っており、現代のメインストリームであるVSCodeは軽くしか触れられていません。

VSCodeはエコシステムを含めて非常に優れたエディタだと思っていますが、Vimコミュニティも活発なユーザが多く非常に良い環境で、ずっとVimを使い続けています。
そして、現代ではLSPの発展が目覚ましく、VimでもLSPクライアントを使うことでVSCodeに近い開発体験を得ることができるようになってきています。
特にTypeScriptの開発についてはVSCodeに割と近いことができているのではないかと思っています。

最近は補完やジャンプはもちろん、Code Actionを実行するUIもリッチになっていてびっくりしました。

:::message
これはVimの標準機能ではなく[coc.nvim](https://github.com/neoclide/coc.nvim)の機能です。
:::
@[tweet](https://twitter.com/yuki_ycino/status/1330869140211474433)

なお、現在の自分のvimrcは3505行(コメント + 空行が約500行)・導入しているプラグイン数は111個になっており、ある程度カスタマイズするのが前提になっているので1から開発環境を整えるにはVSCodeを使うのが無難かもしれません…
カスタマイズしたVimは非常に快適ですがとても深い沼です。

https://github.com/yukiycino-dotfiles/dotfiles

そして、今年のVim活として[TypeScriptでVimプラグインの開発に挑戦](#開発のモチベーション)しました。その結果かなり理想に近いプラグインを作ることができたためVimでの開発が快適になり、更にVimにロックインされるという状態になっています。
その開発しているプラグインが[fzf-preview.vim](https://github.com/yuki-ycino/fzf-preview.vim)です。
(プラグイン名が微妙なのは開発開始当時プレビュー機能が対応していなかったfzf.vimの機能に対して拡張したプラグインだったからで、結果的に想定より遙かに大きくなってしまいました…)

なお、Vim scriptをあまり書かずTypeScriptでの開発に偏っていったため、Vimmerとしては中級者程度かなと思っています…

## モダン開発環境とファジーファインダー、そしてVim

### ファジーファインダーとは

知っている方も多いかとは思いますが、ファジーファインダーについて説明します。

リソース(検索対象)となるリストを受け取り、あいまい検索して候補を選択、その後何らかのアクションを実行するソフトウェアのことをファジーファインダーと呼びます。
正確な名前や正規表現が不要で、高速に対象を絞り込めるのがメリットです。
代表的なCLIでは[fzf](https://github.com/junegunn/fzf)があります。

エディタ上で実行されるファジーファインダーはプロジェクトのファイルリスト全体から対象を高速に検索したり、grep結果をインタラクティブに絞り込んでジャンプするといった機能を提供していることが多いです。

### エディタとファジーファインダー

現代のソフトウェア開発においてコードの実装やリーディングの際に何らかのファジーファインダーを用いるのは半ば常識的になってきています。
分かりやすいものだと、VSCodeでいう `Ctrl+p` (Macでは `Command+p` )のような機能です。

近年は大半のエディタにファジーファインダーが実装されており、特に最近のVimを取り巻く環境では何故かファジーファインダープラグインが乱立しています。
(自分は棚に上げて、既存のを使えばいいんじゃないかという気持ちはありますが)ファジーファインダーがエディタの機能として重要視されているのが分かります。

この記事で解説している[yuki-ycino/fzf-preview.vim](https://github.com/yuki-ycino/fzf-preview.vim)はオールインワン系のファジーファインダープラグインで、徐々にユーザが増えて2020年12月時点でスターを470程度付けて貰えています。
ユーザが増えている理由として、特に海外ではオールインワンなプラグインが好まれる傾向にあるので、そういったものも影響しているのではないかと思います。

有名なファジーファインダープラグインとの客観的な比較は [@yutakatay](https://zenn.dev/yutakatay) さんの記事にまとめられています。
自分があまり比較したことのないプラグインについても整理されているので、興味がある方は一読をお勧めします。

https://zenn.dev/yutakatay/articles/vim-fuzzy-finder

## fzf-preview.vimによる最高の開発体験

自分の開発環境は大半がfzf-preview.vimに依存しています。自分にとって最高のVimプラグインの1つです。
機能として、以下のような操作を一手で行う事ができます。
プラグインからそれぞれコマンドを提供していて、自分は `prefix + {key}` で起動するようにmapしています。

- プロジェクト全体からファイル検索
- 最近開いた(or 保存した)ファイルの選択
- インタラクティブなgrepと対象行へのジャンプ
- エラーが発生しているコードへのジャンプ
- LSPを使っての定義・参照へのジャンプ
- Gitとの強力な連携

上記をファジーファインダーで開き、カーソルが合っている要素を**プレビュー**付きで表示します。
このプレビュー機能が個人的には非常に重要です。

:::message
プレビューはカーソル上のファイル内容を表示し、grepなどの行を扱うものであればファイル内の対象行の前後から表示します。
ショートカットでスクロールできるので、該当ファイルの中を一通り見通してから選択をすることなどが可能です。
また、Git操作ではcommit logやdiffなどを表示します。
:::

つまり、開発におけるファイルを開いたり検索する機能を網羅しているプラグインで、共通のUIでそれらを扱うため様々な操作の起点となっています。

QuickFixへのエクスポートやQuickFixをリソースとして扱う機能も実装しており、他のプラグインとの連携も可能です。
後述の[リファクタリング作業フロー](#リファクタリング)にありますが、[vim-qfreplace](https://github.com/thinca/vim-qfreplace)との連携もその1つです。
また、[gitの様々な操作](#gitとの強力な連携)を提供しています。

以下がgrepを実行している例で、左側で選択している行の周辺がプレビューとして右のブロックに表示されています。
その次の画像ではcommitのdiffを表示しつつgit logを操作しています。この画面からlogに対して様々な操作が可能です。
![grep](https://user-images.githubusercontent.com/5423775/102711979-0d2ccc00-4301-11eb-928b-3471377dde78.png)
*grep with preview*
![git_log](https://user-images.githubusercontent.com/5423775/102713902-a9110480-430e-11eb-9501-7b14bc1bfb1e.png)
*git log with diff preview*

## 他のファジーファインダーと比較しての優位点など

### よりリッチな表示のサポート

Vimのファジーファインダープラグインとしては有名なものに以下のようなものがあります。

- [fzf.vim](https://github.com/junegunn/fzf.vim)
- [telescope.nvim](https://github.com/nvim-telescope/telescope.nvim)
- [denite.nvim](https://github.com/Shougo/denite.nvim)
- [ctrlp.vim](https://github.com/ctrlpvim/ctrlp.vim)

[fzf-preview.vim](https://github.com/yuki-ycino/fzf-preview.vim)とそれら(+ VSCode)を比較したのが下の画像です。
fzf-preview.vimはファイルタイプのアイコンやディレクトリ名のcolorize、ファイルのプレビューなどの機能を実装しており、表示をとてもリッチにしているのが伝わるかと思います。
特にファイル一覧から選択する際のプレビューと、grepやLSP連携の際に対象となっている行周辺のプレビュー(及びスクロール)できるのが重要で、自分はこれによって開発効率が大幅に上がっています。

:::message
画像は各プラグインにcolorizeやdeviconなどの小規模な設定を入れて比較しています。
高解像度なオリジナルファイルにリンクしているので、フォントの細かい部分などはリンク先で確認してみてください。
:::

[![fuzzy_finder_list](https://user-images.githubusercontent.com/5423775/102991641-f97da180-455c-11eb-8555-cb108d391e39.png)](https://user-images.githubusercontent.com/5423775/102991641-f97da180-455c-11eb-8555-cb108d391e39.png)

### デフォルトで提供しているリソースが豊富

ファイル検索でいうと、fzf.vimにはない、最近使ったファイルや最近保存したファイルの順に候補を出す機能なども提供しています。(Most Recent Used, Most Recent Written)
また、LSP(coc.nvim)との連携も実装しており、DiagnosticsやReferencesなどを扱うことができます。
他にもVimの機能であるmark, jumplist, tags, changelistなどと、一部の外部プラグインとの連携にも対応しています。
(一部fzf.vimのみで提供されているリソースもあります。)

![lsp_diagnostics](https://user-images.githubusercontent.com/5423775/102711996-2df52180-4301-11eb-93b5-fb7a32e95128.png)
*LSP Diagnostics*

![lsp_references](https://user-images.githubusercontent.com/5423775/102711997-30577b80-4301-11eb-8a9a-c4a669c4801b.png)
*LSP References*

### 細かい便利な設定が標準で付いている

fzf.vimと比較して空気を読んだ感じのコマンドや設定が標準で付いています。前述したdeviconとの連携もその1つです。
fzf.vimだとそれなりに設定を書く必要がある部分を省略できる感じです。
これは細かいものが多いので説明が難しいのですが、fzf.vimを使った上でfzf-preview.vimを使うと分かってくるかなと思います。

### Gitとの強力な連携

一般的なファジーファインダーの枠から外れる機能なのですが、[gina.vim](https://github.com/lambdalisue/gina.vim), [vim-fugitive](https://github.com/tpope/vim-fugitive)と連携することでgitの操作ができます。
Interactive rebaseはVim内で完結するのが難しいので実装していませんが、他の基本的な操作はほぼ網羅しているかと思います。
具体的にはstatus, add, reset, branch, checkout, merge, rebase, log, stash, reflog, commit, push, fetch, pullあたりの操作についてそれぞれのプレビューを見ながら操作できます。

以下が操作例です。

**git, status, add, commit, log**
@[youtube](l4khCeYgHT0)

**git branch, diff, checkout**
@[youtube](0cCd2NOmbaE)

### 苦手な部分

一方、fzf-preview.vimは細かい挙動のカスタマイズなどが比較的苦手です。
詳細な部分まで詰めて設定したい場合は他のファジーファインダーの方が向いているかもしれません。

## fzf-preview.vimをメイン使った作業フロー

### 機能追加

fzf-preview.vimを使って小規模な機能追加(TypeScriptのtypeにプロパティを増やす)を実際に行った簡単な例です。
![add_property_flow](https://user-images.githubusercontent.com/5423775/102732445-1740e000-437e-11eb-9495-b2f116cff005.png)
@[youtube](rKpbKKewUvM)

### リファクタリング

動的言語(Vim script)での関数名を変更した簡単な例です。置換に[vim-qfreplace](https://github.com/thinca/vim-qfreplace)というプラグインを併用しています。
![refactor_flow](https://user-images.githubusercontent.com/5423775/102732457-1f991b00-437e-11eb-9a32-fa0c2b17c30a.png)
@[youtube](bd07x_gKCcw)

## 開発のモチベーション

### 開発初期

ここからは少し内部の技術的な話に入っていきます。
fzf-preview.vimはファジーファインダーのインターフェイスを提供するプラグインですが、内部的には[fzf](https://github.com/junegunn/fzf)をファジーファインダーとして使っています。
fzfをVim上で使うための公式プラグインの[fzf.vim](https://github.com/junegunn/fzf.vim)で不便に感じる部分の解消と、様々な機能追加を主に行っています。

:::message
ややこしいのですが、ファジーファインダー自体の実装がfzf、Vim用の汎用コマンドを提供しているプラグインがfzf.vimです。
また、(fzf.vimでない)fzfのリポジトリにはVimからfzfを操作するためのユーティリティ関数的なものが置かれています。
:::

fzf.vimは優れたファジーファインダープラグインなのですが、自分がfzf-preview.vimを開発し始めた当時は機能が少なく使い辛い部分も多々ありました。(現在はfzf.vimも機能が拡張され、fzf-preview.vimが提供している機能の一部が重複するなどしています。)
また、fzfから提供されているユーティリティのインターフェイスがかなり使いづらく、ファイル名にアイコンを付けるなどの処理を行うだけでも黒魔術めいたコードを設定ファイルに記述する必要がありました。

自分はfzfの強力なプレビュー機能に惹かれ、最初はfzf.vimを使いながら自分が必要としている機能をvimrcに追加していたのですが、記述量が膨らんでいったので便利な機能をプラグインとして切り出したのがfzf-preview.vimの成り立ちです。
今は標準で対応していますが、NeovimのFloating Windowへの対応をしたかったなどの理由もあります。

### Vim scriptからTypeScriptへの移行

fzf-preview.vimの実装が肥大化するにつれ、fzfから提供しているインターフェイスを扱うのに限界が出てきました。
半ば無理に拡張していた部分の設計が破綻し、自分のVim scriptへの経験・知識不足もありますが言語機能的にも辛くなってきました。

そこで、設計の見直しとTypeScriptへ移行をし、スクラッチで書き直したのが現在のfzf-preview.vimです。
TypeScriptの強力な言語機能・優れた開発体験・エコシステムの強力さなどによってfzfをより抽象的に扱い、様々な機能を柔軟に提供可能になりました。
TypeScriptでの型定義を伴った開発体験はとてもよく、コード量が増えても破綻せずに開発を続けることができています。
npmの資産を使うことができるのも非常に大きかったです。(なんと状態管理にReduxを使うなどしています！)

:::message alert
ただ、そもそも入出力がUNIX思想に寄っているCLIを無理矢理Vimで高機能に使うのに結構無理があって、TypeScriptで実装したfzf-preview.vimも入出力を抽象的に扱う際に黒魔術っぽいことをしています。
:::

また、同様にTypeScriptで実装されたVimプラグインである[coc.nvim](https://github.com/neoclide/coc.nvim)と連携することも容易になり、LSPに関する機能の実装はcoc.nvimを経由して扱っています。

TypeScriptでのVimプラグイン開発については別途まとめたいと思っています。
情報が非常に少なく、最初にまともに動作させるまでがそれなりに大変でした。

プラグインの実装規模ですが、いつの間にか結構増えていって合計約9000行程度になっています。
行数の多いプラグインが優れているというわけではないですが、結構頑張ったなぁと思っています。

```
===============================================================================
 Language            Files        Lines         Code     Comments       Blanks
===============================================================================
 Python                  1           57           40            4           13
 TypeScript            187         7847         6795           69          983
 Vim script             30         1007          873            2          132
===============================================================================
 Total                 218         8911         7708           75         1128
===============================================================================
```

## 動作に必要なソフトウェア

ここまで色々と書いてきましたが、fzf-preview.vimは動作させるのにはVim以外に必要なものがいくつかあります。
というのもTypeScriptの導入も含め、高機能にするため外部のソフトウェアに色々な部分を押しつける前提で実装してきたからです。
Vimユーザはエディタの性質的にもミニマリストの方が比較的多く、ここで好みが結構分かれるようです。
少し面倒かと思いますが、興味のある方はこの節を読んで導入を試してみてください。

なお、Windowsでの動作は[PR](https://github.com/yuki-ycino/fzf-preview.vim/pull/184)で対応中です。

### TypeScriptランタイム

まず、Nodeと通信するのでTypeScriptをJS(CommonJS)にトランスパイルしておく必要があります。
その後に(Neo)vimからCommonJSを実行する(NodeとRPCを使って通信する)のには主に2つの既存実装があります。

- NeovimのRemote Plugin機能を使う
- [neoclide/coc.nvim](https://github.com/neoclide/coc.nvim)の拡張として実装する(独自でRPCを実装しているためVimでも利用できます。)

:::message
自前でRPCを実装してもいいのですが、ここでは割愛します。
:::

なので標準のVimのみでは動作させることができません。上記のどちらかを導入してください。
fzf-preview.vimはwebpackでそれぞれの環境用にクロスビルドを行い、どちらでも動作するようになっています。
それぞれのインストール方法については以下の通りです。

#### Remote Plugin

```
npm install -g neovim
```

##### vim-plug

```vim
Plug 'junegunn/fzf', { 'do': { -> fzf#install() } }
Plug 'yuki-ycino/fzf-preview.vim', { 'branch': 'release', 'do': ':UpdateRemotePlugins' }
```

##### dein.vim

```vim
call dein#add('junegunn/fzf', { 'build': './install --all', 'merged': 0 })
call dein#add('yuki-ycino/fzf-preview.vim', { 'rev': 'release' })
```

#### coc extensions

```vim
:CocInstall coc-fzf-preview
```

### 外部コマンドとプラグイン

存在するのが前提になっているコマンド及びVimプラグインがいくつかあるのでインストールする必要があります。
(おそらく設定次第で不要になりますが、これらを導入した方が楽です。)

#### Requirement

- Node <https://nodejs.org/>
- fzf <https://github.com/junegunn/fzf>
- Python3 <https://www.python.org/>
- ripgrep <https://github.com/BurntSushi/ripgrep>

#### Optional

- bat <https://github.com/sharkdp/bat>
- gina.vim <https://github.com/lambdalisue/gina.vim>

このあたりを導入しておくとかなり快適に使えるかと思います。
他の依存関係については[README](https://github.com/yuki-ycino/fzf-preview.vim/blob/master/README.md)や[doc](https://github.com/yuki-ycino/fzf-preview.vim/blob/master/doc/fzf_preview_vim.txt)に書いてあるのでそちらを参照してみてください。

## まとめ

- 中級VimmerがTypeScriptでVimのファジーファインダープラグインを開発してみた
  - TypeScriptの開発体験はフロントエンドに限らずとてもよい、採用できる環境であれば今後も積極的に検討していきたい
- ファジーファインダーは開発を効率化する上でとても有用なので是非導入するべき
- ファジーファインダー with プレビューは更に開発効率が上がる、軽快なプレビュー機能が搭載されているものがおすすめ
- 必要なものが多いけど、使える環境であれば拙作の[fzf-preview.vim](https://github.com/yuki-ycino/fzf-preview.vim)は個人的にとてもおすすめ

ファジーファインダーは何故かVimで非常に選択肢が多く、選択が難しいのですが導入する価値は十分にあると思います。
どのプラグインを導入するとしても、この記事に書いた内容が少しでも選定の参考になれば幸いです。
また、他のエディタやIDEを使っている方にも、この記事がファジーファインダーの選定・使い方などの参考になればなと思っています。

Vimとは関係ないですが、IntelliJ系のgrep機能などはファジーファインダーのような動作で非常に強力だと聞きました。
最近はJetBrainsの製品を触っていなかったのですが、自分が実装する機能の参考にするためにも、機会があればまた触ってみたいと思っています。

最後になりますが、自分は今後もfzf-preview.vimの開発を続けていきたいと考えているので、気になった方は試しに触って頂けると幸いです。

## おまけ

自分のfzf-preview.vimの設定です。
細かめにカスタマイズしているので冗長になっていますが、基本的にはあまり設定しなくても動作するようになっています。

:::details fzf-preview settings

```vim
let g:fzf_preview_git_files_command   = 'git ls-files --exclude-standard | while read line; do if [[ ! -L $line ]] && [[ -f $line ]]; then echo $line; fi; done'
let g:fzf_preview_grep_cmd            = 'rg --line-number --no-heading --color=never --sort=path'
let g:fzf_preview_mru_limit           = 500
let g:fzf_preview_use_dev_icons       = 1
let g:fzf_preview_default_fzf_options = {
\ '--reverse': v:true,
\ '--preview-window': 'wrap',
\ '--exact': v:true,
\ '--no-sort': v:true,
\ }
let $FZF_PREVIEW_PREVIEW_BAT_THEME  = 'gruvbox'

noremap <fzf-p> <Nop>
map     ;       <fzf-p>
noremap ;;      ;
noremap <dev>   <Nop>
map     m       <dev>

nnoremap <silent> <fzf-p>r     :<C-u>CocCommand fzf-preview.FromResources buffer project_mru<CR>
nnoremap <silent> <fzf-p>w     :<C-u>CocCommand fzf-preview.ProjectMrwFiles<CR>
nnoremap <silent> <fzf-p>a     :<C-u>CocCommand fzf-preview.FromResources project_mru git<CR>
nnoremap <silent> <fzf-p>g     :<C-u>CocCommand fzf-preview.GitActions<CR>
nnoremap <silent> <fzf-p>s     :<C-u>CocCommand fzf-preview.GitStatus<CR>
nnoremap <silent> <fzf-p>b     :<C-u>CocCommand fzf-preview.Buffers<CR>
nnoremap <silent> <fzf-p>B     :<C-u>CocCommand fzf-preview.AllBuffers<CR>
nnoremap <silent> <fzf-p><C-o> :<C-u>CocCommand fzf-preview.Jumps<CR>
nnoremap <silent> <fzf-p>/     :<C-u>CocCommand fzf-preview.Lines --resume --add-fzf-arg=--no-sort<CR>
nnoremap <silent> <fzf-p>*     :<C-u>CocCommand fzf-preview.Lines --add-fzf-arg=--no-sort --add-fzf-arg=--query="<C-r>=expand('<cword>')<CR>"<CR>
xnoremap <silent> <fzf-p>*     "sy:CocCommand fzf-preview.Lines --add-fzf-arg=--no-sort --add-fzf-arg=--query="<C-r>=substitute(@s, '\(^\\v\)\\|\\\(<\\|>\)', '', 'g')<CR>"<CR>
nnoremap <silent> <fzf-p>n     :<C-u>CocCommand fzf-preview.Lines --add-fzf-arg=--no-sort --add-fzf-arg=--query="<C-r>=substitute(@/, '\(^\\v\)\\|\\\(<\\|>\)', '', 'g')<CR>"<CR>
nnoremap <silent> <fzf-p>?     :<C-u>CocCommand fzf-preview.BufferLines --resume --add-fzf-arg=--no-sort<CR>
nnoremap          <fzf-p>f     :<C-u>CocCommand fzf-preview.ProjectGrep<Space>
xnoremap          <fzf-p>f     "sy:CocCommand fzf-preview.ProjectGrep<Space>-F<Space>"<C-r>=substitute(substitute(@s, '\n', '', 'g'), '/', '\\/', 'g')<CR>"
nnoremap <silent> <fzf-p>q     :<C-u>CocCommand fzf-preview.QuickFix<CR>
nnoremap <silent> <fzf-p>l     :<C-u>CocCommand fzf-preview.LocationList<CR>
nnoremap <silent> <fzf-p>:     :<C-u>CocCommand fzf-preview.CommandPalette<CR>
nnoremap <silent> <fzf-p>p     :<C-u>CocCommand fzf-preview.Yankround<CR>
nnoremap <silent> <fzf-p>m     :<C-u>CocCommand fzf-preview.Bookmarks --resume<CR>
nnoremap <silent> <fzf-p><C-]> :<C-u>CocCommand fzf-preview.VistaCtags --add-fzf-arg=--query="<C-r>=expand('<cword>')<CR>"<CR>
nnoremap <silent> <fzf-p>o     :<C-u>CocCommand fzf-preview.VistaBufferCtags<CR>

nnoremap <silent> <dev>q  :<C-u>CocCommand fzf-preview.CocCurrentDiagnostics<CR>
nnoremap <silent> <dev>Q  :<C-u>CocCommand fzf-preview.CocDiagnostics<CR>
nnoremap <silent> <dev>rf :<C-u>CocCommand fzf-preview.CocReferences<CR>
nnoremap <silent> <dev>t  :<C-u>CocCommand fzf-preview.CocTypeDefinitions<CR>

AutoCmd User fzf_preview#initialized call s:fzf_preview_settings()

function! s:buffers_delete_from_lines(lines) abort
  for line in a:lines
    let matches = matchlist(line, '\[\(\d\+\)\]')
    if len(matches) >= 1
      execute 'Bdelete! ' . matches[1]
    endif
  endfor
endfunction

function! s:fzf_preview_settings() abort
  let g:fzf_preview_grep_preview_cmd = 'COLORTERM=truecolor ' . g:fzf_preview_grep_preview_cmd
  let g:fzf_preview_command = 'COLORTERM=truecolor ' . g:fzf_preview_command

  let g:fzf_preview_custom_processes['open-file'] = fzf_preview#remote#process#get_default_processes('open-file', 'coc')
  let g:fzf_preview_custom_processes['open-file']['ctrl-s'] = g:fzf_preview_custom_processes['open-file']['ctrl-x']
  call remove(g:fzf_preview_custom_processes['open-file'], 'ctrl-x')

  let g:fzf_preview_custom_processes['open-buffer'] = fzf_preview#remote#process#get_default_processes('open-buffer', 'coc')
  let g:fzf_preview_custom_processes['open-buffer']['ctrl-s'] = g:fzf_preview_custom_processes['open-buffer']['ctrl-x']
  call remove(g:fzf_preview_custom_processes['open-buffer'], 'ctrl-q')
  let g:fzf_preview_custom_processes['open-buffer']['ctrl-x'] = get(function('s:buffers_delete_from_lines'), 'name')

  let g:fzf_preview_custom_processes['open-bufnr'] = fzf_preview#remote#process#get_default_processes('open-bufnr', 'coc')
  let g:fzf_preview_custom_processes['open-bufnr']['ctrl-s'] = g:fzf_preview_custom_processes['open-bufnr']['ctrl-x']
  call remove(g:fzf_preview_custom_processes['open-bufnr'], 'ctrl-q')
  let g:fzf_preview_custom_processes['open-bufnr']['ctrl-x'] = get(function('s:buffers_delete_from_lines'), 'name')

  let g:fzf_preview_custom_processes['git-status'] = fzf_preview#remote#process#get_default_processes('git-status', 'coc')
  let g:fzf_preview_custom_processes['git-status']['ctrl-s'] = g:fzf_preview_custom_processes['git-status']['ctrl-x']
  call remove(g:fzf_preview_custom_processes['git-status'], 'ctrl-x')
endfunction
```

:::
