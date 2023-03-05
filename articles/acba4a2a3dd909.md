---
title: "WAI-ARIA中級編 ロールの決まり方"
emoji: "🧑‍🏫"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["wai-aria", "html", "アクセシビリティ"]
published: true
---

前回の「[WAI-ARIAを学ぶときに整理しておきたいこと](https://zenn.dev/yusukehirao/articles/e3512a58df58fd)」（以下、「前回の記事」と略）やYouTubeで生配信した「[WAI-ARIA勉強会](https://www.youtube.com/watch?v=ZLL0_W5w1vo)」で

- 要素には**暗黙のロール**をもつ
- `role`属性を使うことでロールを**上書き**できる
- 要素によって**上書きできないロール**がある

と説明した。

しかし、上書きできないロールを`role`属性で指定してしまったときに最終的にロールが何になるかの説明をしていなかったし、要素の条件次第によっては上書き可能なはずのロールが**無効化**されたり、ロールが**変化**したりすることに触れていなかった。今回はそこが仕様でどうなっているのか深堀りしてみたいと思う。

::: message alert
ちなみに、これは**必ずしも覚えないといけない仕様ではないと考えます**。開発時にデベロッパーツールのアクセシビリティツリーを見れば最終的に決定されたロールが確認できますし、支援技術を**実際に使って読み上げを確認することが最も重要です**。

この仕様を知ることは、あくまでもどういったロジックで決定されるのかを知ることにより、**ロールの決定を開発者が予想し**開発のスピードを上げるためだったり、テスト時の予想に反する結果が出た場合に**どこのバグなのか（どこが仕様に反しているのか）を見極める**のに役に立つでしょう。

WAI-ARIAは使わないに越したことはありません。しかしどうしても必要になった場合にはかなり強力な技術です。そういったことに直面した場合にだけ、この記事が役に立つと嬉しいです。
:::

## ロールの決定に影響するもの

まず、簡単にロールがどういったものから決定されるのかをざっくりと最初にまとめる。

- 要素の種類（HTMLとしてのセマンティック）
- 属性
- 構造（先祖・子孫関係）

これを**まず念頭に置いて**、仕様を読みすすめると理解がしやすいと思う。ひとまずは**複雑なことを予め覚悟して**読み進めていきたい。

## 暗黙のロールはどう決まるのか

まずは**暗黙のロール**が定義されている[ARIA in HTML](https://www.w3.org/TR/html-aria/#docconformance)を確認していこう。

### 属性による暗黙のロール

仕様を見てみると早速、**属性が関係している**ことがわかるだろう。「_**a** with href_」「_**a** without href_」という記述があるように、`href`属性をもつ`a`要素と、それを持たない`a`要素で判定される暗黙のロールが異なる。

```html
<a href="path/to"><!-- ロールは link --></a>

<a><!-- ロールは generic --></a>
```

`img`要素はさらにややこしい。

- `alt`なし
- `alt`が空
- `alt`にテキストが入っている（つまり空ではない）

の3つ条件があり、「`alt`なし」と「`alt`にテキストが入っている（つまり空ではない）」はロールが`img`になり、`alt`が空はロールが`presentation`になる。

```html
<img src="path/to" /><!-- ロールは img -->

<img src="path/to" alt="" /><!-- ロールは presentation -->

<img src="path/to" alt="何かしらのテキスト" /><!-- ロールは img -->
```

属性そのものの有無だけでなく、**値によってもロールは変わる**ということだ。

値によってロールが変わることについては、[`input`要素](https://www.w3.org/TR/html-aria/#el-input-text)はわかりやすい。`type`属性によってロールが変化する。チェックボックスであれば`checkbox`ロールになるし、ラジオボタンであれば`radio`ロールになる。

ちなみに、

> input type=text or with a missing or invalid type, with no list attribute

という条件の _with a missing or invalid type_ というのは「`type`属性がない」もしくは「`type`属性に無効なデタラメの値がある」の状態をことを指していて、HTMLにも[定義されている](https://html.spec.whatwg.org/multipage/input.html#attr-input-type-keywords:~:text=The%20missing%20value%20default%20and%20the%20invalid%20value%20default%20are%20the%20Text%20state.)とおり、属性がなかったり属性値が無効だった場合は`type=text`と同じ扱いになるため、`type=text`と同じ条件となっている。

### 構造による暗黙のロール

さらに見ていくと、先祖要素によってロールが変わるものがあることもわかる。[`header`](https://www.w3.org/TR/html-aria/#el-header)要素の暗黙のロールは次のように定義されている。

> If not a descendant of an article, aside, main, nav or section element, or an element with role=article, complementary, main, navigation or region then role=banner
>
> Otherwise, role=generic

否定文から始まっているので非常にややこしいんだけど、つまり言い換えると、

- 基本的には`banner`ロール
- ただし祖先に`article`、`aside`、`main`、`nav`、`section`要素、または`role`属性に`article`、`complementary`、`main`、`navigation`、`region`をもつ要素があれば`generic`ロール

ということだ。

```html
<body>
  <header><!-- ロールは banner --></header>
</body>

<body>
  <article>
    <header><!-- ロールは generic --></header>
  </article>
  <div role="article">
    <header><!-- ロールは generic --></header>
  </div>
</body>
```

[`td`](https://www.w3.org/TR/html-aria/#el-td)要素は3パターンある。

> role=cell if the ancestor table element is exposed as a role=table
>
> role=gridcell if the ancestor table element is exposed as a role=grid or treegrid
>
> No corresponding role if the ancestor table element is not exposed as a role=table, grid or treegrid

- 先祖の`table`要素が`table`ロール（暗黙のロールなので`role`属性なしの場合も）だったら`cell`ロール
- 先祖の`table`要素が`grid`ロールもしくは`treegrid`ロールだったら`gridcell`ロール
- 先祖の`table`要素が上記以外だったらロールなし

```html
<table>
  <tr>
    <td><!-- ロールは cell --></td>
  </tr>
</table>

<table role="grid">
  <tr>
    <td><!-- ロールは gridcell --></td>
  </tr>
</table>

<table role="treegrid">
  <tr>
    <td><!-- ロールは gridcell --></td>
  </tr>
</table>

<table role="presentation">
  <tr>
    <td><!-- ロールは なし --></td>
  </tr>
</table>
```

[`th`](https://www.w3.org/TR/html-aria/#el-th)要素も同様の3パターンなのだが、結果が _role=columnheader, rowheader or gridcell_ と曖昧になってるところが現仕様として困りどころだ。

おそらくは「`thead`要素を親にもつ」とか「`scope`属性による」だったりの条件が明記されているのが望ましいのだけど**ARIA in HTML**には言及がない。**WAI-ARIA 1.2**の[`columnheader`](https://www.w3.org/TR/wai-aria-1.2/#columnheader)の説明にはBase Conceptとして、 _&lt;th[scope="col"]&gt; in [HTML]_ と記載があるので、まあそういうことなんだろうとは思うんだけど、ちゃんと厳密に記載して両仕様で足並みを揃えて欲しいところだ。

### アクセシブルな名前の有無による暗黙ロール

[`section`](https://www.w3.org/TR/html-aria/#el-section)要素は、アクセシブルな名前を持てば`region`ロールになり、持たなければロールはない。

```html
<section aria-label="なにかしらのテキスト"><!-- ロールは region --></section>

<section aria-labelledby="heading-id">
  <!-- ロールは region -->
  <h2 id="heading-id">なにかしらのテキスト</h2>
</section>

<section><!-- ロールは なし --></section>
```

---

さて、ここまでは要素がもつ暗黙のロールで**あくまでも基本**。本番はここからだ。

## `role`属性による明示的なロールの決まり方

基本的には`role`属性に明示的にロールを指定すれば、暗黙のロールを**上書き**することになるが、以下の条件では指定が無視される。

### 仕様にないロールは無視される

これは言わずもがな。当然、仕様にないロールは明示しても意味がない。

```html
<div role="my-role"><!-- ロールは generic --></div>
```

### 抽象ロールは無視される

これは「[前回の記事](https://zenn.dev/yusukehirao/articles/e3512a58df58fd#%E6%8A%BD%E8%B1%A1%E3%83%AD%E3%83%BC%E3%83%AB)」でも説明したとおり。ロールには[抽象ロール](https://www.w3.org/TR/wai-aria-1.2/#abstract_roles)に属するものがあるので注意が必要。

```html
<div role="section"><!-- ロールは generic --></div>
```

### 上書きできないロールは無視される

これも「[前回の記事](https://zenn.dev/yusukehirao/articles/e3512a58df58fd#%E4%B8%8A%E6%9B%B8%E3%81%8D%E3%81%A7%E3%81%8D%E3%81%AA%E3%81%84%E3%83%AD%E3%83%BC%E3%83%AB%E3%81%8C%E3%81%82%E3%82%8B)」で言及しているが、より詳しく説明すると、これは「[ARIA in HTML](https://www.w3.org/TR/html-aria/#docconformance)」の _ARIA roles, states and properties which MAY be used_ の列を参照することになる。

ここでも[暗黙のロール](#%E6%9A%97%E9%BB%99%E3%81%AE%E3%83%AD%E3%83%BC%E3%83%AB%E3%81%AF%E3%81%A9%E3%81%86%E6%B1%BA%E3%81%BE%E3%82%8B%E3%81%AE%E3%81%8B)と同様、要素の条件によって使用できるロールが決まっている。

_a with href_ では`button`をはじめとしたいくつかのロールしか認められていないが、_a without href_ では全てのロールが認められている。

```html
<a href="path/to" role="button"><!-- ロールは button --></a>

<a href="path/to" role="alert"><!-- ロールは link (alertは認められない) --></a>

<a role="alert"><!-- ロールは alert --></a>
```

### アクセシブルな名前をもたない場合に無視されるロールがある

[Handling Author Errors](https://w3c.github.io/aria/#document-handling_author-errors)という解説に**WAI-ARIA 1.3**^[まだ草案]で追記された新しい仕様がある。

> Certain landmark roles require names from authors. In situations where an author has not specified names for these landmarks, it is considered an authoring error. The user agent MUST treat such elements as if no role had been provided. If a valid fallback role had been specified, or if the element had an implicit ARIA role, then user agents would continue to expose that role, instead. Instances of such roles are as follows:
>
> - form
> - region

これは、`form`と`region`ロールは明示してもアクセシブルな名前を持たなければ無視されますよ、という仕様だ。

```html
<div role="form"><!-- ロールは generic --></div>

<div role="form" aria-label="何かしらのテキスト"><!-- ロールは form --></div>
```

暗黙の`form`ロールをもっている`from`要素の場合どうなのかというと、現行の**ARIA in HTML**の[`form`](https://www.w3.org/TR/html-aria/#el-form)要素には _A form is not exposed as a landmark region unless it has been provided an accessible name._ という注釈があり、アクセシブルな名前がなければ実質ロールが機能しないことを示している。（ロールが機能しないことと、ロールが無視されることの差はなんなのか曖昧なところは、もっと厳密に決定して欲しいところ）

どのみち、ロールには「**アクセシブルな名前が必須**」となる場合があり、[`form`](https://www.w3.org/TR/wai-aria-1.2/#form)と[`region`](https://www.w3.org/TR/wai-aria-1.2/#region)はどちらもそうなので、仕様に順当に従えば気にしなくてよい細かい仕様かもしれない。

### 必須の構造を満たしていない場合にロールが消失することがある

ロールには[**Required Context Role**](https://www.w3.org/TR/wai-aria-1.2/#scope)の条件をもっているものがあり、たとえば[`listitem`](https://www.w3.org/TR/wai-aria-1.2/#listitem)ロールの場合、`list`か`directory`ロールを親にもっていないといけない。

> For example, an element with role listitem is only meaningful when contained inside (or owned by) an element with role list.

_only meaningful_ という言葉を使っていて、ロールが`generic`になったり*ロールなし*になるとは言及されていないので解釈はブラウザ次第になるが、いくつかのブラウザではロールが消失したり、アクセシビリティツリーに要素そのものが公開されないケース^[ロールを`none`として計算していて結果的に公開されない状態になるケースも確認された。]があるので注意が必要だ。

```html
<ul>
  <li><!-- ロールは listitem --></li>
</ul>

<ul role="generic">
  <li>
    <!-- ロールが消失したり、アクセシビリティツリーに公開されない場合がある -->
  </li>
</ul>

<ul role="tablist">
  <li>
    <!-- ロールが消失したり、アクセシビリティツリーに公開されない場合がある -->
  </li>
</ul>

<div>
  <li>
    <!-- ロールが消失したり、アクセシビリティツリーに公開されない場合がある -->
  </li>
</div>
```

ちなみに、`table`要素に`role="presentation"`を指定することで子孫要素の意味を打ち消すテクニックを使うことがあるけど、その場合この仕様が作用しているとも言えるし、**ARIA in HTML**の暗黙のロールの条件「先祖の`table`要素が`table`、`grid`、`treegrid`以外のロールだったらロールなし」という仕様が作用しているとも言える。このあたり、各仕様が絶妙に辻褄をあわせているところは仕様を読んでいて面白い。

## 指定しても無視される`presentation`

少し特殊なロールとして`presentation`ロールがあるが、このロール固有で無視されるケースがあることが**WAI-ARIA 1.2**の[Presentational Roles Conflict Resolution](https://www.w3.org/TR/wai-aria-1.2/#conflict_resolution_presentation_none)で決められている。

`presentation`ロールはアクセシビリティツリーに要素を**公開しない**ロールで、つまり`role="presentation"`と記述すれば、支援技術からみたら**何もない**状態にすることができる。

`presentation`ロールの使い所としては

```html
<ul role="tablist">
  <li role="presentation"><button role="tab">Apple</button></li>
  <li role="presentation"><button role="tab">Orange</button></li>
  <li role="presentation"><button role="tab">Banana</button></li>
</ul>
```

のように、`tablist`と`tab`の親子関係の間にある要素を**透明にする**ような場合に効果的に利用できる。

:::message
`presentation`と`aria-hidden`はアクセシビリティツリーに要素を非公開にする点において共通しているけど、`presentation`は自身のみ、`aria-hidden`は自身含めた子孫要素すべてを非公開にする違いがあるので、混同しないように注意したい。

また`none`というロールがあるが、これは`presentation`の別名なので、`none`というロールを見かけたら`presentation`と同じと思っていい。
:::

### フォーカス可能であれば`presentation`は無視される

> If an element is focusable, or otherwise interactive, user agents MUST ignore the presentation role and expose the element with its implicit role, in order to ensure that the element is operable.

ごくごくシンプル。フォーカスできたりインタラクティブであれば、明示的に`presentation`を指定しても無視され、暗黙のロールにフォールバックされる。

理由は「ユーザーが操作できるのに、公開されてないのはおかしいでしょ？」ということ。

```html
<a href="path/to" role="presentation"><!-- ロールは link --></a>

<button role="presentation"><!-- ロールは button --></button>

<input type="text" role="presentation" /><!-- ロールは textbox -->

<div tabindex="0" role="presentation"><!-- ロールは generic --></div>
```

ただし**WAI-ARIA**はHTMLのみに言及できる仕様ではないため、HTMLの各要素のどれがフォーカス可能またはインタラクティブ要素かどうかには明記していない（_interactive_ = [コンテンツ・モデルがインタラクティブコンテンツの要素](https://html.spec.whatwg.org/multipage/dom.html#interactive-content-2)とは明言していない）。ブラウザがどう解釈してどう実装されるかはブラウザ次第になる。この曖昧さに関しては[Issue](https://github.com/w3c/aria/issues/1192)が作られていて、まだクローズされていない。

また、サンプルコードに記載はしたものの、**ARIA in HTML**では`href`をもつ`a`要素も、`button`要素も、`type="text"`をもつ`input`要素も、そもそも`presentation`で上書きできないことになっているので、その仕様に則っても無視されて暗黙のロールにフォールバックされるのは同じである。ここも**ARIA in HTML**と**WAI-ARIA**で矛盾が発生しないようになっている。

### 継承された`presentation`が無視される

:::message
この節は解釈がかなり曖昧です。
:::

前提の知識として**WAI-ARIA**の[**Required Owned Element**](https://www.w3.org/TR/wai-aria-1.2/#mustContain)という仕様がある。これは、たとえば「`list`ロールの子は`listitem`でなければならない」といった構造上のルールだ。

```html
<ul>
  <!-- ul のロールは list なので -->
  <li>
    <!-- 子となる li のロールは listitem でなければならない。ここでは違反していない。 -->
  </li>
</ul>
```

そして、この**Required Owned Element**において`presentation`ロールは**継承**される性質を持つ。

```html
<ul role="presentation">
  <!--
    ul のロールは list で元々 Required Owned Element をもつ
    この ul 自身のロールは presentation
  -->
  <li>
    <!--
      親が presentation となった Required Owned Element なので
      ロールは presentation
    -->
  </li>
</ul>
```

そして、その`presentation`が無視される条件がある。

> If a required owned element has an explicit non-presentational role, user agents MUST ignore an inherited presentational role and expose the element with its explicit role. If the action of exposing the explicit role causes the accessibility tree to be malformed, the expected results are undefined.

- **Required Owned Element**の要素
- `presentation`を継承している
- 明示的なロール（`presentation`でない）をもつ

の3つの条件が揃えば、継承された`presentation`は無視されて、明示的なロールが適用される。ただし、これの結果、構造が不正になる場合はロールが未定義になってしまう。

う〜ん、この仕様は何度読んでも難しい。そもそも`presentation`を継承しているということは**Required Owned Element**の要素ということになるし、親が`presentation`の状態ということは**Required Context Role**から構造の不正にになるので、どう考えても未定義にしかなりえないと思うんだけど…何か重要な仕様を見逃してるのかもしれない…。※ご指摘待ってます。

### グローバルWAI-ARIAプロパティをもつと`presentation`は無視される

> If an element has global WAI-ARIA states or properties, user agents MUST ignore the presentation role and expose the element with its implicit role.

要素が[グローバルなWAI-ARIAプロパティ](https://www.w3.org/TR/wai-aria-1.2/#global_states)をもつのなら、`presentation`ロールは無視されて要素は公開される。

```html
<div role="presentation" aria-busy="false"><!-- ロールは generic --></div>

<div role="presentation" aria-live="polite"><!-- ロールは generic --></div>
```

> However, if an element has only non-global, role-specific WAI-ARIA states or properties, the element MUST NOT be exposed unless the presentational role is inherited and an explicit non-presentational role is applied.

説明には続いて上記のように書かれている。逆に非グローバルでロール固有のプロパティ**のみ**を持つ場合、そのまま`presentation`ロールとなる、ということだが、グローバルプロパティもってないんだからそりゃそうでしょ、といった説明だ^[仕様ってへんなところでやけに詳しく冗長に書かれていることがある。]。

## Presentational Childrenの要素はロールが無意味

ロールには**Children Presentational**という性質をもち、いくつかのロールはこれが`true`となっている。

この[**Presentational Children**](https://www.w3.org/TR/wai-aria-1.2/#childrenArePresentational)がどういうものかというと、たとえば`button`ロールの**Children Presentational**の性質は`true`で、このロールの**子孫要素はアクセシビリティツリーに公開されない**、というもの。一見不思議に思うかもしれないが、`button`ロールは**アクセシブルな名前**をコンテンツから計算するので、中身自体がツリーから消えても問題ない。

```html
<button>何かしらのテキスト</button>
<!--
  button要素の計算されたアクセシビリティプロパティは
  Role: button
  name: 何かしらのテキスト （👈 アクセシブルな名前をしっかり持っている）
  focusable: true
-->
```

具体的にこれの何がロールに影響を与えるかというと、`button`要素にテキストだけでなく要素を入れるとわかりやすい。

```html
<button>
  <img
    src="path/to"
    alt="何かしらのテキスト"
  /><!-- img要素はアクセシビリティツリーに公開されない -->
</button>
<!--
  ただし、button要素の計算されたアクセシビリティプロパティは
  Role: button
  name: 何かしらのテキスト （👈 計算されてalt属性から抽出される）
  focusable: true
-->
```

このように`img`要素だろうがなんだろうが、`button`ロールをもつ要素に何を入れても公開されない。

つまり、`button`要素の中で明示的にロールを指定したりしても、機能しないということだ。

```html
<button>
  <div role="alert">メッセージ</div>
  <!-- div要素はアクセシビリティツリーに公開されない -->
</button>
<!--
  ただし、button要素の計算されたアクセシビリティプロパティは
  Role: button
  name: メッセージ （👈 計算されてdivのコンテンツから抽出される）
  focusable: true
-->
```

:::message

ただ、この仕様はあくまでも**SHOULD NOT**として定義されているので、いくつかブラウザでは**Presentational Children**をアクセシビリティツリー公開していたりする。

実際のところこの仕様は際どくて、簡単に要素を非アクセシブルにしかねない。

```html
<div role="img">
  <h3>何かしらの見出し</h3>
  <p>何かしらのテキスト</p>
  <a href="path/to">何かしらのリンク</a>
</div>
```

上記のコードは、仕様に則れば、`h3`要素、`p`要素はアクセシビリティツリーに公開されない。開発者のちょっとしたミスで起きかねないこの状態を、いくつかのブラウザでは採用していないのは、実際のところ納得はいく。今まで世界中でたくさんの`role`の間違った使い方があったことを考えると、この仕様を採用すると今までアクセシブルだったページが突然使い物にならなくなる可能性もゼロじゃない。悩ましい互換性の問題をはらんでいるのだ。

ちなみに`a`要素はフォーカス可能要素なのでアクセシビリティツリーに強制的に公開される性質がある。

:::

## ブラウザと支援技術の確認を忘れずに

以上が把握している限りの**ロールの決まり方**なのだけど、これは**あくまでも仕様の話**。実際はブラウザの実装次第な部分は多く、さらにそのブラウザと支援技術との組み合わせでも挙動は変わるので、そのマークアップが期待通りかどうかは実際に動作させてみて確認するしかない。

:::message

### ブラウザが独自に提供するロール

「ブラウザの実装次第」と言えば、この説明を忘れていた。ブラウザはWAI-ARIAで**定義されていないロール**を提供している。デベロッパーツールでアクセシビリティツリーを見ている人は気づいている人も多いかもしれない。

たとえばChromeでは、`main`に内包される`header`のロールは`HeaderAsNonLandmark`となっている。**ARIA in HTML**では`generic`にも関わらず、だ。これは筆者の推測でしかないのだが[**ARIA in HTML**の過去のスナップショット](https://www.w3.org/standards/history/html-aria)を見るに、かつて一時期は`main`に内包されるロールは「対応するロールなし（_no corresponding role_）」という**未定義**ということになっており、Chromeはどうやら未定義に対しては独自のロールを定義するようだ。その名残りで`HeaderAsNonLandmark`があるように思うので、しばらくしたら**ARIA in HTML**に従って`generic`に変わることが予想される。Chromeの場合、ロール名が大文字で始まっているものは独自のロールとのようで、`section`要素（アクセシブルな名前がない）のロールとなっている`Section`ロールは、これは抽象ロールの[`section`](https://www.w3.org/TR/wai-aria-1.2/#section)ではなく、Chrome独自のものだと考えたほうが良い。**ARIA in HTML**の方向性としては「対応するロールなし（_no corresponding role_）」は徐々に何かしらのロールを割り当てることになってるので、今後はこの独自のロールもそれに置き換えられていくはずだ。

ちなみにFirefoxのアクセシビリティツリーはなぜかほとんどのロールを上位の抽象ロールで表示しており（たとえば`main`も`banner`も`navigation`も`landmark`となってしまっている）、現段階ではあまり使い物にならない。

Safariは**WAI-ARIA**に存在するロールのみで提供している。
:::

## 最後に

なぜ、この記事を書いたかというと、もちろんWAI-ARIAの理解を深めるためもそうだけど、[**Markuplint**](https://markuplint.dev/)の[WAI-ARIAのルール](https://markuplint.dev/rules/wai-aria)を作る上でこのあたりの調査をしたからだ。まだ`v2.x`では中途半端な実装しかできていなかったが、近日リリース予定の`v3.0.0`ではこのあたりを**かなり強化した**。すでに[`v3.0.0-rc`](https://github.com/markuplint/markuplint/releases/tag/v3.0.0-rc.3)はプレリリースしていて試すことができるし、また、興味があればどういった実装をしたのか[ソースコード](https://github.com/markuplint/markuplint/blob/6cb4f1656d4cd7554891006181c1f79cea4d9bfa/packages/%40markuplint/ml-spec/src/dom-traverse/get-computed-role.ts)見てもらえると嬉しい。この記事に書いたことのほとんどがJavaScriptで書かれている。

つまり、この記事の変なところがあった場合にご指摘いただけるとMarkuplintも改善される、という寸法です。ご協力よろしくお願いします！
