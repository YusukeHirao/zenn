---
title: フロントエンドid属性管理戦略
emoji: 🆔
type: tech
topics: [frontend, html, アクセシビリティ, markuplint]
published: true
---

アクセシビリティのチェックなどを行っているとよく発見される問題に**IDの重複**がある。HTMLでは`id`属性はグローバル属性でありすべての要素に指定できるが、**その値は一意である必要があり、ドキュメント内において重複があってはならないことになっている**。

ただ実際に実装してみたり開発経験のある人ならご存知のとおり、滅多なことでこの重複が問題になることは少ない。HTMLのパースは中断することなくブラウザは要素を描画するし、CSSのセレクタは期待通り要素を特定してスタイルを適用する。なのでこの重複に対してそこまで気を使ってこなかった人も多いだろうし、先のアクセシビリティチェックでよく発見されるのもそういった背景があるのだろうと思う。

しかし表面的に問題が起きていなくとも、実際には重大な構文エラーであり、潜在的に多くの問題を抱えている。IDの重複が引き起こす問題は単純で、そのIDを参照する処理は**ひとつめの要素しか参照しない**。つまり重複している2つ目以降の要素は無視されてしまうのだ^[[セレクタ](https://www.w3.org/TR/selectors-4/#id-selectors)は仕様上この限りではなく、CSSのIDセレクタは重複していても全てのセレクタが適用される。DOM APIの`querySelectorAll`でもIDセレクタはマッチした分の要素を全てコレクションに含める。]。

```html
<div id="foo">Foo1</div>
<div id="foo">Foo2</div>

<script>
  const el = document.getElementById("foo");
  console.log(el.textContent); // => "Foo1"
</script>
```

<!-- prettier-ignore-start -->
```html
<label for="bar">Bar</label>
<input id="bar" /><!-- アクセシブルな名前: "Bar" -->
<select id="bar"></select><!-- アクセシブルな名前: （なし） -->
<textarea id="bar"></textarea><!-- アクセシブルな名前: （なし） -->
```
<!-- prettier-ignore-end -->

特に`label`要素の`for`属性やIDを参照するARIAプロパティでは正確に要素の関係をつくれないので、アクセシブルな名前が空になりスクリーンリーダーで**何も読み上げない**事象が発生したりする。WCAGでは[達成基準 4.1.1 構文解析（適合レベルA）](https://waic.jp/docs/UNDERSTANDING-WCAG20/ensure-compat-parses.html)がこれに関連し、[達成方法 H93: ウェブページの id 属性値が一意的 (ユニーク) であるようにする](https://waic.jp/docs/WCAG-TECHS/H93)や[F77: 達成基準 4.1.1 の失敗例 － ID 型の値が重複している](https://waic.jp/docs/WCAG-TECHS/F77)のように具体的にIDの重複について言及している。Axeでも[ARIAおよびラベルに使用されているIDは一意でなければなりません](https://dequeuniversity.com/rules/axe/4.3/duplicate-id-aria?lang=ja)といったルールが設けられている。

とにかく「**アイディーチョウフク、ダメ、ゼッタイ**」なので、開発において何かしらの工夫が必要になる。ほとんどのウェブ開発においてコンポーネントやHTMLのパーツはいくつかのファイルに分割されており、どこでどんな名前のIDが書かれていて、どこでインポートやインクルードされて結合しているのか把握するのは困難だ。最終的なレンダリングHTMLをチェックすれば確認はできるのだが、果たしてそのタイミング以外で防いだり確認する方法は無いのか戦略を考えたい。

## 動的にIDを生成する

まず簡単な戦略としては動的にIDを生成することだ。HTML要素をコンポーネントの単位で管理している昨今の開発手法において最も手軽な方法と言える。これはIDのハードコーディングを避けることによって、IDの命名や他と重複しているかどうかの確認や管理をする手間が省ける。[@xrxoxcxox](https://twitter.com/xrxoxcxox)さんのこの記事にあるような手法で、ID属性とその関連を示す属性（`for`属性やARIAプロパティ）などで利用しやすい。

https://qiita.com/xrxoxcxox/items/2c498537292c6388cb80

## IDはどんなケースで利用されるのか

さて動的に生成すれば管理の手間が省けて重複を避けることができて問題解決！と思いきやそうは問屋が卸さない。なぜかというと、IDが**固定値でなければないない場合が存在する**からだ。

ではまずIDが利用されるケースを洗い出しみよう。

- 属性からの参照
  - [`itemref`属性](https://html.spec.whatwg.org/multipage/microdata.html#attr-itemref)（グローバル属性）
  - [`form`属性](https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#attr-fae-form)（フォームコントロール要素）
  - [`headers`属性](https://html.spec.whatwg.org/multipage/tables.html#attr-tdth-headers)（`th` `td`要素）
  - [`list`属性](https://html.spec.whatwg.org/multipage/input.html#attr-input-list)（`input`要素）
  - [`for`属性](https://html.spec.whatwg.org/multipage/forms.html#attr-label-for)（`label`要素）
  - [`for`属性](https://html.spec.whatwg.org/multipage/form-elements.html#attr-output-for)（`output`要素）
  - [`aria-activedescendantd`プロパティ](https://www.w3.org/TR/wai-aria-1.2/#aria-activedescendant)
  - [`aria-controls`プロパティ](https://www.w3.org/TR/wai-aria-1.2/#aria-controls)
  - [`aria-describedby`プロパティ](https://www.w3.org/TR/wai-aria-1.2/#aria-describedby)
  - [`aria-details`プロパティ](https://www.w3.org/TR/wai-aria-1.2/#aria-details)
  - [`aria-errormessage`プロパティ](https://www.w3.org/TR/wai-aria-1.2/#aria-errormessage)
  - [`aria-flowto`プロパティ](https://www.w3.org/TR/wai-aria-1.2/#aria-flowto)
  - [`aria-labelledby`プロパティ](https://www.w3.org/TR/wai-aria-1.2/#aria-labelledby)
  - [`aria-owns`プロパティ](https://www.w3.org/TR/wai-aria-1.2/#aria-owns)
- CSSのセレクタ
- DOM操作からの参照（`getElementById`や`querySelector`など）
- **URLフラグメント**（URLの末尾の`#`で始まる識別子）
- 解析ツールなどの**ウェブビーコンの特定要素**として
- **スクレイピングの目印**

ひとつめの「属性からの参照」については先の動的生成で解決できる。ふたつめの「CSSのセレクタ」はそもそもセレクタとしてのIDの使用を禁止すればよい。CSS設計や命名規則で規定してStylelintを利用することでIDそのものを使わないようにすることができる。3つめの「DOM操作からの参照」も動的生成でうまくやることができるだろう。そもそも動的生成がDOM操作のひとつだ。

さて問題は「URLフラグメント」「ウェブビーコンの特定要素」「スクレイピングの目印」なわけだが、これはIDは固定な値である必要があり、ランダムもしくは何かしらを基準にしたハッシュ値を動的に生成したIDはかなり扱いにくい。どれも1文字でも変わってしまうと機能しなくなってしまう。「URLフラグメント」はエンドユーザーへ**部分的なリンク切れ**を提供することになるし、ウェブビーコンはプロダクトの**解析用のデータを残せなくなる**。スクレイピングはレアケースでほとんどの場合無視してもいいかもしれないが、スクレイピングが行われていることを前提にしているサービスは気をつけないといけない。

## [markuplint](https://markuplint.dev)を使った動的IDと固定IDの棲み分け

利用ケースを踏まえると、動的IDの定義に適した要素と固定IDでなければならない要素とそれぞれあることがわかる。そのため、IDを管理するルールとしては

- 原則として`id`属性値は動的に生成する
- 固定値が必要な場合は任意の値を設定できるがページ内重複しないこと

となる。

具体的にルールが決まればあとはリンターの出番なわけで、markuplintでこのルールを元に問題を発見できるようにしてみる。

Next.jsなどのように`components`と`pages`のようにディレクトリが分かれている設計だと都合がよい。どれがページの元となるのか明確にわかるフレームワークであれば、ページ単位で重複管理が可能になる。

```
📂 src
├📂 components
│├📄 aaa.tsx
│├📄 bbb.tsx
│├📄 ccc.tsx
│└📄 ddd.tsx
└📂 pages
 ├📄 index.tsx
 ├📄 001.tsx
 ├📄 002.tsx
 └📄 003.tsx
```

たとえば、markuplintのコンフィグを次のように設定する。

```json
{
  "rules": {
    // IDの重複を警告する
    "id-duplication": true,
    // IDがハードコーディングされていたら警告する（動的生成を促す）
    "no-hard-code-id": true
  },
  "overrides": {
    // pagesディレクトリ内のファイルのみにルールを再定義
    "./src/pages/**/*": {
      "nodeRules": [
        {
          // URLフラグメント用に対象を見出しのみに絞る
          "selector": "h1,h2,h3,h4,h5,h6",
          "rules": {
            // ルールを無効化してIDのハードコーディングを許容する
            "no-hard-code-id": false
          }
        },
        {
          // Google Tag Managerトリガー
          "selector": "#gtm-trigger01,#gtm-trigger02",
          "rules": {
            // ルールを無効化してIDのハードコーディングを許容する
            "no-hard-code-id": false
          }
        }
      ]
    }
  }
}
```

基本は[`no-hard-code-id`](https://markuplint.dev/rules/no-hard-code-id)により`id`属性値をハードコーディングしていると警告されるが、`pages`内のページコンポーネントには警告がされないようになる。

```tsx
export default function () {
  return (
    <>
      <h1 id="fruits">いろんなくだもの</h1>
      <h2 id="apple">りんご</h2>
      <DetailApple />
      <TriggerButton id="gtm-trigger01" />
      <h2 id="orange">みかん</h2>
      <DetailOrange />
      <TriggerButton id="gtm-trigger02" />
    </>
  );
}
```

なお、`pages`内でも[`id-duplication`](https://markuplint.dev/rules/id-duplication)が有効になっているので、ここで重複定義をしてしまうことはない。

---

このように非標準技術依存ではあるが、フレームワークやリンターの機能をうまく利用してルールを作ることで、安定で堅牢な管理ができるようになる。

フロントエンドで`id`属性の扱いで悩んでいた方の参考になると嬉しい。
