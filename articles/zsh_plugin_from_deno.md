---
title: "Deno + TypeScriptでzshプラグインを実装して最高になった"
emoji: "🔍"
type: "tech"
topics: ["Deno", "TypeScript", "zsh", "fzf"]
published: true
---

## 概要

今回作ったプラグインは [yuki-yano/zeno.zsh](https://github.com/yuki-yano/zeno.zsh) です。
昔 [yuki-yano/fzf-preview.zsh](https://github.com/yuki-yano/fzf-preview.zsh) というものを作っていたのですが、シェルスクリプトでの実装と保守が厳しすぎて放置しており、それをDenoで再実装したものになります。

zshで頻繁に使う機能の拡張を実装しており、zshの操作体験を改善できると思います。
まずは[デモ](#デモ)を見てみてください。

具体的なプラグインの機能としては以下があります。

- abbrevを用いたsnippetの展開
- ファジーファインダー(fzf)を用いた補完
- fzfを用いたsnippetの挿入
- その他いくつかの便利utility

Denoを採用した理由はいくつかあるのですが、それについては[後述](#denoの採用について)します。

## デモ

**Abbrev snippet**
@[youtube](0Yp0czGluIc)

**Fuzzy completion**
@[youtube](Ovkmjo0cweo)

## zeno.zshの主要機能

### Abbrev snippet

zshでよく使われるaliasに近い機能として実装しているものです。
snippetを定義し、同時に設定したkeywordからSpace及びEnterでsnippetを展開します。

例えば

```yaml
snippets:
  - name: git status
    keyword: gs
    snippet: git status --short --branch
```

と設定していた場合、 `$ gs<Space>` で `$ git status --short --branch` と展開されます。
また、 `$ gs<Enter>` でも展開されたコマンドが実行されるようになっています。

そして、これは行頭でなくとも展開されます。

```yaml
snippets:
  - name: "null"
    keyword: "null"
    snippet: ">/dev/null 2>&1"
```

という設定の場合は、 `$ ls null<Space>` で `$ ls >/dev/null 2>&1` が展開されます。
これについては邪魔になる場合もあるかと思うので、今後オプションで有効か切り替え可能にするかもしれません。

snippet展開のもう1つの機能として、placeholderへの対応というものがあります。
placeholderのformatは `{{任意のテキスト}}` です。

```yaml
snippets:
  - name: git commit message
    keyword: gcim
    snippet: git commit -m '{{commit_message}}'
```

上記の設定であれば、 `$ gcim<Space>` で `$ git commit -m '<Cursor>'` と、placeholderの位置にカーソルが移動した上で展開されます。
また、placeholderを複数挿入して補完と連携が可能です。これについては後述します。

### Insert snippet

snippetはfzfを用いて挿入可能です。
ZLEを実行すると以下のような画面が表示され、選択したsnippetがカーソル位置に挿入されます。

![insert-snippet](https://user-images.githubusercontent.com/5423775/119255745-4f33a400-bbf8-11eb-9b6f-03d551dc9d54.png)

### Fuzzy completion

設定された正規表現にマッチした状態で補完すると、fzfを用いた補完が起動します。
こういった設定は一般的にはfzfをパイプしたコマンドを定義するなどが多いかと思うのですが、zeno.zshでは通常の補完と区別せず実行可能となっています。
また、あくまで補完するだけなので直接fzfからコマンドを実行した場合と違いhistoryにコマンドを残すことが可能です。

zeno.zshではbuiltinでgitの補完を実装しているのでgit補完は設定不要です。
補完できるコマンドの詳細についてはREADMEを参照してください。

例を挙げると、以下のようなUIの補完が起動します。

![git-add](https://user-images.githubusercontent.com/5423775/119255923-26f87500-bbf9-11eb-9ef7-fbd0202ad532.png)
![git-checkout](https://user-images.githubusercontent.com/5423775/119255926-28c23880-bbf9-11eb-82b7-89feafae2bb6.png)

設定は以下のフォーマットで行います。これはkillコマンドの補完にpsの出力を使う例です。

```yaml
completions:
  - name: kill
    patterns: 
      - "^kill( -9)? $"
    sourceCommand: "ps -ef | sed 1d"
    options:
      --multi: true
      --prompt: "'Kill Process> '"
    callback: "awk '{print $2}'"
```

また、placeholderとの連携についてですが、補完を確定した際に次のplaceholderへとカーソルが移動するようになっています。
なので、例えば `git diff {{branch1}} {{branch2}}` というsnippetを定義していた場合、1つめのブランチの補完後に2つめのブランチを補完できます。

### fzf-tmux integration

Optionalな機能ですが、補完にfzf-tmuxを使う事ができます。
最近実装されたpopupを用いると以下のようなUIで補完が可能です。

![fzf-tmux-completion](https://user-images.githubusercontent.com/5423775/119256227-a6d30f00-bbfa-11eb-9716-fee756049868.png)

## Denoの採用について

### シェルスクリプトが厳しい

シェルスクリプトは複雑な処理を書くには向いていない言語だと思っています。
配列やハッシュの扱いも非常に弱く、記法も直感的でないものが多いです。
また、ぐぐったときに情報が少なめなのも厳しいです。

以前作ったプラグインはどこかに手を入れるたびにどこかが壊れるような実装になってしまっており、機能追加とバグ修正に非常にコストがかかっていました。
他にもYAMLを読み込むこともできず、設定ファイルを独自フォーマットで書かざるを得なかった、などもあります。

なので、まともなプラグインを提供するためにはある程度モダンな言語を採用する必要がありました。

### 言語としてDenoを採用した理由

実は最初はRustで実装しようとしていたのですが、しばらく書いてから学習コストと堅牢に作るための実装コストで諦めたという経緯があります。
その後Denoへ移行したのですが、その理由は以下となります。

- 自分が普段からNode, フロントエンドでTypeScriptを使うことが多く慣れている
- Deno + TypeScriptの開発体験がとても良い
- TypeScriptを使うことで固いところは固く、柔軟に書きたい部分は柔軟に実装できる
- 実装規模が大きくなってきた場合でもスケールしやすい
- Nodeと違って開発・実行環境の準備が容易
- LLの中では実行が高速

実際、自分は最近小物を実装する際はDeno + TypeScriptを採用することが多く、開発体験もとても良いです。

## プラグインの仕組み

zshにはZLEという仕組みがあり、基本的に全ての操作はZLEの実行という形で動作しています。
例えば `<C-b>` で1文字戻る操作は `backword-char` 、Enterでのコマンド実行は `accept-line` です。

zeno.zshではSpace及びEnterに独自のZLEを割り振り、その中でDenoを実行しています。
DenoにzshのBUFFERを渡し、条件に一致した場合はabbrevやfuzzy completionの起動、一致しない場合は通常の処理にFallbackという流れです。
なのでSpaceなどを入力するたびにDenoのプロセスを起動しており、かなり富豪的な実装となっています。

実際の挙動なのですが、入力として `gs` を渡してのabbrevのデバッグ出力が以下です。
zshのBUFFERをzenoコマンドにpipeして渡し、標準出力をZLE内で読んで1行目がsuccessだった場合はBUFFERとCURSORを更新します。
2行目が展開結果、3行目がcursorのpositionになっています。

```sh
$ echo "gs\n" | zeno --mode=auto-snippet
success
git status --short --branch
28
```

Completionのデバッグ出力が以下です。
2行目がfzfにpipeするコマンド、3行目がfzfに渡すオプション、4行目がfzfの出力を受け取るcallbackコマンドです。

```sh
$ echo "git add " | zeno --mode=completion
success
git -c color.status=always status --short
--bind="ctrl-d:preview-half-page-down,ctrl-u:preview-half-page-up,?:toggle-preview" --expect="alt-enter" --preview="[[ \$(git diff -- {-1}) ]] && git diff --color=always -- {-1} || [[ \$(git diff --cached -- {-1} ) ]] && git diff --cached --color=always -- {-1} || bat --color=always {-1} 2>/dev/null || tree {-1} 2>/dev/null" --ansi --height='80%' --multi --prompt='Git Add Files> '
perl -nle '@arr=split(/ /,$_); print @arr[$#arr]'
```

このような流れで処理行い、snippetの展開や補完を実現しています。

### 最後に

Shell Scriptでは苦労して相当時間がかかった実装をDenoへ移植し短期間で実装できました。
プロトタイプには1日弱、主要機能がほぼ形になるまでが2-3日だったかと記憶しています。

Denoはまだ安定していないと思いますが、プライベートでちょっとしたものを動かすのには選択肢に入ってくる優秀なランタイムだと思っています。
今後も引き続きDeno, TypeScriptをキャッチアップしつつ開発を継続していきたいと思っています。

また、もし興味を持たれた方がいれば [zeno.zsh](https://github.com/yuki-yano/zeno.zsh) を試していただけると幸いです。
