---
title: "Invoker Commands APIのイベントのまとめ"
emoji: "🔥"
type: "tech"
topics: ["html", "javascript", "DOM"]
published: true
---

`<dialog>`、Popover、Invoker CommandsがHTML標準で整いつつあります。JavaScriptを書かずにHTMLだけでインタラクティブなUIを作ることができるようになっていっているわけですが、それでもイベントをフックして何かしらの処理を実行したいことはあります。これらのUIのイベントはいくつか種類があるので、それぞれのUIや呼び出し方法に対応するイベントをまとめてみました。

## 前提

### `<dialog>`

`<dialog>`はHTML標準のダイアログUIです。このダイアログUIはモードレスダイアログとモーダルダイアログの2つのモードがあります。モードレスダイアログは`open`属性を付与することで表示されますが、モーダルダイアログは`showModal()`メソッドを呼び出すことで表示されます。つまりこれまではモーダルダイアログを開く手段はJavaScriptを使うしかありませんでした。

```html
<button type="button" id="open-dialog">Open Dialog</button>
<dialog id="dialog">
  <h1>Dialog</h1>
  <button id="close-dialog">Close Dialog</button>
</dialog>
```

```js
document.getElementById("open-dialog").addEventListener("click", () => {
  document.getElementById("dialog").showModal();
});

document.getElementById("close-dialog").addEventListener("click", () => {
  document.getElementById("dialog").close();
});
```

### Popover

一方、JavaScript不要でポップオーバーUIを制御できる仕組みがHTML標準になりました。`popover`属性を持つ要素は初期で非表示の状態になり、`popovertarget`属性をもつ`<button>`要素をクリックすることで表示されます。

```html
<button type="button" popovertarget="popover">Open Popover</button>
<div popover id="popover">...</div>
```

このポップオーバーUIの特徴は「**モードレス**」の展開要素ということです。このポップオーバー以外をクリックしたりフォーカスを移動したりすることができます（そのとき同時にポップオーバーを閉じるかどうかは`popover`属性に`auto`か`manual`を設定することで変更できます）。

しかし、このポップオーバーUIはあくまでも「モードレス」であり、`<dialog>`に`popover`属性を付与してもモーダルダイアログには当然なりません。**PopoverのAPIではモーダルダイアログは作れないということです**。

### Invoker Commands

そして登場したのがInvoker Commandsです。Invoker Commandsは`<button>`要素に`command`属性を付与することで、`<dialog>`やPopoverを制御することができます。`command`属性はいくつかの規定のコマンドを設定することができ、そのなかに`show-modal`というコマンドがあります。このコマンドを`<button>`要素に設定することで、`<dialog>`をモーダルダイアログとして表示することができます。もちろんJavaScriptも不要です。_ﾔｯﾀﾈ!_

```html
<button type="button" command="show-modal" commandfor="dialog">
  Open Dialog
</button>
<dialog id="dialog">
  <h1>Dialog</h1>
  <button command="close" commandfor="dialog">Close Dialog</button>
</dialog>
```

## 対応するイベント

JavaScript不要でポップオーバーUIもモーダルダイアログも実装できるようになりましたが、イベントをフックして何かしらの処理を行いたい場合はたまにあります。そこで、それぞれのUIに対応するイベントをまとめてみます。

