---
title: Popover API - JavaScript不要、HTMLのみでポップオーバーUI
emoji: 🎈
type: tech
topics: [HTML, JavaScript, UI]
published: true
---

:::details 更新履歴

- 2023-09-18 [3/10〜9/18までの更新](https://github.com/whatwg/html/issues?q=label%3A%22topic%3A+popover%22+is%3Aclosed)に対応する修正
- 2023-03-10 [属性の破壊的変更](https://github.com/whatwg/html/pull/8962)による修正
- 2023-02-19 初稿

[Git履歴](https://github.com/YusukeHirao/zenn/commits/main/articles/popover-api-and-attributes.md)
:::

HTML Standardに`popover`属性をはじめとした[Popover API](https://html.spec.whatwg.org/multipage/popover.html)が正式にマージされました。[Open UI](https://open-ui.org/)によって提案されていた^[[Popover API (Explainer)](https://open-ui.org/components/popover.research.explainer)]APIで、名前がPopoverなのかPopupなのか紆余曲折の末、やっとHTML Standardとなります。

現段階で実装されているブラウザは少ないですが、簡易サンプルを作ったので体験しながら読んでいただくといいかもしれません。

:::message

- ~~2023年2月19日時点でGoogle Chrome Canaryのみで動作確認できていました。~~
- ~~2023年3月現在では仕様変更の影響か動作確認できるブラウザがありません。~~
- 2023年9月現在でChromeとEdgeが対応しています。Safariは`v17`から対応予定です。([HTMLElement API: popover | Can I use](https://caniuse.com/mdn-api_htmlelement_popover))

:::

:::details CodeSandboxの簡易サンプル

@[codesandbox](https://codesandbox.io/embed/popover-api-f9fxex?fontsize=14&hidenavigation=1&theme=dark)

:::

## 3つの属性

まずはHTMLの属性が新しく追加されているところに注目しましょう。

- [`popover`属性](https://html.spec.whatwg.org/multipage/popover.html#attr-popover)
- [`popovertarget`属性](https://html.spec.whatwg.org/multipage/popover.html#attr-popovertarget)
- [`popovertargetaction`属性](https://html.spec.whatwg.org/multipage/popover.html#attr-popovertargetaction)

以上の3つが追加されました。`popover`属性はポップオーバーする要素自体に。`popovertarget`属性と`popovertargetaction`属性はそれを展開する**ボタン**^[`button`要素と、`type=submit`・`type=button`・`type=reset`・`type=image`をもつ`input`要素]に対して設定します。つまり、`a`要素や`div`要素には設定できません。これはアクセシビリティの観点から革新的と言えます。**ついにdivボタンが消える…！**

```html
<button type="button" popovertarget="p1">開閉ボタン</button>
<div popover id="p1">ポップオーバーUI</div>
```

たったこれだけで、ポップオーバーUIが動作します。**JavaScriptは不要**です。`popovertarget`属性のidがマッチしていれば、`<details>`・`<summary>`と同様にJavaScriptからメソッドを呼び出すことなく実装できます。

`popover`属性が付与された要素は、デフォルトで非表示（`display: none`状態）となり、展開された場合には[トップレイヤー](https://fullscreen.spec.whatwg.org/#top-layer)に表示されます。トップレイヤーとは、簡単に言うと、`z-index`をどんなに大きな数にしても超えられない表示レイヤーで、`<dialog>`もここに表示されます。`popover`属性の値は「空」か`auto`か`manual`で、デフォルトは`auto`です。「空」は`auto`と同等なので、値なしで`<div popover>`と書くこともできます。

ポップオーバーUIを制御するトリガー側の属性は2種類あります。

- `popovertarget`属性: 対象のポップオーバーUIの**ID**を指定します。
- `popovertargetaction`属性: アクションを種類を指定します。`toggle`、`show`、`hide`のいずれかです。デフォルトは`toggle`なので、この属性は省略することが出来ます。

筆者の感想としては「**本気でJavaScript不要にしたかったんだな**^[実際、提案の[**ゴール**](https://open-ui.org/components/popover.research.explainer#goals)には “Avoid the need for any Javascript for most common cases.”（ほとんどの場合、Javascriptは必要ありません。）と記載されています]」ということが伝わってきます。

## 簡易非表示（Light Dismiss）

今回新たに[**簡易非表示**](https://html.spec.whatwg.org/multipage/popover.html#popover-light-dismiss)（英語原版では**Light Dismiss**）^[簡易非表示という言葉は、日本語翻訳のページに合わせています]という概念が登場しました。

そんなに難しいものではなく

- ESCキーで閉じる
- ポップオーバーUIの外側をクリックすると閉じる

という単純な機能ですね。外側をクリックは一般的によく採用されてきたパターンですし、ESCキーも同様にキーボード操作でも容易に閉じることができるでアクセシビリティも担保できています。

この**簡易非表示（Light Dismiss）** は、`popover`属性が`auto`のときに有効になります（つまり値なしでもOK）。これもJavaScriptを使わずに有効になるのはとても有用です。

## auto/manual

`popover`属性の値は`auto`と`manual`を受け取りますが、この2つの違いは先に説明した簡易非表示以外もあります。

`auto`は**開いたときに他のポップオーバーUIを閉じます**。一方、`manual`は**他のポップオーバーを閉じません**。表にまとめると、

| 値       | 複数表示 | 他のポップオーバーUIを | 簡易非表示（Light Dismiss） |
| -------- | -------- | ---------------------- | --------------------------- |
| `auto`   | できない | 閉じる                 | 有効                        |
| `manual` | できる   | 閉じない               | 無効                        |

ということになります。目的合わせて使い分けるといいでしょう。

## JavaScriptで利用するには

ここまでJavaScript不要と紹介していますが、使えないわけではありません。DOM APIとしてもきちんと操作可能なAPIを提供しているので、これを利用してより目的にあった実装をすることも可能です。

### メソッド

3つのメソッドを使うことができます。

```html
<div popover id="p1">ポップオーバーUI</div>
```

```js
const element = document.querySelector("#p1");

// ポップオーバーUIを表示する
element.showPopover();

// ポップオーバーUIを非表示にする
element.hidePopover();

// 表示・非表示を切り替える（戻り値は開閉の結果）
const isShow = element.togglePopover();

// 強制的に表示・非表示を切り替える
element.togglePopover(true);
```

`popover`属性を持たない要素を`showPopover`などで呼び出すと例外（[`NotSupportedError`](https://webidl.spec.whatwg.org/#notsupportederror)）を投げます。

### イベント

イベントは`beforetoggle`と`toggle`の2種類です。イベントオブジェクトは[`ToggleEvent`](https://html.spec.whatwg.org/multipage/interaction.html#toggleevent)インターフェイスで`newState`と`oldState`というプロパティを持ちます。

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
HTML Standardに記載されている情報だけではなく、[Popover API (Explainer)](https://open-ui.org/components/popover.research.explainer)による情報を元に、筆者による調査と解釈も含めて解説します。
:::

### アクセシビリティオブジェクトのマッピング

気になるアクセシビリティオブジェクトのマッピングですが、以下のような変化が起こります^[2023年9月現在Chromeにて観測]。

| 要素                          | ロール                                                                                                                  | ステート              |
| ----------------------------- | ----------------------------------------------------------------------------------------------------------------------- | --------------------- |
| `popover`属性をもつ要素       | `group`                                                                                                                 |                       |
| `popovertarget`属性をもつ要素 | `button`（変化なし^[暗黙的に`button`ロールになる`button`要素と`input`要素にしか`popovertarget`属性は許可されないため]） | `expanded=false/true` |

また`popover`属性を付与した時点で非表示（`display:none`）になりアクセシビリティツリーから消えることを意味するので、その点は留意が必要です。

### 読み上げに関する工夫

以下のように言及されています。

> Whenever possible ensure the popover element is placed immediately after its triggering element in the DOM. Doing so will help ensure that the popover is exposed in a logical programmatic reading order for users of assistive technology, such as screen readers.

これは、可能な限りポップオーバー要素を、それをトリガーする要素の直後にDOM内に配置するようにすることを推奨しています。このようにすることで、スクリーンリーダーなどの支援技術を使用するユーザーにとって、論理的なプログラム読み取り順序で公開されることを確保するのに役立ちます。

```html
<button type="button" popovertarget="p1">開閉ボタン</button>
<div popover id="p1">ポップオーバーUI</div>
```

### フォーカスの移動

基本的にトリガー要素からポップオーバー要素にフォーカスは移動しません。しかしポップオーバーの要素内に`autofocus`属性をもつフォーカス可能要素があれば移動します。

```html
<button type="button" popovertarget="p1">開閉ボタン</button>
<div popover id="p1">
  <button autofocus>開くと同時にフォーカスが当たる要素</button>
</div>
```

この場合、ポップオーバーが閉じられたとき、フォーカスがポップオーバー要素の内側にある場合は**トリガー要素にフォーカスが戻ります**（ポップオーバー要素の外側の場合はそのままフォーカスは移動しません）。

ただし`popover="manual"`の場合は、ポップオーバーが閉じてもフォーカスは戻りません^[ただし、ポップオーバーがアクセシビリティツリー上から消えるのでフォーカスは浮いた状態となります。]。これは、JavaScriptで独自の実装をすることを想定しているためと考えられます。

## CSS擬似クラス

Popover APIのページには擬似クラスについて言及はありませんが、[デフォルトCSS](https://html.spec.whatwg.org/multipage/rendering.html#flow-content-3)の定義と、[擬似クラス](https://html.spec.whatwg.org/multipage/semantics-other.html#selector-popover-open)のページで仕様を確認することができます。

CSSでは擬似クラスが1つ適用されます。

- `:popover-open`: 表示時に適用されるクラス

## `dialog`要素での利用

結論から言うと、モーダルダイアログに利用することは**できません**。`popovertarget`属性は`showModal`メソッドに相当する呼び出しができないからです。

```html
<!-- showModalが呼び出されるわけではない -->
<button type="button" popovertarget="m1">開閉ボタン</button>
<dialog popover id="m1">ダイアログ</dialog>
```

また`popover`属性をもち、既に`dialog`要素がポップオーバー要素として開いている状態で`showModal`を呼び出すと`DOMException`エラーとなります^[[The showModal() method steps](https://html.spec.whatwg.org/multipage/interactive-elements.html#dom-dialog-showmodal)のステップ4参照。2023年9月現在Chromeでは未実装]。

Popover APIは簡易非表示（Light Dismiss）が便利なため、モーダルダイアログでも利用したいところですが`showModal`メソッドが呼び出せないため、簡易非表示（Light Dismiss）が有効になりません。実装の際は注意が必要です。

## `id`属性を使わない実装

`Element.popoverTargetElement`にポップオーバー要素のノードを渡すことで、`id`属性を使わずに実装することができます。

```html
<button type="button">開閉ボタン</button>
<div popover>ポップオーバーUI</div>
```

```js
const button = document.querySelector("button");
button.popoverTargetElement = button.nextElementSibling;
```

これにより、不要な`id`値を生成することなく実装することができます。Reactなどの実装でも`useRef`を使って同様のことができるでしょう。

## ブラウザ実装状況

2023年9月現在でChromeとEdgeが対応しています。Safariは`v17`から対応予定です。([HTMLElement API: popover | Can I use](https://caniuse.com/mdn-api_htmlelement_popover))

## ポリフィル

[Popover Attribute Polyfill](https://github.com/oddbird/popover-polyfill)が有志によって提供されています。SafariとFirefox向けに利用してみるとよいでしょう。

```sh
npm install @oddbird/popover-polyfill
```
