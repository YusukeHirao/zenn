---
title: Popover API - JavaScript不要、HTMLのみでポップオーバーUI
emoji: 🎈
type: tech
topics: [HTML, JavaScript, UI]
published: true
---

HTML Standardに`popover`属性をはじめとした[Popover API](https://momdo.github.io/html/popover.html)が正式にマージされました。[Open UI](https://open-ui.org/)によって提案されていた^[[Popover API (Explainer)](https://open-ui.org/components/popover.research.explainer)]APIで、名前がPopoverなのかPopupなのか紆余曲折の末、やっとHTML Standardとなります。

現段階で実装されているブラウザは少ないですが、簡易サンプルを作ったので体験しながら読んでいただくといいかもしれません。

:::message

2023年2月現在ではGoogle Chrome Canaryのみで動作確認できています。

:::

:::details 簡易サンプル

@[codesandbox](https://codesandbox.io/embed/popover-api-f9fxex?fontsize=14&hidenavigation=1&theme=dark)

:::

## 4つの属性

まずはHTMLの属性が新しく追加されているところに注目しましょう。

- [`popover`属性](https://momdo.github.io/html/popover.html#attr-popover)
- [ポップオーバーターゲット属性](https://momdo.github.io/html/popover.html#the-popover-target-attributes)
  - [`popovertoggletarget`属性](https://momdo.github.io/html/popover.html#attr-popover-toggle-target)
  - [`popoverhidetarget`属性](https://momdo.github.io/html/popover.html#attr-popover-hide-target)
  - [`popovershowtarget`属性](https://momdo.github.io/html/popover.html#attr-popover-show-target)

以上の4つが追加されました。`popover`属性はポップオーバーする要素自体に。ポップオーバーターゲット属性はそれを展開する**ボタン**^[`button`要素と、`type=submit`・`type=button`・`type=reset`・`type=image`をもつ`input`要素]に対して設定します。

```html
<div popover="auto" id="p1">ポップオーバーUI</div>

<button type="button" popovertoggletarget="p1">開閉ボタン</button>
```

たったこれだけで、ポップオーバーUIが動作します。**JavaScriptは不要**です。ポップオーバーターゲット属性のidがマッチしていれば、どんなに離れた場所にある要素でも動作するので、遠隔操作版の`<details>`・`<summary>`と言ってもいいもしれません。

`popover`属性が付与された要素は、デフォルトで非表示（`display: none`状態）となり、展開された場合には[トップレイヤー](https://fullscreen.spec.whatwg.org/#top-layer)に表示されます。トップレイヤーとは、簡単に言うと、`z-index`をどんなに大きな数にしても超えられない表示レイヤーで、`<dialog>`もここに表示されます。

ポップオーバーターゲット属性は3種類あり、それぞれ目的に合わせて柔軟に使い分けることができます。

- `popovertoggletarget`属性: 閉じていれば開き、開いていれば閉じる
- `popoverhidetarget`属性: 閉じるのみ
- `popovershowtarget`属性: 開くのみ

著者の感想としては「本気でJavaScript不要にしたかったんだな^[実際、提案の[**ゴール**](https://open-ui.org/components/popover.research.explainer#goals)には “Avoid the need for any Javascript for most common cases.”（ほとんどの場合、Javascriptは必要ありません。）と記載されています]」ということが伝わってきます。

## 簡易非表示（Light Dismiss）

今回新たに[**簡易非表示**](https://momdo.github.io/html/popover.html#popover-light-dismiss)（英語原版では**Light Dismiss**）^[簡易非表示という言葉は、日本語翻訳のページに合わせています]という概念が登場しました。

そんなに難しいものではなく

- ESCキーで閉じる
- ポップオーバーUIの外側をクリックすると閉じる

という単純な機能ですね。外側をクリックは一般的によく採用されてきたパターンですし、ESCキーも同様にキーボード操作でも容易に閉じることができるでアクセシビリティも担保できています。

この**簡易非表示**は、`popover="auto"`というように属性値に`auto`を指定したときのみに自動的に有効になります。これもJavaScriptを使わずに有効になるのはとても有用です。

## 複数表示と自動非表示

`popover`属性の値は`auto`と`manual`を受け取りますが、この2つの違いは先に説明した簡易非表示以外もあります。

`auto`は**開いたときに他のポップオーバーUIを閉じます**。一方、`manual`は**他のポップオーバーを閉じません**。表にまとめると、

| 値       | 複数表示 | 他のポップオーバーUIを | 簡易非表示 |
| -------- | -------- | ---------------------- | ---------- |
| `auto`   | できない | 閉じる                 | 有効       |
| `manual` | できる   | 閉じない               | 無効       |

ということになります。目的合わせて使い分けるといいでしょう。

## JavaScriptで利用するには

ここまでJavaScript不要と紹介していますが、使えないわけではありません。DOM APIとしてもきちんと操作可能なAPIを提供しているので、これを利用してより目的にあった実装をすることも可能です。

### メソッド

3つのメソッドを使うことができます。

```html
<div popover="auto" id="p1">ポップオーバーUI</div>
```

```js
const element = document.querySelector("#p1");
element.showPopover();
element.hidePopover();
element.togglePopover();

// togglePopoverはfocusPreviousElementオプションを受け取りますが、詳細は調査中です
element.togglePopover(true);
```

### イベント

イベントは2種類です。イベントオブジェクトは`newState`と`oldState`というプロパティを持ちます。

```js
element.addEventListener("beforeToggle", (ev) => {
  console.log(ev.newState); // "open" もしくは "closed"
  console.log(ev.oldState); // "open" もしくは "closed"

  // beforeToggleイベント内ではpreventDefaultで開閉を抑制することができます。
  ev.preventDefault();
});

element.addEventListener("toggle", (ev) => {
  console.log(ev.newState); // "open" もしくは "closed"
  console.log(ev.oldState); // "open" もしくは "closed"
});
```

`popover="manual"`は簡易非表示を無効にしていることなどから、JavaScriptで独自の実装をするために設けられたものとも考えることができます。うまく扱っていきたいところです。

## アクセシビリティ

:::message
HTML Standardに記載されている情報ではなく、[Popover API (Explainer)](https://open-ui.org/components/popover.research.explainer)による情報を元に、著者による調査と解釈も含めて解説します。
:::

気になるアクセシビリティオブジェクトのマッピングですが、セマンティクスへの影響（つまりroleへの上書き）は基本的にないことになっています。しかし、`popover`属性を付与した時点で非表示（`display:none`）になるので、その時点ではアクセシビリティツリーから消えることを意味するので、その点は留意が必要です。表示された時点でセマンティクスが変化することはありません。

一方、ポップオーバーターゲット属性を付与したボタン側には`expanded`ステートがアクセシビリティオブジェクトに公開されます^[2023年2月現在 Chrome Canaryにて観測]。つまり`aria-expanded`を付与させたのと同様になるということですね。ここも自動的に制御されるのでJavaScript不要で、かゆいところに手が届いています。

### 読み上げに関する工夫

上記のようにセマンティクスの影響がないことや、あくまで`expanded`での関係になるので、モーダルダイアログと異なりフォーカスが移動することはありません。また、読み上げの順番に影響を与えることもありません。この場合にはいくつかの工夫が必要となるでしょう。

#### DOMの順番で工夫する

HTMLの記述の順番をボタン→ポップオーバーUIのとすることで、スクリーンリーダーのカーソルが自然と開いた要素に移動できるようにします。

```html
<button type="button" popovertoggletarget="p1">開閉ボタン</button>
<div popover="auto" id="p1">ポップオーバーUI</div>
```

#### ライブリージョンを利用する

ポップオーバーUIが表示されたタイミングで読み上げます。ただし、再度読みげたり詳細な読み上げ操作をするには、その要素に移動しないといけないので、場合によってはあまり上手い方法とは言えません。

```html
<div popover="auto" id="p1" aria-live="polite">ポップオーバーUI</div>

<button type="button" popovertoggletarget="p1">開閉ボタン</button>
```

他にも、`autofocus`属性を使って強制的にフォーカス移動させる方法も考えることができますが^[実際、`autofocus`属性の仕様もポップオーバーと併用できる点が追記されています。[差分](https://whatpr.org/html/8717/0f1d234...3a3a9ed/interaction.html)]、トグルボタンに戻ってくる手段がJavaScriptを使う以外におそらくないため有効な方法とは言えません。そもそも`dialog`要素の[モードレスダイアログでフォーカスを移動させることの問題点](https://github.com/whatwg/html/issues/7707)が指摘されています。基本的には**フォーカスの強制移動は避けて**、それでもどうしても必要な場合は十分にアクセシビリティを考慮して採用するべきでしょう。

## CSS擬似クラス

CSSでは擬似クラスが2つ適用されます。

- `:open`: 表示時に適用されるクラス
- `:closed`: 非表示時に適用されるクラス

基本的には非表示状態は`display: none`となるので積極的な活用方法はないように思いますが、`transition`などをうまく利用したアニメーションを実装する場合には必要になるかも知れません。

## `dialog`要素での利用

結論から言うと、モーダルダイアログに利用することは**できません**。なぜかと言うと、ポップオーバーターゲット属性からJavaScript（DOM API）を使用せずに呼び出す場合、`showModal`に相当する呼び出しができないからです。

```html
<dialog popover="auto" id="m1">ダイアログ</dialog>

<!-- showModalが呼び出されるわけではない -->
<button type="button" popovertoggletarget="m1">開閉ボタン</button>
```

さらに、明確に併用できないことがDOM APIで厳密に規定されました。`popover`属性をもつ`dialog`要素が`showModal`を呼び出すと`DOMException`エラーとなります^[2023年2月現在 Chrome Canaryでは未実装]。

```html
<dialog popover="auto" id="m1">ダイアログ</dialog>
```

```js
const dialog = document.querySelector("#m1");

dialog.showModal();
// => DOMException例外が投げられます。
```

Popover APIはあくまでもモードレスなUIにのみ利用できる、という位置づけとなるようです。設計時、実装時ともに気をつけないといけません。

## ブラウザ実装状況

2023年2月時点でGoogle Chrome Canaryで動作確認できます。

まだ、[MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes)や[Can I Use](https://caniuse.com/?search=popover)にも情報が作られていないので、徐々に公開されるのを待ちましょう。
