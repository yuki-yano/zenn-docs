---
title: "自分がNeovimでフロントエンド開発を行っている理由とVSCode"
emoji: "🛠️"
type: "tech"
topics: ["Vim", "Neovim", "VSCode"]
published: true
---

:::message
この記事は[Vim Advent Calendar 2021-3](https://qiita.com/advent-calendar/2021/vim)の25日目の記事です。
:::

フロントエンドエンジニアをしているYano ([@yuki_ycino](https://twitter.com/yuki_ycino)) といいます。
先日もVimについてAdvent Calendarで記事を書かせて貰ったのですが、この記事では自分がNeovimを使っている理由や、NeovimとVSCodeを比べて思っていることなどを書いていきます。

## 前置き

先日[Neovimでのフロントエンド開発環境 2021](https://zenn.dev/yano/articles/vim_frontend_development_2021)という記事を書いたのですが、正直minimumなTypeScript開発環境は(VSCodeユーザから見ると)VSCodeの下位互換になっており、Vimを使うモチベーションが相当ないと試してみる動機にならないような内容だったかと思います。
この記事ではその障壁の解消の手伝いをできたらいいかなと思っています。
https://zenn.dev/yano/articles/vim_frontend_development_2021

また、[vim-jp](https://vim-jp.org/docs/chat.html)で話していて何故自分がVSCodeではなくてNeovim + [coc.nvim](https://github.com/neoclide/coc.nvim) + 各種Vim Pluginに携行しているかを気付かされたので、この記事で書いていきたいと思います。
話していた内容は以下の画像です。

![vim-jp-comment](https://user-images.githubusercontent.com/5423775/146823390-4d187df2-f6ea-4f0d-8f9d-27deacbf4cbb.png)

### 自分がNeovimを愛用している理由

Vimは人によって全然使い方が変わるエディタなのですが、自分はVimをVSCodeに近いスタイルでIDEライクに使用しています。

そして自分がVSCodeではなくNeovimを使っている理由は、coc.nvimによるLSPのサポート・Vimのtextobj, operator, macroなどの高度なテキスト編集機能・Vim script, Luaを用いた豊富なプラグインの導入、の全てを同時に実現できるからです。
更に、自分の場合は導入している各種プラグインのいくつかをかなり強力にカスタマイズしており(約5000行の設定ファイルがあり…)、自分用に最適化されロックインされてしまって抜け出せなくなってしまっているというのもあります。
また、coc.nvimはTypeScriptで実装されているVimのプラグインで、VSCodeの拡張をforkしたものを使うことができます。

Vimのテキスト編集及びVim Pluginの資産に加えてVSCodeの資産を使えると考えると、Neovim + coc.nvim + (Neo)vim Pluginは最強のテキストエディタの一角と言ってもいいのではないでしょうか。
その分設定は大変ですが、環境を整える価値はあるかと思っています。

## Neovim + coc.nvim + (Neo)vim Plugin

前述した自分がNeovimを使っている重要な要素及びその理由は以下になります。

### Neovim

Vimは標準で(主にtextobj, operator, dot repeat, macroなどを用いた)強力なテキスト編集機能を持っています。
操作に慣れるまでが大変ではあるのですが、これらのテキスト編集機能は慣れると他のエディタへの移行が難しくなるほど強力です。

### coc.nvim

Neovimに[coc.nvim](https://github.com/neoclide/coc.nvim)を導入することでVSCodeライクな開発が可能になり、VSCodeの拡張をforkしたものを使うこともできるようになります。
coc.nvimを導入した時点で(ファイラやファジーファインダーなどについては別途導入しないと不足していますが)LSPを用いた開発についてはVSCodeと比べて遜色ない機能を使うことができます。

### Vim Plugin

Vim pluginには大量のプラグイン資産があり、VSCodeの拡張では代替できないプラグインも大量に存在します。
近年ではNeovim専用のLua製プラグインも数多く登場してきており、UIのリッチさ及び速度の面で非常に優秀です。

## VSCodeとNeovimの思想の違い

### VSCode

言わずもがなVSCodeは非常に少ない設定でIDEライクに動作する強力なエディタです。
例えばTypeScriptについては設定不要で開発環境が整備され、他の言語などについてもMarketplaceから拡張を簡単に検索してインストールできます。

しかし、個人的にVSCodeの設定をしていて欠点だと思っている部分もあり、設定ファイルがJSONでしか記述できないという点です。
VSCodeの本体及び拡張機能の設定についてはJSONファイルで記述します。JSONはプログラマブルではないので、定義されていない設定をそもそも書くことができません。
ただ、多くのユーザは開発環境の整備にコストをかけたくないはずで、プログラマブルに設定が記述できてしまった場合のサポートコストなどを考慮するとVSCodeの思想は理解できます。

### Neovim

VSCodeは基本的に設定ファイルがJSONでしか記述できないのに対して、NeovimはVim script, Luaでプログラマブルに設定可能です。
(Vim scriptは言語仕様が厳しめの言語ではあるのですが)VSCodeのJSONを使った設定と比べると圧倒的に柔軟性が高いです。
また、VSCodeではbuilt-inのUIが大半ですがVimはそのあたりについてかなり自由に拡張でき、例えばファイラをプラグインで実装するなど根本的なUIの実装も行う事ができます。
感覚的にはVSCodeの設定がJavaScript/TypeScriptで動的に設定でき、UIなどについても独自に実装するためのAPIが提供されているイメージです。

Vim scriptを使って強力かつ柔軟な設定がすぐに書けるのがVimを使い続けている大きな理由の1つです。

## Neo(vim)の強さについて

### VSCodeに劣らないモダンなUI

Vimといえば端末を使い、ほとんど装飾のない黒い画面でファイルを編集しているというイメージの強い方が多いと思います。
しかし、近年の(Neo)vimはVSCodeに遜色ない(は流石に言いすぎですが)、非常にリッチなUIと機能を使って開発できるようになっています。
まずは次の画像を見てください。自分が普段TypeScript開発に使っているNeovimのスクリーンショットになります。

![neovim_dev](https://user-images.githubusercontent.com/5423775/147381080-ee7ca7f5-9205-4520-a815-db80203c0d42.png)

どうでしょうか、VSCodeと同様にLSPを使いCompletionやDiagnosticsの表示、更に補完候補の情報のHover表示なども行えています。
自分は常に表示しているわけではないのですがファイラもVSCodeライクなものを使っており、機能で言えばVSCodeのファイラより強力なファイル操作も可能です。

端末で使うアプリケーションという制約上、インラインでブラウザを使ったり画像の表示などは行えませんが、そういった部分以外についてはVSCodeと遜色ない開発体験が実現できます。

### 自由度の高さ

上記に加え、UIや使い勝手などが気に食わなければ自分で実装するという選択肢があります。
例えばこの画像はカーソル下の変数についてLSPでReferencesを引いている例となっています。
これについては既存のプラグインのUIが使いづらいものだったため、自分がプラグインとして実装しました。

![fzf_preview_references](https://user-images.githubusercontent.com/5423775/147381451-712cdbfc-5b1b-47b7-9a09-e32423ad3944.png)

詳細は以下の記事を参考にしてください。
https://zenn.dev/yano/articles/vim_with_fzf_preview_is_best_experience

このプラグイン以外にも同様の機能を実現しているものは複数ありますが、それらから選ぶ、または自作するという選択肢を取ることが可能です。
今回はUIについての例ですが、これはUIに限らずVimの機能全てにおいて言えることです。

また、後述しますがVSCodeはUIの大幅な拡張などは難しく、更にプラグインを作成するコストがVimと比べると高いです。(多分そのはず？勘違いしてたらすみません)

### 豊富なプラグイン

当然なのですがVimには長い歴史があり、ユーザにはハッカー気質な人が多いです。
そのためプラグインという資産が大量に存在し、現在も数多くのプラグインが開発され続けています。
また、日本のVimmerは熱狂的なユーザが非常に多く、[vim-jp Slack](https://vim-jp.org/docs/chat.html)では毎日のようにプラグイン(開発)についての話で盛り上がっています。

以下の2つの記事ではプラグインを用いたフロントエンド開発環境構築についてと、自分が常用しているプラグインについて少し解説しています。
https://zenn.dev/yano/articles/vim_frontend_development_2021
https://zenn.dev/yano/articles/vim_plugin_top_10

### 強力なテキスト編集機能

Vimにはtextobj, operator, dot repeat, macroなどといった非常に強力なテキスト編集機能が存在します。
(見ても何をしているか分からない可能性が高そうなのですが)とりあえず自分がよく行う編集についてこの動画を見てみてください。
あとでどういうことをしているか説明します。
@[youtube](cKWll2BN0dw)

#### textobjとoperator

textobjはテキストを特定の単位で選択する機能、operatorはtextobjの単位に対してなんらかの操作をするための機能です。
分かりづらいと思うので一例を説明すると、 `"foo bar"` というテキストがあり、カーソルが `foo` の上にあるとします。

このときに、 `diw` を押下すると `foo` が削除されます。
この操作は `d` operatorと `iw` textobjに分解でき、結果として意味としては `delete in word` となり、単語の削除が行われます。
同様に `ciw` で `change in word` で単語を削除してINSERTモードに入る、 `yiw` で `foo` をyank(registerに入れる)といった操作が可能です。
`iw` 以外にも様々な操作ができ、 この場合は `di"` がダブルクォートの内側の削除、 `da"` がダブルクォートも含めての削除として実行されます。

#### dot repeat

Vimでは最後に行ったテキスト編集操作を `.` を押下することで再実行できます。
さっきの例でいうと、 `"foo bar"` の上で `ciwhoge<Esc>` と実行すると `"hoge bar"` となりますが、その後に `bar` の上で `.` を押下すると `"hoge hoge"` となります。

正直これだけだとメリットが伝わりにくいと思うのですが、これらを組み合わせることで動画のようなテキスト編集が可能になります。

#### 動画について

主に[vim-sandwich](https://github.com/machakann/vim-sandwich)というプラグインを使ってテキストの操作している例になります。
`saiwg` (sandwich add in word generics)で `number`をPromise genericsで囲い、dot repeatを使って `.` を押すことで `string` も同様にPromiseで囲っています。
また、 `saagg` (sandwich add around generics to generics)でGenerics全体をGenericsで囲う操作をしています。
あとは `sdg` でGenericsを削除する・ `` sr"` `` で囲いに使っている `"` を `` ` `` に変える ・ `saiw$` で単語を `${}` で囲む・ `srff` (sandwich replace function to function) で関数名を変えるなどです。
当然、 `c` を使い `cig` を押下することでGenericsの内の型名の変更なども実行できます。

多少はVimのテキスト編集の強力さが伝わったかと思います。

## VimとVSCodeのプラグイン開発

自分はVSCodeのプラグインは軽くしか作ったことがないのですが、Vimのプラグインと比較して開発から導入までの敷居にかなりの差があると思っています。
(VSCodeのプラグインはskeletonをベースに色々な箇所を書き直してbuildしてからVSCodeにインストールしないといけないイメージがあります、勘違いならすみません)

Vimは特定の規約に沿ってディレクトリを作成し、そのディレクトリのパスを設定ファイルに記述するだけでプラグインとして動作します。
また、GitHubにリポジトリを作成しておけば、buildなどの作業なしで誰でもそのプラグインが使えるようになります。

例えば自分は直近で[ripgrep](https://github.com/BurntSushi/ripgrep)のgrep結果をcocの補完ソースにする[coc-rg](https://github.com/yuki-yano/coc-rg)を作成したのですが、リポジトリをskeletonから1つ作り、その中に以下のようなファイルを1枚置くだけでインストールが完了しました。(ただこれはcoc extensionsなのでTypeScriptで書かれており、特殊なプラグインではあります)

これだけの手間とコードで実用的なプラグインを開発できるのはVimの大きなメリットだと思っています。

```typescript
import { CompleteResult, ExtensionContext, sources } from 'coc.nvim';
import { execSync } from 'child_process';

export const activate = async (context: ExtensionContext): Promise<void> => {
  context.subscriptions.push(
    sources.createSource({
      name: 'rg',
      doComplete: async ({ input }) => {
        return await getCompletionItems({ input: input.replace(/([\\\[\]^$.*])/g, '\\$1') });
      },
    })
  );
};

const options = [
  '--no-filename',
  '--no-heading',
  '--no-line-number',
  '--color=never',
  '--only-matching',
  '--word-regexp',
  '--ignore-case',
];

const getCompletionItems = async ({ input }: { input: string }): Promise<CompleteResult> => {
  input = input.replace(/^-+/, '')

  if (input.length < 2) {
    return { items: [] };
  }

  const stdout: string = execSync(`rg ${options.join(' ')} ${input}[a-zA-Z0-9_-]+ ${process.cwd()}`, {
    encoding: 'utf-8',
    maxBuffer: 1024 * 1024 * 1000,
  });

  return {
    items: stdout.split('\n').map((word) => ({ word, menu: '[rg]' })),
  };
};
```

また、Vim Pluginを開発する際には[vim-jp Slack](https://vim-jp.org/docs/chat.html)に来ればVim scriptにすさまじく習熟した人たちが多数いるため、気軽に質問できます。
自分も質問した際には大規模なリファクタなどを行う事になることなどがあったりします。

## 最後に

この記事では自分がどうしてVSCodeではなくNeovimを用いて開発しているかを解説しました。
実際、VSCodeの方がVimよりも導入コストや慣れのという点で圧倒的に優れていると思っているのですが、自分がVimに情熱を注いでいる理由が少しでも伝われば幸いです。

また、この記事を見てVimに興味の沸いた方がいらっしゃったら是非一度Vimでの開発を試してみてください。

もしVimについて知りたいことや質問があれば[vim-jp Slack](https://vim-jp.org/docs/chat.html)や[Twitter](https://twitter.com/yuki_ycino)で連絡を頂ければ答えられると思うので、是非お願いします。
それではより良いVimライフをお送りください！