| イベント名        | イベントオブジェクト                                                                   | バブリング | `<dialog>`                  | Popover                     | Invoker（呼び出し元）ボタン |
| ----------------- | -------------------------------------------------------------------------------------- | ---------- | --------------------------- | --------------------------- | --------------------------- |
| `command`         | [`CommandEvent`](https://html.spec.whatwg.org/multipage/interaction.html#commandevent) | ❌️ しない | ✅️ Invokerからの呼び出し時 | ✅️ Invokerからの呼び出し時 | ❌️ 自らは発火しない        |
| `beforetoggle`    | [`ToggleEvent`](https://html.spec.whatwg.org/multipage/interaction.html#toggleevent)   | ❌️ しない | ✅️ 表示または非表示の直前  | ✅️ 表示または非表示の直前  | ❌️ 自らは発火しない        |
| `toggle`          | [`ToggleEvent`](https://html.spec.whatwg.org/multipage/interaction.html#toggleevent)   | ❌️ しない | ✅️ 表示または非表示の直後  | ✅️ 表示または非表示の直後  | ❌️ 自らは発火しない        |
| `close`           | [`Event`](https://dom.spec.whatwg.org/#interface-event)                                | ❌️ しない | ✅️ 非表示の直後            | ❌️ 発火しない              | ❌️ 自らは発火しない        |
| `click`（おまけ） | [`PointerEvent`](https://w3c.github.io/pointerevents/#pointerevent-interface)          | ✅️ する   | ❌️ 発火しない              | ❌️ 発火しない              | ✅️ 当然発火する            |

CodeSandboxに[サンプルコード](https://codesandbox.io/p/sandbox/p7fj2m)を書いたので百聞は一見に如かず、コンソールを見ながらイベントの発火を確認してみてください。

### 発火の順番

#### `<dialog>` + `command="show-modal"`

1. `command`
2. `beforetoggle`
3. `click`（Invokerボタン）
4. `toggle`

#### `<dialog>` + `ESC`キーで閉じる

1. `beforetoggle`
2. `toggle`
3. `close`

#### Popover (`command="toggle-popover"`)

1. `command`
2. `beforetoggle`
3. `click`（Invokerボタン）
4. `toggle`

#### Popover（Light Dismiss）

1. `beforetoggle`
2. `toggle`

#### Popover (`command="show-popover"`)

1. `command`
2. `beforetoggle`
3. `click`（Invokerボタン）
4. `toggle`

#### Popover (`command="hide-popover"`)

1. `command`
2. `beforetoggle`
3. `click`（Invokerボタン）
4. `toggle`

### イベントの使い分け

#### `command`イベント

まず、`command`イベントはInvoker(呼び出し元)ボタンに発火するのではなく、**対象のポップオーバーUIやダイアログUIに対して発火します**。そのため`addEventListener`の対象に注意が必要です。

`CommandEvent`は`command`属性に設定された値を`event.command`で、呼び出し元を`event.source`で取得できます。つまり、**複数の呼び出し元があったとしても**これらの情報を使って処理を分岐させることができます。

```js
target.addEventListener("command", (event) => {
  if (event.command === "show-modal") {
    console.log(event.source); // 呼び出し元
  }
});
```

`command`属性には`--`で始まるカスタムコマンドを設定することができます。

```html
<button type="button" command="--custom-command" commandfor="foobar">
  Custom Command
</button>
```

```js
target.addEventListener("command", (event) => {
  if (event.command === "--custom-command") {
    // カスタムコマンドの処理
  }
});
```

フック処理で言えば後述する`beforetoggle`/`toggle`イベントの方が有用なケースが多いので、基本的には`command`イベントは**カスタムコマンドの実装で利用することになる**と思います。

#### `beforetoggle`/`toggle`イベント

::: message
この2つのイベントは、Popoverに対してはHTMLの標準として規定されていますが、**`<dialog>`にはまだ正式には規定されていません。** ChromeとFirefoxには実験的に先行実装されています。Safariでは[ポリフィル](https://www.npmjs.com/package/dialog-toggle-events-polyfill/)を利用することになります。
:::

`beforetoggle`/`toggle`イベントは、Invoker Commands APIとは関係なく、他の方法で呼び出されても発火します。`showModal()`で明示的に表示したり`ESC`キーで閉じたときにも発火します。そのため、UIそのものの状態を監視するのに適しています。どのUIでも共通して発火するので実装漏れを防ぐためにも、基本的にはこのイベントを使うことを最初に検討するといいかもしれません。

`ToggleEvent`は`newState`と`oldState`というプロパティを持ち、それぞれ表示・非表示の状態を表します。

```js
target.addEventListener("beforetoggle", (event) => {
  console.log(event.newState); // "open" もしくは "closed"
});
```

#### `close`イベント

`<dialog>`には`close`イベントがありますが、**Popoverにはないので注意が必要です**。`<dialog>`が閉じられたときに発火します。`<dialog>`の`close()`メソッドを呼び出したときにも発火します。

#### `click`イベント

呼び出し元をクリックしたりエンターキーなどで実行をしたときに発火しますが、このボタンからの起点に限定されるので開閉のフック処理に使うことはあまり適さないと思います。

## まとめ

- フック処理は実装漏れを防ぐためにも、どのUI・どの呼び出しからでも共通で発火する`beforetoggle`/`toggle`イベントを前提に検討することよさそう
- `command`イベントはカスタムコマンドの実装が主な用途
- `close`イベントは`<dialog>`にのみにしか発火しないし、`show`イベントや`open`イベントもないので勘で実装しないように注意
