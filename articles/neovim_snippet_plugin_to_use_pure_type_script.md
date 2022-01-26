---
title: "TypeScriptでプログラマブルにSnippetを定義/実装できるNeovimプラグインを作った"
emoji: "🛠️"
type: "tech"
topics: ["Vim", "Neovim", "TypeScript", "Deno"]
published: true
---

フロントエンドエンジニアをしているYano ([@yuki_ycino](https://twitter.com/yuki_ycino)) といいます。
この記事は自分が先日作ったNeovim用のSnippetプラグイン、[tsnip.nvim](https://github.com/yuki-yano/tsnip.nvim) の紹介記事になります。

技術的には最近のVim界隈で使われている諸々を組み合わせて面白いことをやっていると思っていて（Vim scriptとLuaとDeno(denops)を組み合わせた闇鍋感）、また既存のSnippetプラグインとはかなり発想が違ったプラグインとなっているため、興味がある方は読んでいただけると幸いです。

## 前置き

皆さんはSnippetプラグインを利用したことがあるでしょうか。
VSCodeでも割と使われていて、 `if` や `console.log` や `class` のような構文を使いやすい形にさっと展開してくれる便利なやつです。

Vimにも複数のSnippetプラグインがあり、以下の記事で比較もされています。
https://zenn.dev/shougo/articles/snippet-plugins-2020

この記事では自分がPure TypeScript Object (Deno) を使ってSnippetを定義できるプラグインを開発した話と、その経緯について解説します。

## デモ

まず最初にデモを見てみてください。
なんとなく動作のイメージが掴めるのではないかと思います。
詳細についてはあとで解説します。

@[youtube](XiXRz3SvoXM)

## モチベーション

自分が書いた [無人島に持っていく(Neo)vimプラグイン10選 (TS開発環境編)](https://zenn.dev/yano/articles/vim_plugin_top_10) という記事で解説しているのですが、自分は今まで [ultisnips](https://github.com/SirVer/ultisnips) というSnippetプラグインをメインで使っていました。
このプラグインは機能的にはおそらく最強だと思っているのですが、個人的にはいくつか不満点がありました。

- snippet定義が独自DSL
  - とはいえ他のプラグインもJSONで管理しているものなどが多く、それもあまり好みではないのですが・・・
- プログラマブルに書けるが、DSLの中にPythonをインラインで埋め込んで実装する必要がある
  - Python in DSLと、Python依存の両方ともあまり好みではない

というのがあり、Pythonが使えることで非常に複雑なSnippetを定義するのが強いのですが、DSLの中にPythonのコードを書いて頑張るのがなかなか辛い感じでした。

そこで、いっそのこと [denops.vim](https://github.com/vim-denops/denops.vim) を使ってPure TypeScriptでSnippet定義をできるようにすればDSL不要でSnippetを書いていくことができると思い、実装することにしました。

## Snippetプラグインの実装の難易度

開発するのを決めたのはいいのですが、Snippetプラグインの開発には非常に大きな壁があります。
それは展開後の入力に対する複雑な状態(カーソル位置や入力の複製など)の管理を行う必要があるということです。
エディタはStateの塊のようなものなので、入力に対するSnippet状態管理は非常に複雑な実装になります。

どのような機能が必要かは前述の [スニペットプラグインについて 2020 年版](https://zenn.dev/shougo/articles/snippet-plugins-2020) を見ればなんとなく分かるかと思います。
また、プログラマブルな処理を挟み込みたい場合は更に難易度が上がります。

## tsnip.nvim での解決策

今回作成した [tsnip.nvim](https://github.com/yuki-yano/tsnip.nvim) では、開き直ってそのあたりの状態管理を一切持たないことにしました。
どういうことかというとSnippet展開時に受け取るパラメータをInteractiveに入力し、全てのパラメータの入力が完了したタイミングでパラメータを注入したSnippetを1つのテキストとして挿入しています。
この手法だと通常のプラグインと違い展開後のState管理は不要になるため、実装をシンプルにすることができます。

入力についてはNeovimのUIライブラリである [nui.nvim](https://github.com/MunifTanjim/nui.nvim) を使い、レンダリング結果のPreviewにはVirtual Lines(Ghost Text?)を使っています。
ユーザの入力を受けながら最終的に挿入されるテキストを確認することができるUIになっているため、既存のSnippetプラグインのUXとは違いますが人によってはシンプルで分かりやすいものになったのではないかと思っています。

以下がuseStateの入力に[tsnip.nvim](https://github.com/yuki-yano/tsnip.nvim)を使った比較的シンプルな例です。

@[youtube](f0D5ktqBRKg)

上記のSnippet定義はTypeScript(Deno)でこのように実装しており、ただのTSファイルなのでLSPの恩恵(型安全など)を受けながら書くことができます。
useStateのsetterの部分は入力されたstate名からTSで文字列を変換して動的に生成しており、プログラマブルなSnippetの実装が非常に強いことが分かると思います。

以下がこのSnippetの定義です。

```typescript
import { Snippet } from "https://deno.land/x/tsnip_vim@v0.3/mod.ts";

export const state: Snippet = {
  name: "useState",
  text: "const [${1:state}, set${State}] = useState(${2:default_value})",
  params: [
    {
      name: "state",
      type: "single_line",
    },
    {
      name: "default_value",
      type: "single_line",
    },
  ],
  render: ({ state, default_value }) =>
    `const [${state?.text ?? ""}, set${
      state != null
        ? `${state.text?.charAt(0).toUpperCase()}${state.text?.slice(1)}`
        : ""
    }] = useState(${default_value?.text ?? ""})`,
};
```

このように多少冗長な記法にはなってしまうのですが、個人的にはプログラマブルなSnippetを定義するには非常に使いやすいプラグインになりました。
また、このSnippetでは使っていないですがDenoのサードパーティのライブラリをimportしてそのまま使うこともできます。

## 複雑な例

[デモ](#デモ)の例が自分が一番複雑に使っているものなので解説します。
これは自分がReactでComponentファイルを作ったときに雛形として使っているものです。

2種類のPropsを複数行で入力し、それをparseしながらComponentの引数側でDestructuringした結果を展開するというものです。
あとはtsnipの機能としてファイル名を受け取ることができるので、ファイル名を元にexportするComponentの名前を動的に生成もしています。

具体的な実装は以下になります。

```typescript
import { Snippet } from "https://deno.land/x/tsnip_vim@v0.3/mod.ts";
import { pascalCase } from "https://deno.land/x/case@v2.1.0/mod.ts";

const parseProps = (props: string) =>
  props.split("\n").map((line) => (line.split(":").at(0))).filter((line) =>
    line != null
  ).join(", ");

export const fct: Snippet = {
  name: "FC file template",
  params: [
    {
      name: "ContainerProps",
      type: "multi_line",
    },
    {
      name: "Props",
      type: "multi_line",
    },
  ],
  render: ({ ContainerProps, Props }, { fileName }) => `
import styled from "@emotion/styled"
import type { FC } from "react"

type ContainerProps = {
  ${ContainerProps?.text?.split("\n").join("\n  ") ?? ""}
}

type Props = ContainerProps & {
  ${Props?.text?.split("\n").join("\n  ") ?? ""}
}

const Component: FC<Props> = ({ ${
    parseProps(`${ContainerProps?.text ?? ""}\n${Props?.text ?? ""}`)
  } }) => {
  return <></>
}

export const StyledComponent = styled(Component)\`\`

const ContainerComponent: FC<ContainerProps> = (props) => {
  const { ${parseProps(ContainerProps?.text ?? "")} } = props

  return <StyledComponent {...props} />
}

export const ${
    pascalCase(fileName.text.replace(/\.tsx$/, "") ?? "")
  } = ContainerComponent
    `,
};
```

## 他のSnippetプラグインとの使い分け

少しの間使っていたのですが、 [tsnip.nvim](https://github.com/yuki-yano/tsnip.nvim) はプログラマブルな大きめのSnippetには向いているのですが、(パラメータの入力に補完を効かせることができないなど)UIの都合上小さめのSnippetには一般的なSnippetプラグインの方が向いているかと感じました。

どちらかというとテンプレートに近い特性のSnippetについて [tsnip.nvim](https://github.com/yuki-yano/tsnip.nvim) を使うのが向いているかと思います。
そのため、自分はtsnipと並行して何らかのSnippetプラグインを使っていく予定です。

## UIについて

UIは [nui.nvim](https://github.com/MunifTanjim/nui.nvim) を使って、Inputの部分は簡単にリッチに作ることができました。
また、イベントハンドリング周りも簡単に定義できたのでPreviewと連携させるのも難しくなかったです。

Previewについては比較的最近Neovimに実装された[Virtual Lines](https://github.com/neovim/neovim/pull/15351)を使うことでいい感じに出すことができました。
これは特定の行の下にファイルに干渉しない仮想の行を挿入するという機能です。
Virtual Linesについては面白いけど使い道が思いつかないなってずっと思っていたものだったので、今回丁度いい用途が見つかってよかったです。

今回はこの2つを組み合わせることで、個人的には割と理想に近いUIを実現できました。

## プラグインの動作環境と依存関係

今回はUIの実装の都合上、Neovim限定のプラグインになりました。
LuaのUIライブラリ([nui.nvim](https://github.com/MunifTanjim/nui.nvim))を使っている都合上、Luaでイベントハンドリングのコードを書かないといけなく、Denoを取り扱うために[denops.vim](https://github.com/vim-denops/denops.vim)への依存も必要で、Vimの機能と一部繋ぎ込むためにVim scriptも少し書いています。
自分としてはいいプラグインになったと思っているのですが、依存が重くなってしまったとは思っています。(自分は更に[coc.nvim](https://github.com/neoclide/coc.nvim)から使っているためにNode依存もしちゃっています)

## 最後に

この記事では自分が作成したSnippetプラグインの[tsnip.nvim](https://github.com/yuki-yano/tsnip.nvim)について解説しました。
自分的にはプログラマブルに使えるSnippetプラグインとしてはかなり理想のものを作る事ができたと思っていて、この後も引き続き改善していきたいと思っています。

この記事を読んで、もし興味が沸いた人がいらっしゃれば試しに使っていただけると幸いです。

それではより良いVimライフをお送りください！
