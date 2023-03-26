---
title: 【仕様の読み方】HTMLの要素をどうやって学ぶか
emoji: 🔍
type: idea
topics: [html, waiaria, accessibility]
published: true
---

`<search>`要素がHTML Standardに追加されました。私も初めて出会う要素になるわけですが、とても良い機会なので、私が要素を調べる際に**どうやって調べて学んでいるのか**を共有したいと思います。これは新しい要素に限らず、既存の要素の調査に応用できると思います。また、初学者はもちろん、マークアップを生業としている方にも参考になると思います。

## 新要素追加の経緯を調べる

:::message
ここは興味がある人だけで構わないと思います。事実だけを知りたい人は混乱の元になる可能性があるので飛ばしてください。
:::

まずはHTML StandardのGitHubのPRからスタートするとよいでしょう。議論や更新はGitHubで行われています。たとえば、今回の`<search>`は[Add the &lt;search&gt; element #7320](https://github.com/whatwg/html/pull/7320)というPRによって更新されました。

そもそも更新自体のキャッチアップ方法は[クローズされたPRを更新順にして確認](https://github.com/whatwg/html/pulls?q=is%3Apr+is%3Aclosed+sort%3Aupdated-desc)してもいいですし、更新のみをツイートしている[@htmlstandard](https://twitter.com/htmlstandard)のTLを確認してもいいと思います。私の場合はTwitterを見ていることが多いです。`<search>`の追加もこれで知りました。

https://twitter.com/htmlstandard/status/1639083303117651968

ツイートのリンクからはPRの本体ではなく差分のページに移動します。そこからPRの本体に移動するには**ブランチとPR番号**が記載されているところから、リンクとなっているPR番号をクリックすることで移動できます。

![スクリーンショット: GitHubの差分のページで、ブランチとPR番号の部分に印をつけている](/images/learn-new-html-elements/ss-pr.png)

あとはそのPRに本体に記載されている内容や関連Issue、関連PR、関連提案文を確認しましょう。こういったものは基本的に提案から始まっているので、最初の提案と最終的な仕様に食い違いが生まれるのはよくあります。もしかしたら結果から確認したほうが混乱が少ないかも知れません。

## 要素の仕様が記載されているページを確認する

要素の仕様から見ていきましょう。まずはそのページの探し方ですが、[目次ページ](https://html.spec.whatwg.org/multipage/indices.html)を見るとよいでしょう。要素や属性の一覧が表で記載されています。ブラウザのページ内検索機能を使ったりして該当の要素を探します。今回は`<search>`を探すので[Elements](https://html.spec.whatwg.org/multipage/indices.html#elements-3)の表から`search`を検索します。

![スクリーンショット: 要素一覧で「search」と検索してsearch要素の行がヒットしている様子](/images/learn-new-html-elements/finding-in-indices.png)

この時点で、“Container for search controls”と概要が記載されていたり、コンテンツカテゴリが「フローコンテンツ」と「パルパブルコンテンツ」であること、期待される親や子のコンテンツモデルが「フローコンテンツ」であること、もつことができる属性がグローバル属性のみであること、IDLインターフェイスが`HTMLElement`であることが既にわかります。ここをざっと読むだけでも有用ですね。把握できる人はここで十分かもしれません。ただし、ここは更新漏れがよくPRに挙がることがあるので、しっかりとした調査においては鵜呑みにせずに仕様ページを見ましょう。この表は非規範的（_non-normative_）でもありますし。

先頭の見出しセルがリンクになっているので、そこから要素の仕様ページに移動しましょう。`<search>`は“[4.4.15 The search element](https://html.spec.whatwg.org/multipage/grouping-content.html#the-search-element)”というセクションで説明さています。セクション番号は特に気にする必要はないでしょう。分類上この番号になっているだけです。要素の追加や削除でも番号が変わります^[今回、`<search>`の追加により`<div>`が4.4.15から4.4.16に移動しました。そんなもんです。]。

## 要素の性質を確認する

![スクリーンショット: 4.4.15 The search elementセクションの要素の性質リスト](/images/learn-new-html-elements/the-search-el-specification.png)

要素の性質がリストアップされています。この性質はとても重要で、HTMLの仕様を確認する上での「字句的ルール」「語彙的ルール」「意味論的ルール」^[書籍[HTML解体新書](https://www.borndigital.co.jp/book/25999.html)より。P14「02 HTMLの仕様 - コンテンツ制作者のルール」参照。]のうち、「**字句的ルール**」「**語彙的ルール**」がここで規定されていると思ってください。

### Categories

要素自身がコンテンツモデルにおけるどのコンテンツカテゴリに属するのかです。`<search>`は「フローコンテンツ」と「パルパブルコンテンツ」に属していることがわかります。

実はここはあまり気にする必要はありません。コンテンツカテゴリとしては一覧を見ることのほうが遥かに多いので「ここを頭に入れておかねば…！」なんて決して思わないでください。覚える必要はありません。どちらかというと重要なのは、次に続く2つです。

### Contexts in which this element can be used

これは訳すと「どの文脈で使えるか？」ですが、つまり「親」や「先祖」は何が許可されるか、です。`<search>`の場合は“Where flow content is expected.”となっているので、「親要素 = フローコンテンツ」であることが求められます。

ここで初めて、親要素のコンテンツカテゴリを調べる、という順番でいいのです。

### Content model

Content（内容）として許可できるコンテンツカテゴリを示します。つまり「子」や「子孫」は何が許可されるか、です。`<search>`の場合は「フローコンテンツ」を許可しています。

`<search>`の場合はこれだけでとてもシンプルですが、たとえば`<div>`は複雑です（同じページの次の要素なのでスクロールして移動してみてください）。

> Content model:
>
> If the element is a child of a dl element: one or more dt elements followed by one or more dd elements, optionally intermixed with script-supporting elements.
> If the element is not a child of a dl element: flow content.

要約すると、次のいずれかを満たすことを要求しています。

- `<div>`が`<dl>`以外の子要素の場合、子要素にフローコンテンツを許可する
- `<div>`が`<dl>`の子要素の場合、子要素は1つ以上`<dl>`に加えて1つ以上`<dd>`が続く（さらにスクリプトサポーティング要素が間に混ざっても良い）

「あー、待って待って待って…なんて…？」ってなるかと思います。コンテンツモデルの規定には、こういう複雑なケースがあります。親要素が何であるかという条件がついたり、子要素の要求としてコンテンツカテゴリではなく特定の要素や、数（例の場合「1つ以上」）の制限も設けます。シンプルな方が稀だと思ってください。

**要素自身のコンテンツカテゴリだけ把握しても、要素の入れ子関係が把握できるわけではない**ということがわかると思います。**親・先祖のルール、子・子孫のルールまで含めて1つの「入れ子ルール」として捉える必要があります。**

### Tag omission in text/html

これはタグの省略の可否です。`<search>`では“Neither tag is omissible.”となってるので「どちらも」省略できません。「どちらも」というのは「開始タグ」と「終了タグ」のことを表しています。「終了タグ」の省略の仕様を知っている人は多いと思いますが、実は「開始タグ」を省略できる要素もあるのでこう書かれています。

“in text/html”というのもあくまでも「HTML構文では」ということですね。XMLやXHTML、SVGその他、厳密にHTMLと異なる場所ではその限りではないということですね。まあ、ただそんなケースは滅多にお目にかかれないですが。

### Content attributes

これは要素自身がもてる属性が規定されます。`<search>`ではグローバル属性のみですね。要素によっては、ここに要素固有の属性がリストアップされます。要素固有の属性のそれぞれは解説は、要素自体の解説あとに続きます。

### Accessibility considerations

アクセシビリティ関連の規定ですが、要するにWAI-ARIAの規定です。“For authors”は、HTMLを記述する人が参考にするべき資料で[ARIA in HTML](https://w3c.github.io/html-aria/)にリンクされます。一方、“For implementers”は、UA（ブラウザ）実装者が参考にするべき資料で[HTML AAM (HTML Accessibility API Mappings)](https://w3c.github.io/html-aam/)にリンクされます。

おそらくこの記事を読んでいるのはHTMLを記述するウェブ制作やUI開発する方々だと思いますので、“For authors”の[ARIA in HTML](https://w3c.github.io/html-aria/)を確認しましょう。

![スクリーンショット: “Rules of ARIA attribute usage by HTML element”表のsearch要素](/images/learn-new-html-elements/aria-in-html-search-el.png)

要素がもつ**暗黙のセマンティクス（Implicit ARIA semantics）**（「暗黙のロール」と言い換えてもよい）と、設定して良いロール、ARIAステート・プロパティが規定されています。

`<search>`は暗黙のロールが`search`であり、`form`、`group`、`none`、`presentation`、`region`というロールの上書きが許可されています。そして“Global aria-_ attributes and any aria-_ attributes applicable to the allowed roles.”という規定は、まず[グローバルARIAステート・プロパティ](https://www.w3.org/TR/wai-aria-1.2/#global_states)が許可されていることを示し、加えて「ロールに規定されている固有のARIAステート・プロパティ」が許可されることを意味しています。

この「ロールに規定されている固有のARIAステート・プロパティ」というのは、その要素がどのロールの状態にあるかによって変わります。`<search>`は暗黙のロールが`search`なので、何も指定がなければ[`search`ロール](https://www.w3.org/TR/wai-aria-1.2/#search)の規定するARIAステートとプロパティが許可されますが、`<search role="form">`としていたら、それは[`form`ロール](https://www.w3.org/TR/wai-aria-1.2/#form)に規定されているものしか許可されません。ロールによって異なるので「要素 = 許可されるARIAステート」と**直接紐付けないように注意しましょう**。

### DOM interface

これはJavaScriptでDOM操作をするときに把握するべき規定です。`<search>`では`HTMLElement`に規定されているので、現段階では特別にプロパティやメソッドをもつわけではないということがわかります。要素固有のインターフェイスをもつ場合はここが`HTMLSearchElement`のような名前になっていることが多いです。その場合、要素固有のプロパティやメソッドのそれぞれは解説は、要素自体の解説あとに続きます。

---

さて、ここまでの性質は、章の冒頭で述べたとおり「**字句的ルール**」と「**語彙的ルール**」です。解釈の余地はありません。完全に決まっているルールです。「こうかもしれない」「こういう可能性がある」といった曖昧な規定ではありません。それは機械的にチェック可能であることを意味します。つまり**覚える必要はありません**。自分の書いたHTMLがこの規定に従っているかどうかは、[Nu Html Checker](https://validator.w3.org/nu/#textarea)、[axe DevTools - Web Accessibility Testing](https://chrome.google.com/webstore/detail/axe-devtools-web-accessib/lhdoppojpmngadmnindnejefpokejbdd)、[axe Accessibility Linter](https://marketplace.visualstudio.com/items?itemName=deque-systems.vscode-axe-linter)、[Markuplint](https://markuplint.dev/)などのツールに頼れば解決します。もう一度言いますが、**覚える必要はありません**。ツールを導入しましょう。

## 意味論的ルールを読み解く

さて、ここからが本番です。ここから曖昧さが含まれ、解釈の余地が入ってきます。

![スクリーンショット: search要素の説明の冒頭からNoteとExample](/images/learn-new-html-elements/explanation-search-el.png)

- 本文の説明
- Note（注釈）
- Example（コード例）

それぞれがいくつか含まれることが多いので、それらを見ながら読み解いていきましょう。本文の説明を基本として、注釈はあくまで注釈として読み解く必要がありますが、わりと**重要なことが注釈に含まれているケースがある**ので、注意が必要です。また、注釈はコード例の前フリをしている場合もあるので、**読み飛ばせるところはほぼ無い**と思ってください。

本文や注釈に含まれる用語に対してリンクが貼られていることがありますが、それが解読のヒントになり、また**字面だけで解釈しないようにリンク先の用語の解説や使用される文脈もきちんと読んだほうが理解の齟齬が減る**でしょう。

また、この解説部分は更新されることがあります。特に新しく登場した要素や属性があれば、頻繁に更新されるでしょう。最近だと`<dialog>`や**Popover API**に関連する要素などは頻繁に更新されています。アクセシビリティに関わる注意点などが追加で盛り込まれることもあります。定期的にチェックすることをオススメします（もしくはPRを監視しましょう）。おそらく`<search>`も例に漏れなく数ヶ月〜数年にわたって更新が繰り返されるはずです。

英語に関してはDeepLやGPTなどを翻訳機を駆使して読み解いていっていいと思います。初学者にとっては専門用語だらけで目が回るかも知れませんが、こんなのは慣れです。繰り返し読むトレーニングを積めば、少しずつわかってくると思います。

この意味論的ルールは、読んだ人の解釈に依るところが多くなるので、なるべく同じマークアップを生業にする人や有識者と議論をすることをオススメします。ご自身の所属する組織やコミュニティで解釈スレッドを立ち上げるのも面白いかもしれません。筆者もそういった議論は大好きなので、もし機会があれば誘ってください。ここがHTMLの醍醐味のひとつですから。
