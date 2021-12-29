---
title: "1つの発言から数時間でVimプラグインが2つ+αできた話"
emoji: "⚡"
type: "tech"
topics: ["Vim", "Neovim"]
published: true
---

## 概要

相変わらずvim-jpコミュニティのフットワークの軽さと勢いがすごいのを感じたので、そのときに開発されたプラグインの紹介、また開発に至った流れやどういうことがあったのかを書いてみる記事です。
数日前に自分が行った1つの発言が発端で、すごい勢いでプラグインが開発されていきました。
フィードバックも多く、機能追加・修正・改善も数分単位が行われるような盛り上がりでした。

そこから生まれたプラグインは以下の3つです。

- [yuki-yano/fuzzy-motion.vim](https://github.com/yuki-yano/fuzzy-motion.vim) (自作)
- [hrsh7th/vim-seak](https://github.com/hrsh7th/vim-seak)
- [hrsh7th/vim-searchx](https://github.com/hrsh7th/vim-searchx)

なお、どのような流れで開発が進んでいったかは[開発の流れ](#開発の流れ)を見てみてください。

## プラグイン紹介

easymotion系及び、Vimの検索機能を強化するプラグインが作られました。
easymotionはVSCodeでいう[jumpy](https://marketplace.visualstudio.com/items?itemName=wmaurer.vscode-jumpy)のようなプラグインです。

### fuzzy-motion.vim

[fuzzy-motion.vim](https://github.com/yuki-yano/fuzzy-motion.vim)は画面内の単語一覧を候補にfuzzy matchを行い、一致したものにラベルを付与してジャンプするためのプラグインです。
[denops.vim](https://github.com/vim-denops/denops.vim)に依存しており、実装はほぼ全てDeno + TypeScriptで行われています。

既存のeasymotion系のプラグインとの違いとして、fuzzy matchなので特徴のある文字を雑に入力してジャンプ先を絞り込めるのが優位点です。
例えば `fooBarHoge` のような単語には `foobar` でもいいし、 `fbh` でもマッチします。
複数の単語境界を設定可能（デフォルト設定で対応済み）で、複数の単語が繋がっている `array.map(item => item)` のようなものに対しても `arrmi` のような入力で先頭にマッチします。

TypeScriptを使って実装することで、npmからDenoにportingされた資産をそのまま使うことができたため実装工数も少なめでした。
具体的にはfuzzy matchの処理とスコアリングについては[fzf-for-js](https://github.com/ajitid/fzf-for-js)をそのまま使わせて貰っています。

以下がデモです。単語に対してファジーマッチしてジャンプしているのが分かるかと思います。
@[youtube](-9uptIcApQU)

### vim-seak

[vim-seak](https://github.com/hrsh7th/vim-seak)はVim標準の検索(incsearch)を強化して、検索中にラベリングを行うことで目的の箇所に楽にジャンプするためのプラグインです。
Vimデフォルトの機能をHackしている都合上、かなりトリッキーな実装が必要になっていた反面、普段使っている検索機能が強化されるということで頻繁に行う操作が改善されるというメリットがあります。

標準のincsearchの動作が邪魔になることがあって機能の方向性にかなり悩んでいたのですが、結果としてvim-seakはPoCとしての役割を終え、[vim-searchx](https://github.com/hrsh7th/vim-searchx)へと開発が引き継がれることになりました。

以下がデモです。標準の検索中に候補の1文字左にラベルが表示され、ラベルの文字を押下するとそこまでジャンプしているのが分かるかと思います。
@[youtube](rAWvskphxs4)

### vim-searchx

[vim-searchx](https://github.com/hrsh7th/vim-searchx)はvim-seakで得た知見を元に1から検索機能を開発しているプラグインです。
標準の検索機能をエミュレートした独自の検索機能として実装しているため、vim-seakに比べて自由度は高そうです。

現状はvim-seakと大体同等の機能を実装している途中のため、デモは省略します。
また、プラグイン名が変更される可能性もありそうです。

## 開発の流れ

これらのプラグインの開発に至った流れを書いていきたいと思います。

### 使いやすいeasymotion系プラグインがほしい・・・

数日前に自分が以下のような発言を[vim-jp Slack](https://vim-jp.org/docs/chat.html)で行いました。

既存のeasymotion系のプラグインがなんとなく使いづらいので、改善したプラグインを作りたいという旨の発言でした。
これは多くの人が感じている問題でeasymotionのラベリングによる移動は認知負荷が高いという話も以前から何度か話題になっていました。

![slack_log_1](https://user-images.githubusercontent.com/5423775/147655549-02540336-1acb-4cad-a917-182c94d785c7.png)

その後、脳内でなんとなく使いやすいかもしれない案が思いついたので、実装してみようという旨の発言をしました。
VimのプラグインだしPoCを作って使ってみる程度なら数時間でいけそうじゃない？ってくらいの気持ちでした。

![slack_log_2)](https://user-images.githubusercontent.com/5423775/147655559-2cad8299-bbbb-42d5-8a82-f420ed0403bf.png)

直後に[hrsh7th](https://github.com/hrsh7th)さんも類似のプラグインを試しに作ってみるみたいな流れになって、勢いで開発が始まりました。
(hrsh7thさんは[nvim-cmp](https://github.com/hrsh7th/nvim-cmp)などのNeovimで非常にシェアが高いプラグインを開発している方です)

![slack_log_3](https://user-images.githubusercontent.com/5423775/147655571-74ec4efb-6b89-4a66-8ef1-f866d558c764.png)

### 最初の発言から数時間後にPoCが完成

hrsh7thさん作のPoCが完成してました。あまりにも早すぎる…
（なお自分は少し体調が悪くなってたので寝ていてこのタイミングを見逃しました）
[hrsh7th/vim-seak](https://github.com/hrsh7th/vim-seak)

![slack_log_4](https://user-images.githubusercontent.com/5423775/147655579-314352f2-2440-45b1-880e-cff1115c1678.png)

次が異常な速度でたくさんの人が試してフィードバックが出てる一部の例です。
この後もすごい勢いでログが流れて開発が続いていました。

![slack_log_5](https://user-images.githubusercontent.com/5423775/147657673-2b5ab3d0-008e-4315-a135-c99f792b87ee.png)

### 翌朝

寝てる間にめっちゃ開発が進んでるログを見て、自分も[vim-seak](https://github.com/hrsh7th/vim-seak)を試してみました。
試してみて（コンセプトは違いますが）自分が考えていた[fuzzy-motion.vim](https://github.com/yuki-yano/fuzzy-motion.vim)も割といけそうに思ったので、昨日ちょっとだけやってた開発を再開しました。

![slack_log_6](https://user-images.githubusercontent.com/5423775/147655681-1606a17d-7d63-42c5-922f-4c96e7f8f2ef.png)

### 起きてから数時間後に自分のPoCが完成

雑に作ってみてとりあえず自分も最低限試せる形ものが数時間でできました。
何人かに試して貰ってかなり手応えがあったので、このタイミングでもう少しちゃんと作りこむことにしました。
[fuzzy-motion.vim](https://github.com/yuki-yano/fuzzy-motion.vim)

![slack_log_7](https://user-images.githubusercontent.com/5423775/147655688-878317ca-d5bb-4a32-bddf-2f7906f1b86e.png)

### その後…

[vim-seak](https://github.com/hrsh7th/vim-seak)がすごい盛り上がりで、開発が活発に進んでいました。
[fuzzy-motion.vim](https://github.com/yuki-yano/fuzzy-motion.vim)もドッグフーディングをしつつフィードバックをいくつか貰いながら改善を進めていきました。

1-2日後に[vim-seak](https://github.com/hrsh7th/vim-seak)はPoCとしての役目を終えて、[vim-searchx](https://github.com/hrsh7th/vim-searchx)で1から作り直されることになりました。
[fuzzy-motion.vim](https://github.com/yuki-yano/fuzzy-motion.vim)は機能がシンプルなプラグインのため、細かいところの調整や修正をし、無事ちゃんとしたプラグインとしてリリースが完了しました。

## Vimのプラグイン開発はとても簡単

上記の勢いが異常だったのもあるのですが、そもそもVimプラグインは開発・試用・公開までの手数がとても少なくて簡単に行う事ができます。
以下の記事でも少し書いたのですが、実用的なプラグインが数時間で作れるのはVimの大きな魅力だと思います。
Vimに興味のある方がいれば、一度プラグイン開発をしてみると面白い体験ができると思います。（今ならdenopsを使ってTypeScriptでの開発もできますし）

https://zenn.dev/yano/articles/why_use_neovim_instead_of_vscode_for_development

## 最後に

今回は自分の適当な発言を発端に有用なVimプラグインが数時間で(自分は翌日なのでちょっと盛っていますが)2つ+α開発された事例を紹介しました。
今回のは一例ですが、[vim-jp Slack](https://vim-jp.org/docs/chat.html)では頻繁にこのような活動が行われています。
[エンジニアの楽園 vim-jp](https://blog.tomoya.dev/posts/vim-jp-is-a-paradise-for-engineers/)にもあるようにVimに限らず幅広い話題で賑わっていますので、もし興味の沸いた方がいれば参加していただければ幸いです。
