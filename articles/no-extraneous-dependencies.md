---
title: モノレポでの依存設定忘れを防ぐ
emoji: 🖇
type: tech
topics: [nodejs, markuplint, gulp, monorepo, eslint]
published: true
---

## 結論

**eslint-plugin-import**の[`import/no-extraneous-dependencies`](https://github.com/import-js/eslint-plugin-import/blob/main/docs/rules/no-extraneous-dependencies.md)を使え。

以下、経緯。

## 概要

- 事の発端はこれ: [gulp v4.0.2 を使っている環境で markuplint v3.3.0 を導入するとエラーになる](https://dskd.jp/archives/120.html)
- Issueは https://github.com/markuplint/markuplint/issues/740

もっと早く気付けばよかったのだけど、

```shell
error: TypeError: debug_1.green.bold is not a function
```

という時点で[ansi-colors](https://www.npmjs.com/package/ansi-colors)を使った処理に関するところなので、そのパッケージまわりを見ればよかった。
[越智さん](https://twitter.com/otiext)が直接の依存パッケージをすべて洗ってくれていたので、その後の特定は早かった。ありがとうございます。

## 原因

結論から言うと、**依存パッケージの設定し忘れ**。

エラーが起こった箇所は`@markuplint/rules`の一部のコードで起こっている。この`@markuplint/rules`のdependenciesに`ansi-colors`を**設定し忘れ**ていただけだった。**完全に凡ミス**。恥ずかしいを通り越して情けない。

## 何故気づけなかったのか

- `@markuplint/rules`はMarkuplintのデフォルトのルールを集めたパッケージで単体で利用するものではない
- `markuplint`本体が[enquirer](https://www.npmjs.com/package/enquirer)というパッケージを採用していて、これが**ansi-colors**に依存していた
- `@markuplint/rules`はモノレポで`markuplint`と一緒に管理・開発しているので、開発中はもちろん、CIのテストでも**enquirer**経由で**ansi-colors**が常にインストールされた状態なので、dependenciesに`ansi-colors`を設定していなくても問題なかった
- さらに基本的な利用においても`markuplint`は`@markuplint/rules`に依存しているので、`@markuplint/rules`と`ansi-colors`はほぼ必ず一緒にnode_modulesに入ることになる

ちなみに`ansi-colors`がどういうパッケージを経由して依存しているかは次のコマンドで確認できる

```shell
yarn list ansi-colors
```

## gulpでは何が起きていたのか

問題が起こる**gulp**関連のパッケージはほとんどが[plugin-error](https://www.npmjs.com/package/plugin-error)を経由してansi-colorsの`v1.0.1`に依存している。Markuplintが**enquirer**経由で依存していたansi-colorsのバージョンは`v4.1.1`。

ツリーを作るとこんな感じ。

- dskd.jp
  - └ gulp-\*
    - └ plugin-error
      - └ **ansi-colors@1.0.1**
  - └ markuplint
    - └ enquirer
      - └ **ansi-colors@4.1.1**
    - └ @markuplint/rules
      - └ **ansi-colors設定し忘れ**

**ansi-colors設定し忘れ**ている`@markuplint/rules`は、`import * from "ansi-colors"`したときに**ansi-colors@1.0.1**と**ansi-colors@4.1.1**のどちらかをインポートする。どうやらここでインストールされた順番が関係してくるらしい。

越智さんの検証では、要するに**ansi-colors@1.0.1**が先にインストールされているケースで問題が発生していた。`@markuplint/rules`は**ansi-colors@1.0.1**をインポートし、その結果v1とv4のAPIの違いから`error: TypeError: debug_1.green.bold is not a function`というランタイムエラーを引き起こしてしまったようだ。

`@markuplint/rules`に`"ansi-colors": "^4.1.3"`を追加し、Canaryリリース（`markuplint@3.0.0-dev.96+3b9f1720`）をして試したところ正常に動作することを確認できた。**ansi-colors@1.0.1**を先にインストールしていたとしても、`@markuplint/rules`がきちんと**ansi-colors@4.1.3**を持つようになるので問題は解消された。実際に問題が解消されたnode_modulesのディレクトリを覗くと以下のように解消されている。

- node_moduules
  - └ ansi-colors (1.0.1)
  - └ enquirer
    - └ node_modules
      - └ ansi-colors (4.1.3) ※キャレット設定されているので設定バージョンより上がる
  - └ @markuplint/rules
    - └ node_modules
      - └ ansi-colors (4.1.3)

## どうやってこのミスを防ぐのか

原因がわかったところで、このミスを防ぐ方法を考えないといけない。依存パッケージによってはすぐに見つかるだろうけど、今回のは組み合わせが絶妙すぎたとも思う。gulp以外でもansi-colorsを依存にもつパッケージを組みわせてMarkuplintを利用している人もいると思うけど、おそらくansi-colorsがv1.xではなく偶然APIが一致していてエラーを起こしていないだけだろう。たまたま上手くいってる。**このたまたまというのがプログラミングの世界では一番怖いんだよ**。なんとかしたい。

ということで調べてみるとESLintのプラグインの**eslint-plugin-import**に[`import/no-extraneous-dependencies`](https://github.com/import-js/eslint-plugin-import/blob/main/docs/rules/no-extraneous-dependencies.md)というルールがある。依存設定していなかったり`devDependencies`に設定しているものをインポートしようとすると警告するものだ。

**eslint-plugin-import**は他のルールを利用していて導入していたのに、このルールを有効にしていなかった…。なんてことだ。もったいないことしていたし、実際迷惑をかけてしまった。申し訳ない。

これを読んでESLintのルール周りに詳しい方がいたら、Markuplintのpackage.jsonを見てPRしてほしい。「これ導入/有効にしてないのかよ、ありえんのやけど」というやつがあったら是非よろしくおねがいします。
