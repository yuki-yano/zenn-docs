---
title: "無人島に持っていく(Neo)vimプラグイン10選 (TS開発環境編)"
emoji: "⚙️"
type: "tech"
topics: ["Neovim", "Vim"]
published: true
---

## 概要

自分は普段Neovimを使って(主にTypeScriptでの)開発を行っています。
NeovimでのTypeScript開発は環境を整えればVSCodeと遜色ない開発体験を得ることができると思っています。
また、Vimの操作に慣れれば一部の編集機能についてはVimの方が優れていると感じています。

この記事では自分の開発に欠かせないプラグイン10個を紹介しようと思います。

## プラグイン10選

- VimをIDE化するプラグイン [coc.nvim](https://github.com/neoclide/coc.nvim)
- 様々な操作の起点に使うFuzzy Finder [fzf-preview.vim](https://github.com/yuki-yano/fzf-preview.vim)
- 高機能なファイラ [fern.vim](https://github.com/lambdalisue/fern.vim)
- Vim上でGitを操作する [gina.vim](https://github.com/lambdalisue/gina.vim)
- テキストを囲う操作を可能にする [vim-sandwich](https://github.com/machakann/vim-sandwich)
- 文字の入力を自動で展開する [lexima.vim](https://github.com/cohama/lexima.vim)
- Snippetプラグイン [UltiSnips](https://github.com/SirVer/ultisnips)
- 一括置換を行う [vim-qfreplace](https://github.com/thinca/vim-qfreplace)
- 汎用タスクランナー [vim-quickrun](https://github.com/thinca/vim-quickrun)
- 構文を解析し、ハイライトなどが可能になる [nvim-treesitter.vim](https://github.com/nvim-treesitter/nvim-treesitter)

自分は現在124個のプラグインを入れているのですが、その中から敢えて10個を抜き出すとしたら上記になります。

## プラグインの解説

### [coc.nvim](https://github.com/neoclide/coc.nvim)

端的に言うとVimをVSCode化するプラグインです。
主要機能を抜粋するとCompletion, Diagnostics, Code navigation, Code actionなどが使えるようになります。

coc extensionsと呼ばれる拡張をインストールすることで、様々な言語への対応などが行えます。
LSPクライアントでもあるため、VSCodeでの開発体験の大半を再現可能です。

例えばTypeScriptの開発をする場合は、 `:CocInstall coc-tsserver` を実行することでVSCodeに近いTypeScript開発環境が整います。

また、coc.nvim本体がTypeScriptで実装されており、Nodeを使ってVimと連携しています。
そのためcoc extensionsは主にTypeScriptで開発されており、VSCodeのプラグインをforkして開発されたものも多数あります。
(GUI操作については再現できないものも多いですが)VSCodeプラグインの機能をVimで使いたければ、forkして一部を書き換えればVim上で扱うことができるようになります。

**Completion**
![coc_completion](https://user-images.githubusercontent.com/5423775/130346043-1fb9427c-d85a-4938-a717-195f9728d97f.png)

**Show document**
![coc_show_document](https://user-images.githubusercontent.com/5423775/130345979-a281d4f1-3a86-41a8-b8b3-5bf0b273b188.png)

**Diagnostics**
![coc_diagnostics](https://user-images.githubusercontent.com/5423775/130346008-146355ce-38af-4127-96b9-85cb7c3ed185.png)

**Code action**
![coc_code_action](https://user-images.githubusercontent.com/5423775/130346013-39e7adbb-9957-4f01-be0b-524f40803122.png)

### [fzf-preview.vim](https://github.com/yuki-yano/fzf-preview.vim)

自作のFuzzy Finderプラグインです。様々な操作の起点として使っています。
詳細については以前書いた記事を読んでみてください。
https://zenn.dev/yano/articles/vim_with_fzf_preview_is_best_experience

TypeScriptで実装しており、coc extensionsとしてインストールすることでcoc.nvimとの連携も可能です。

**Files**
![fzf_preview_files](https://user-images.githubusercontent.com/5423775/130346369-936590ca-88c8-41c6-adf7-f10bdb5817d4.png)

**Grep**
![fzf_preview_grep](https://user-images.githubusercontent.com/5423775/130346385-2b1304e4-687f-4088-80ad-f11462e7de72.png)

**References**
![fzf_preview_references](https://user-images.githubusercontent.com/5423775/130346388-3766cd46-060b-4d60-9bc8-3a3638395a27.png)

### [fern.vim](https://github.com/lambdalisue/fern.vim)

高機能なファイラです。全画面で使うこともできますが、主に左側に固定幅表示するIDEスタイルで使用しています。
Vimのファイラについては次の記事が参考になります。
https://zenn.dev/lambdalisue/articles/3deb92360546d526381f

基本的なファイルの操作はもちろん、Vimのテキスト編集を使って複数ファイルの一括リネームなども可能なのでよく使っています。

また、fern自体がプラガブルになっており、pluginを作成して拡張可能です。
[Plugins · lambdalisue/fern.vim Wiki](https://github.com/lambdalisue/fern.vim/wiki/Plugins)

プラグインとしてはgitと連携をするものや、自分が作ったものとしてFloating Windowを使ってカーソル上のファイルのpreviewを行うものがあります。
[yuki-yano/fern-preview.vim: Add a file preview window to fern.vim.](https://github.com/yuki-yano/fern-preview.vim)

**fern**
![fern](https://user-images.githubusercontent.com/5423775/130427390-83881c29-2a6f-478e-869f-4079f23d8d2c.png)

### [gina.vim](https://github.com/lambdalisue/gina.vim)

Vim上からGitを操作するためのプラグインです。
一般的なGitの操作はもちろん、キラー機能として `patch` サブコマンドを使ってのstage, unstageの操作、 `chaperon` サブコマンドを使ってのIDEライクなconflictの解消などがあります。

上記はそれぞれvimdiffを使って編集するようになっており、Vimの操作の恩恵をそのまま受けながら文字単位でGitの操作が可能です。

また、前述のfzf-preview.vimは内部的にginaと連携しており、Fuzzy FinderのUIからそのままGitを操作できます。

**Patch**
![gina_patch](https://user-images.githubusercontent.com/5423775/130430848-d72b4910-0ba1-40a9-9192-2bed11d0d885.png)

**Conflict resolution**
![gina_chaperon](https://user-images.githubusercontent.com/5423775/130430916-e0a0b916-315a-4e8a-bd48-290a8336e4ef.png)

**status, add, commit, log**
@[youtube](l4khCeYgHT0)

### [vim-sandwich](https://github.com/machakann/vim-sandwich)

Vimのoperatorとtextobjを使ってテキストを囲う操作を提供するプラグインです。(選択範囲の編集も可能です)

分かりにくいと思うので実例を挙げると、
`foo` という単語の上で `saiw'` と入力すると `'foo'` に編集されます。

これを分割すると

- `sa` (sandwich add)
- `iw` (in word)
- `'` ( `'` で囲う)

という操作になっています。

add以外にもdelete, replaceが可能で、replaceで例を挙げると
`'foo'` 上で `` sr'` `` を入力すると `` `foo` `` に編集されます。

どういう編集が行われるをユーザ定義可能で、例えば自分はTypeScriptでは
`foo` -> `saiw$` -> `${foo}` と編集されるように定義しています。

また、関数で囲う操作、ユーザ定義のtextobjを使った編集も可能です。
自分は [vim-textobj-functioncall](https://github.com/machakann/vim-textobj-functioncall) を使って関数・ジェネリクスなどでの編集も行っています。
実例を挙げると

- `foo` -> `saiwfconsole.log<Enter>` -> `console.log(foo)`
- `foo(bar)` -> `srffhoge<Enter>` -> `hoge(bar)`
- `string` -> `saiwgPromise<Enter>` -> `Promise<string>`

などになります。

### [lexima.vim](https://github.com/cohama/lexima.vim)

文字の入力を自動で展開するプラグインです。
一般的によく使われるものとしては `(` を `()` に展開するなどがあります。

この手のプラグインは多数あるのですが、その中でもlexima.vimは拡張性が高いので使い続けています。
正規表現を使って設定可能で、自分はTypeScriptでArrow Functionの展開に使う設定を入れています。

```vim
let s:rules += [
\ { 'filetype': ['typescript', 'typescriptreact'], 'char': '>', 'at': '\s([a-zA-Z, ]*\%#)',            'input': '<Left><C-o>f)<Right>a=> {}<Esc>',                 },
\ { 'filetype': ['typescript', 'typescriptreact'], 'char': '>', 'at': '\s([a-zA-Z]\+\%#)',             'input': '<Right> => {}<Left>',              'priority': 10 },
\ { 'filetype': ['typescript', 'typescriptreact'], 'char': '>', 'at': '[a-z]((.*\%#.*))',              'input': '<Left><C-o>f)a => {}<Esc>',                       },
\ { 'filetype': ['typescript', 'typescriptreact'], 'char': '>', 'at': '[a-z]([a-zA-Z]\+\%#)',          'input': ' => {}<Left>',                                    },
\ { 'filetype': ['typescript', 'typescriptreact'], 'char': '>', 'at': '(.*[a-zA-Z]\+<[a-zA-Z]\+>\%#)', 'input': '<Left><C-o>f)<Right>a=> {}<Left>',                },
\ ]
```

上記の設定で

- `(<cursor>)` -> `>` -> `() => {<cursor>}`
- `(foo<cursor>)` -> `>` -> `(foo) => {<cursor>}`
- `hoge((foo, bar<cursor>))` -> `>` -> `hoge((foo, bar) => {<cursor>})`

などの展開が行われます。

### [UltiSnips](https://github.com/SirVer/ultisnips)

Snippet機能を提供するプラグインです。
スニペットプラグインの比較については次の記事が参考になります。
https://zenn.dev/shougo/articles/snippet-plugins-2020

UltiSnipsはVim script, Pythonを使ってSnippetの動的な展開が可能です。
入力したテキストを引数に取って関数を実行して展開する、ファイル名を元にテキストを展開する、などが可能です。

以下がReactのComponentを展開する設定と実際の挙動です。
Propsの型から各コンポーネントの引数のDestructuringと、ファイル名からコンポーネント名を動的に生成しています。

```
global !p
def parse_props(args):
  prop_names = []
  for line in args.split("\n"):
    arr = line.split(":")
    if len(arr) == 0:
      continue
    prop_names.append(arr[0].lstrip())

  return ', '.join(prop_names)
endglobal

snippet fct "FC file template"
import styled from "@emotion/styled"
import type { FC } from "react"

type PassedProps = {
  ${2}
}

type ContainerProps = {
  ${3}
}

type Props = PassedProps & ContainerProps

const Component: FC<Props> = ({ `!p snip.rv = parse_props(t[2] + "\n" + t[3])` }) => {
  return <></>
}

const StyledComponent = styled(Component)``

const ContainerComponent: FC<PassedProps> = (props) => {
  const { `!p snip.rv = parse_props(t[2])` } = props
  return <StyledComponent {...props} />
}

export const `!p snip.rv = ''.join(list(map(lambda v: v.capitalize(), snip.basename.split('-'))))` = ContainerComponent
endsnippet
```

@[youtube](E51L13LFWbc)

### [vim-qfreplace](https://github.com/thinca/vim-qfreplace)

QuickFixを用いてテキストの一括置換を可能にするプラグインです。
Grep結果の一括置換に使うことが多いかと思います。

前述のfzf-preview.vimがgrep結果のQuickFixへの出力機能を実装しており、自分は主にそれを起点に実行することが多いです。
[fzf-preview.vimの紹介記事](https://zenn.dev/yano/articles/vim_with_fzf_preview_is_best_experience)で実例としてあげたのですが、次のような使い方をしています。

@[youtube](bd07x_gKCcw)

### [vim-quickrun](https://github.com/thinca/vim-quickrun)

様々な使い方や設定が可能な高機能タスクランナーです。
編集中のファイルをそのままランタイムに渡して実行結果を出力したり、プロジェクト全体にlintを走らせて結果をQuickFixに出力するなど、設定次第で色々な使い方が可能です。

```
console.log('test')
```

というファイルで `:QuickRun deno` を実行した例が以下です。

![quickrun_deno](https://user-images.githubusercontent.com/5423775/130439776-ba3c80c3-3bc4-4cfe-b6d7-16fd490bee82.png)

プロジェクトにtscコマンドを実行してエラー一覧を出力する例が以下です。

@[youtube](j7k1kYxOjwg)

### [nvim-treesitter.vim](https://github.com/nvim-treesitter/nvim-treesitter)

構文解析をし、Syntax highlightなどを提供してくれるNeovim用プラグインです。
Vimは Syntax highlight を正規表現で実装しているので、それと比較して高精度かつ豊富なハイライトが行われます。

Syntax Highlight以外にも様々なことが可能になっています。
以下の記事で詳細に説明されているので、是非読んでみてください。
[Neovim v0.5リリース記念 v0.5の新機能を紹介します【後編】 | MoT Lab (Mobility Technologies Engineering Blog)](https://lab.mo-t.com/blog/neovim-v05-introduction-new-features-part-2)

## おわりに

今回は普段使っている中から無理矢理10個のプラグインを選んで紹介したのですが、1つ1つについて記事が書けるくらいのものばかりなので大変でした。
どれも素晴らしいプラグインなので、(Neo)vimを使う際には是非試していただけると幸いです。

分からない点や質問などがあれば、[Twitter](https://twitter.com/yuki_ycino)や[vim-jp](https://blog.tomoya.dev/posts/vim-jp-is-a-paradise-for-engineers/)などで声をかけていただければ返答できるかと思います。
