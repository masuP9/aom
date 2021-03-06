# アクセシビリティオブジェクトモデル

**著者:**

- Alice Boxhall, Google, aboxhall@google.com
- James Craig, Apple, jcraig@apple.com
- Dominic Mazzoni, Google, dmazzoni@google.com
- Alexander Surkov, Mozilla, surkov.alexander@gmail.com

**Table of Contents**

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [はじめに](#%E3%81%AF%E3%81%98%E3%82%81%E3%81%AB)
- [モチベーションとなるユースケース](#%E3%83%A2%E3%83%81%E3%83%99%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%81%A8%E3%81%AA%E3%82%8B%E3%83%A6%E3%83%BC%E3%82%B9%E3%82%B1%E3%83%BC%E3%82%B9)
- [アクセシビリティオブジェクトモデル](#%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B7%E3%83%93%E3%83%AA%E3%83%86%E3%82%A3%E3%82%AA%E3%83%96%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88%E3%83%A2%E3%83%87%E3%83%AB)
  - [ARIA属性を反映する](#aria%E5%B1%9E%E6%80%A7%E3%82%92%E5%8F%8D%E6%98%A0%E3%81%99%E3%82%8B)
    - [Spec/implementation status](#specimplementation-status)
  - [要素の参照を反映する](#%E8%A6%81%E7%B4%A0%E3%81%AE%E5%8F%82%E7%85%A7%E3%82%92%E5%8F%8D%E6%98%A0%E3%81%99%E3%82%8B)
    - [ユースケース 2: IDREFsを使用することなく関係プロパティを設定する](#%E3%83%A6%E3%83%BC%E3%82%B9%E3%82%B1%E3%83%BC%E3%82%B9-2-idrefs%E3%82%92%E4%BD%BF%E7%94%A8%E3%81%99%E3%82%8B%E3%81%93%E3%81%A8%E3%81%AA%E3%81%8F%E9%96%A2%E4%BF%82%E3%83%97%E3%83%AD%E3%83%91%E3%83%86%E3%82%A3%E3%82%92%E8%A8%AD%E5%AE%9A%E3%81%99%E3%82%8B)
    - [Spec/implementation status](#specimplementation-status-1)
  - [`ElementInternals`オブジェクトによるカスタム要素のデフォルトセマンティクス](#%E3%82%AB%E3%82%B9%E3%82%BF%E3%83%A0%E8%A6%81%E7%B4%A0%E3%81%AEapi)
    - [ユースケース1: 非反映のデフォルトアクセシビリティプロパティをウェブコンポーネントに設定する](#%E3%83%A6%E3%83%BC%E3%82%B9%E3%82%B1%E3%83%BC%E3%82%B91-%E9%9D%9E%E5%8F%8D%E6%98%A0%E3%81%AE%E3%83%87%E3%83%95%E3%82%A9%E3%83%AB%E3%83%88%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B7%E3%83%93%E3%83%AA%E3%83%86%E3%82%A3%E3%83%97%E3%83%AD%E3%83%91%E3%83%86%E3%82%A3%E3%82%92%E3%82%A6%E3%82%A7%E3%83%96%E3%82%B3%E3%83%B3%E3%83%9D%E3%83%BC%E3%83%8D%E3%83%B3%E3%83%88%E3%81%AB%E8%A8%AD%E5%AE%9A%E3%81%99%E3%82%8B)
    - [Spec/implementation status](#specimplementation-status-2)
  - [支援技術からのユーザーアクションイベント](#%E6%94%AF%E6%8F%B4%E6%8A%80%E8%A1%93%E3%81%8B%E3%82%89%E3%81%AE%E3%83%A6%E3%83%BC%E3%82%B6%E3%83%BC%E3%82%A2%E3%82%AF%E3%82%B7%E3%83%A7%E3%83%B3%E3%82%A4%E3%83%99%E3%83%B3%E3%83%88)
    - [新しい入力イベントタイプ](#%E6%96%B0%E3%81%97%E3%81%84%E5%85%A5%E5%8A%9B%E3%82%A4%E3%83%99%E3%83%B3%E3%83%88%E3%82%BF%E3%82%A4%E3%83%97)
    - [ユースケース 3: 支援技術からのイベントをリッスンする](#%E3%83%A6%E3%83%BC%E3%82%B9%E3%82%B1%E3%83%BC%E3%82%B9-3-%E6%94%AF%E6%8F%B4%E6%8A%80%E8%A1%93%E3%81%8B%E3%82%89%E3%81%AE%E3%82%A4%E3%83%99%E3%83%B3%E3%83%88%E3%82%92%E3%83%AA%E3%83%83%E3%82%B9%E3%83%B3%E3%81%99%E3%82%8B)
    - [Spec/implementation status](#specimplementation-status-3)
  - [仮想アクセシビリティノード](#%E4%BB%AE%E6%83%B3%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B7%E3%83%93%E3%83%AA%E3%83%86%E3%82%A3%E3%83%8E%E3%83%BC%E3%83%89)
    - [ユースケース4; DOMでない仮想のノードをアクセシビリティツリーに追加する](#%E3%83%A6%E3%83%BC%E3%82%B9%E3%82%B1%E3%83%BC%E3%82%B94-dom%E3%81%A7%E3%81%AA%E3%81%84%E4%BB%AE%E6%83%B3%E3%81%AE%E3%83%8E%E3%83%BC%E3%83%89%E3%82%92%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B7%E3%83%93%E3%83%AA%E3%83%86%E3%82%A3%E3%83%84%E3%83%AA%E3%83%BC%E3%81%AB%E8%BF%BD%E5%8A%A0%E3%81%99%E3%82%8B)
    - [Spec/implementation status](#specimplementation-status-4)
  - [`ComputedAccessibleNode` によるアクセシビリティツリーの完全な確認](#computedaccessiblenode-%E3%81%AB%E3%82%88%E3%82%8B%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B7%E3%83%93%E3%83%AA%E3%83%86%E3%82%A3%E3%83%84%E3%83%AA%E3%83%BC%E3%81%AE%E5%AE%8C%E5%85%A8%E3%81%AA%E7%A2%BA%E8%AA%8D)
    - [ユースケース5: 計算されたツリーを確認する](#%E3%83%A6%E3%83%BC%E3%82%B9%E3%82%B1%E3%83%BC%E3%82%B95-%E8%A8%88%E7%AE%97%E3%81%95%E3%82%8C%E3%81%9F%E3%83%84%E3%83%AA%E3%83%BC%E3%82%92%E7%A2%BA%E8%AA%8D%E3%81%99%E3%82%8B)
    - [Spec/implementation status](#specimplementation-status-5)
    - [なぜ最終的に計算されたプロパティにアクセスするのか](#%E3%81%AA%E3%81%9C%E6%9C%80%E7%B5%82%E7%9A%84%E3%81%AB%E8%A8%88%E7%AE%97%E3%81%95%E3%82%8C%E3%81%9F%E3%83%97%E3%83%AD%E3%83%91%E3%83%86%E3%82%A3%E3%81%AB%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B9%E3%81%99%E3%82%8B%E3%81%AE%E3%81%8B)
  - [このAPIの対象者](#%E3%81%93%E3%81%AEapi%E3%81%AE%E5%AF%BE%E8%B1%A1%E8%80%85)
  - [`AccessibleNode`に何が起こったのか?](#accessiblenode%E3%81%AB%E4%BD%95%E3%81%8C%E8%B5%B7%E3%81%93%E3%81%A3%E3%81%9F%E3%81%AE%E3%81%8B)
- [次のステップ](#%E6%AC%A1%E3%81%AE%E3%82%B9%E3%83%86%E3%83%83%E3%83%97)
  - [インキュベーション](#%E3%82%A4%E3%83%B3%E3%82%AD%E3%83%A5%E3%83%99%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3)
- [謝辞](#%E8%AC%9D%E8%BE%9E)
- [付録](#%E4%BB%98%E9%8C%B2)
  - [背景: 支援技術とアクセシビリティツリー](#%E8%83%8C%E6%99%AF-%E6%94%AF%E6%8F%B4%E6%8A%80%E8%A1%93%E3%81%A8%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B7%E3%83%93%E3%83%AA%E3%83%86%E3%82%A3%E3%83%84%E3%83%AA%E3%83%BC)
    - [アクセシビリティノードプロパティ](#%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B7%E3%83%93%E3%83%AA%E3%83%86%E3%82%A3%E3%83%8E%E3%83%BC%E3%83%89%E3%83%97%E3%83%AD%E3%83%91%E3%83%86%E3%82%A3)
  - [背景: DOMツリー、アクセシビリティツリー、そしてプラットフォームのアクセシビリティAPI](#%E8%83%8C%E6%99%AF-dom%E3%83%84%E3%83%AA%E3%83%BC%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B7%E3%83%93%E3%83%AA%E3%83%86%E3%82%A3%E3%83%84%E3%83%AA%E3%83%BC%E3%81%9D%E3%81%97%E3%81%A6%E3%83%97%E3%83%A9%E3%83%83%E3%83%88%E3%83%95%E3%82%A9%E3%83%BC%E3%83%A0%E3%81%AE%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B7%E3%83%93%E3%83%AA%E3%83%86%E3%82%A3api)
    - [ネイティブHTMLをアクセシビリティツリーにマッピングする](#%E3%83%8D%E3%82%A4%E3%83%86%E3%82%A3%E3%83%96html%E3%82%92%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B7%E3%83%93%E3%83%AA%E3%83%86%E3%82%A3%E3%83%84%E3%83%AA%E3%83%BC%E3%81%AB%E3%83%9E%E3%83%83%E3%83%94%E3%83%B3%E3%82%B0%E3%81%99%E3%82%8B)
    - [ARIA](#aria)
  - [付録: `AccessibleNode` の命名](#%E4%BB%98%E9%8C%B2-accessiblenode-%E3%81%AE%E5%91%BD%E5%90%8D)
  - [Appendix: Partial proposed IDL for virtual accessibility nodes](#appendix-partial-proposed-idl-for-virtual-accessibility-nodes)
  - [Appendix: partial proposed IDL for `ComputedAccessibleNode`](#appendix-partial-proposed-idl-for-computedaccessiblenode)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## はじめに

この取り組みの目的は、Webプラットフォームへの追加機能を開発して、開発者が支援技術APIに情報を提供できるようにし、ブラウザーがそれらのAPIに提供する情報を理解することです。

## モチベーションとなるユースケース

既存のAPIの背景は[付録](#%E4%BB%98%E9%8C%B2)で見つけることができる。

ウェブ上でできることの境界を押し広げているウェブアプリケーションは、APIが不十分なためそれらをアクセシブルにするため苦闘している。特に、ブラウザとやりとりするネイティブAPIと比較して表現力に劣っている。

1. ページの著者が上書きできる[ウェブコンポーネント](https://developer.mozilla.org/en-US/docs/Web/Web_Components)のデフォルトアクセシビリティプロパティの設定
   - 現在、ウェブコンポーネントのデフォルトセマンティクスを定義するにはARIAを使用しなければならない。
   これにより、真に詳細な実装であるARIA属性がDOMに「リーク」してしまう。
   - このことは _必要ではなく_ ウェブコンポーネントに限られる _かも_ 知れない。
2. IDREFs を必要としない[関係属性](https://www.w3.org/TR/wai-aria-1.1/#attrs_relationships)の設定
   - 現在、いくつかのARIAの関係を示すには、著者が一意のIDを関係の対象となりうる要素に指定しなければならない
   - [`aria-activedescendant`](https://www.w3.org/TR/wai-aria-1.1/#aria-activedescendant)のような場合、
   それはUIに応じて、数百ないしは数千もの要素のうち、一つを参照するかもしれない。
   この要求は、多くの余分なDOMの属性が必要となり、これらのAPIを複雑にする。
3. 支援技術からのイベントをリスニングする
   - 現在、組み込み要素 _だけ_ がイベントに反応することができ、
   通常、[「increment」](https://developer.apple.com/documentation/objectivec/nsobject/1615076-accessibilityincrement) のようなユーザーアクションによって引き起こされる。
4. アクセシビリティツリーにDOMでないノード（仮想ノード）を追加する
   - 例えば、`<canvas>` 要素で構築された複雑なUIや、`<video>` 要素を用いたリモートデスクトップのストリーミングなどを表現するなど
   - そのためには、少なくとも要素と同じようなアクセシビリティプロパティや、他の仮想ノードとの親/子/その他の関係性、位置や次元を表す必要がある。
5. 計算済のアクセシビリティツリーの確認
   - 開発者は現在、ARIAやその他のアクセシビリティプロパティがどのように適用されているかを調べたりテストする方法を持っていない。

## アクセシビリティオブジェクトモデル

アクセシビリティオブジェクトモデル (AOM)は、上記のユースケースに取り組むためのHTMLや関連する標準に対する一連の変更である。

(注: 以前のバージョンのAOMに慣れ親しんでいれば、[`AccessibleNode`に何が起こったのか?](#accessiblenode%E3%81%AB%E4%BD%95%E3%81%8C%E8%B5%B7%E3%81%93%E3%81%A3%E3%81%9F%E3%81%AE%E3%81%8B)と疑問を持つかもしれない。)

### ARIA属性を反映する

ARIA 属性をHTML要素に[反映する](https://html.spec.whatwg.org/multipage/common-dom-interfaces.html#reflect)。

```js
el.role = "button";
el.ariaPressed = "true";  // aria-pressed は3つのステートを持つ属性
el.ariaDisabled = true;   // aria-disabled は true/false を持つ属性
```

#### Spec/implementation status

This is now a part of the [ARIA 1.2 spec](https://www.w3.org/TR/wai-aria-1.2/#idl-interface).

This is shipping in Safari,
and implemented behind a flag (`enable-experimental-web-platform-features`) in Chrome.

### 要素の参照を反映する

ARIAプロパティの直接的反映。`aria-labelledby` のような関係属性の文字列に反映させる。

```js
el.ariaDescribedBy = "id1";
```

結果は

```html
<div aria-describedby="id1"></div>
```

要素を参照する非反映のプロパティでAPIを拡張することを提案する。

```js
el.ariaDescribedByElements = [labelElement1, labelElement2];
el.ariaActiveDescendantElement = ownedElement1;
```

> Note: the `Element` or `Element` suffixes are a naming choice
> for the reflected property,
> and do not imply that there will be both string and Element properties
> for the same attribute.

これにより、関係に属するそれぞれの要素にグローバルに固有なID属性を割り当てることなく要素間のセマンティクスの関係性を示すことができる。

さらにこれは、`ShadowRoot`を使用した著者がShadow DOMの境界を超えて関係性を明示することを可能にする

#### ユースケース 2: IDREFsを使用することなく関係プロパティを設定する

現在、Shadow DOMの境界を超えて関係を表そうとする著者は次のように `aria-activedescendant` の使用を試すだろう。

```html
<custom-combobox>
  #shadow-root (open)
  |  <!-- これはうまくいかない！ -->
  |  <input aria-activedescendant="opt1"></input>
  |  <slot></slot>
  <custom-optionlist>
    <x-option id="opt1">Option 1</x-option>
    <x-option id="opt2">Option 2</x-option>
    <x-option id='opt3'>Option 3</x-option>
 </custom-optionlist>
</custom-combobox>
```

これは失敗で、なぜならIDREFsはshadowRoot、またはそれらが現れる文書の文脈の範囲に限られる。

要素参照を利用することで著者は代わりにこの関係性をプログラムで示すことができる。

```js
const input = comboBox.shadowRoot.querySelector("input");
const optionList = comboBox.querySelector("custom-optionlist");
input.activeDescendantElement = optionList.firstChild;
```

このことにより、関係が自然に表現されるようになる。

#### Spec/implementation status

- このAPIはWHATWG HTML使用の変更として[提案されている](https://github.com/whatwg/html/issues/3515)。
- There is an [open PR](https://github.com/whatwg/html/pull/3917) on the HTML spec
  fleshing out the details for this API.
- This is used in the [ARIA editor's draft](https://w3c.github.io/aria/#AriaAttributes).
- This is [currently being implemented in Blink](https://www.chromestatus.com/feature/6244885579431936).

### Default semantics for Custom Elements via the `ElementInternals` object

We propose that Custom Element authors be able to provide default semantics
via the `ElementInternals` object.

A custom element author may use the `ElementInternals` object
to modify the semantic state of an instance of a custom element
in response to user interaction.

The properties set on the `ElementInternals` object
are used when mapping the element to an accessible object.

If the author-provided semantics conflict with the Custom Element semantics,
the author-provided semantics take precedence.

> Note: this is analogous to setting an "instance variable" -
> a copy of these semantic properties is created for each instance of the custom element.
> The semantics defined in each apply only to their associated custom element instance object.

#### ユースケース1: 非反映のデフォルトアクセシビリティプロパティをウェブコンポーネントに設定する

今、ウェブコンポーネントを制作するライブラリの著者はネイティブ要素にとっては暗黙であるセマンティクスを表すためにARIA属性を「生やす」ことを強いられている。

```html
<!-- ページの著者はカスタム要素をネイティブ要素を使用するように使用する -->
<custom-tablist>
  <custom-tab selected>Tab 1</custom-tab>
  <custom-tab>Tab 2</custom-tab>
  <custom-tab>Tab 3</custom-tab>
</custom-tablist>

<!-- カスタム要素がセマンティクスを表すために余分な属性を「生やす」ことを強制される -->
<custom-tablist role="tablist">
  <custom-tab
    selected
    role="tab"
    aria-selected="true"
    aria-controls="tabpanel-1"
    >Tab 1</custom-tab
  >
  <custom-tab role="tab" aria-controls="tabpanel-2">Tab 2</custom-tab>
  <custom-tab role="tab" aria-controls="tabpanel-3">Tab 3</custom-tab>
</custom-tablist>
```

Using `ElementInternals` to set the default semantics,
a Custom Element may avoid needing to sprout attributes,
and also avoid losing its semantics if authors decide to delete ARIA attributes.

```js
class CustomTab extends HTMLElement {
  constructor() {
    super();
    this._internals = customElements.createInternals(this);
    this._internals.role = "tab";
  }

  // カスタム「active」属性を監視する
  static get observedAttributes() {
    return ["active"];
  }

  connectedCallback() {
    this._tablist = this.parentElement;
  }

  setTabPanel(tabpanel) {
    if (tabpanel.localName !== "custom-tabpanel" || tabPanel.id === "")
      return;  // 静かに失敗する

    this._tabpanel = tabpanel;
    tabpanel.setTab(this);
    this._internals.ariaControls = tabPanel; // 反映されない
  }

  // 属性に反映するカスタムプロパティの setters/getters

  attributeChangedCallback(name, oldValue, newValue) {
    switch (name) {
      case "active":
        let active = newValue != null;
        this._tabpanel.shown = active;

        // カスタム「active」属性が変更された時、
        // アクセシブルな「selected」ステートを同期し続ける
        this._internals.ariaSelected = newValue !== null;

        if (selected) this._tablist.setSelectedTab(this); // 他のタブが 「active」で無いことを保証
        break;
    }
  }
}

customElements.define("custom-tab", CustomTab);
```

これらの要素を使用する著者は通常通りARIAを用いてデフォルトセマンティクスを上書きすることができる。

例えば、著者は `<custom-tablist>` 要素の見た目を縦並びに変更できる。彼らはそれを示すために `aria-orientation` 属性を追加することで、カスタム要素に実装されているデフォルトセマンティクスを上書きすることができる。

```js
class CustomTabList extends HTMLElement {
  constructor() {
    super();
    this._internals = customElements.createInternals(this);
    this._internals.role = "tablist";
    this._internals.ariaOrientation = "horizontal";
  }

  // ...
}

customElements.define("custom-tablist", CustomTabList);
```

```html
<custom-tablist aria-orientation="vertical" class="vertical-tablist">
  <custom-tab selected>Tab 1</custom-tab>
  <custom-tab>Tab 2</custom-tab>
  <custom-tab>Tab 3</custom-tab>
</custom-tablist>
```

#### Spec/implementation status

- There is an [open PR](https://github.com/whatwg/html/pull/4658) on the WHATWG HTML spec.
- This is [currently being implemented in Blink](https://chromestatus.com/feature/5962105603751936).

### 支援技術からのユーザーアクションイベント

支援技術ユーザーのプライバシーを保護するために、通常、支援技術からのイベントは合成されたDOMイベントを発生させる。The events are determined by
platform conventions and partially documented in the [ARIA Authoring Practices Guide (APG)](https://www.w3.org/TR/wai-aria-practices/#aria_ex).

| **支援技術のイベント** | **ターゲット** | Orientation/Direction | **DOMイベント** |
| ------------------ | ----------- | --------------------- | ------------- |
| `click` or `press` | _すべての要素_ |                       | `click` MouseEvent |
| `focus` | _all focusable elements_ |                       | `focus` Event |
| `blur` | No targets, as `blur` could potentially 'out' AT users. |                       | None |
| `select`              | Elements whose computed role supports `aria-selected` |                       | `click` MouseEvent                 |
| `dismiss` or `escape` | _すべての要素_ |                       | `Escape` KeyboardEvent             |
| `contextMenu`         | _すべての要素_ |                       | `contextmenu` MouseEvent           |
| `increment`           | Elements w/ computed role `progressbar`, `scrollbar`, or `slider` | vertical              | `Up` KeyboardEvent                 |
|                       | ""                                                                | horizontal LTR        | `Right` KeyboardEvent              |
|                       | ""                                                                | horizontal RTL        | `Left` KeyboardEvent               |
|                       | Elements w/ computed role `spinbutton`                            | orientation n/a       | `Up` KeyboardEvent                 |
| `decrement`           | Elements w/ computed role `progressbar`, `scrollbar`, or `slider` | vertical              | `Down` KeyboardEvent               |
|                       | ""                                                                | horizontal LTR        | `Left` KeyboardEvent               |
|                       | ""                                                                | horizontal RTL        | `Right` KeyboardEvent              |
|                       | Elements w/ computed role `spinbutton`                            | orientation n/a       | `Down` KeyboardEvent               |
| `scrollByPage`        | TBD (possibly custom scroll views)                                |                       | TBD (possibly `PageUp`/`PageDown`) |
| `scrollIntoView`      | TBD                                                               |                       | No equivalent DOM event            |
| `setValue`            | n/a                                                               |                       | No equivalent DOM event            |

#### Notes on the previous table:
- DOM KeyboardEvent sequences include keyup/keydown.
- DOM MouseEvent sequences include mousedown/mouseup and touchstart/touchend where relevant.
- `contextmenu` sequence may need to include MouseEvents, including `mousedown`/`mouseup`/`auxclick`/`contextmenu`.
- Control orientation is determined by the computed value of `aria-orientation` which
  defaults to `horizontal` for `progressbar` and `slider`, and defaults to `vertical` for
  `scrollbar`.
- Natural language direction is determined by the computed value of `dir` which usualy computes to
  to `ltr` (`auto` in most contexts resolves to `ltr`), but can be set to `rtl` for languages such
  as Arabic and Hebrew.
- The DOM event target for DOM KeyboardEvent sequences is the currently focused DOM element,
  regardless if the AT's "point of regard" matches the document.activeElement.
- If a web author does not cancel the DOM event with `Event.preventDefault()` and/or
  `Event.stopPropagation()`, the DOM event should propagate out of the web view an potentially
  trigger the platform behavior of the assistive technology event. For example, if an iOS
  user triggers a native dismiss/escape event but the web author does not capture or cancel the
  DOM Escape key sequence, the browser or system should execute the default functionality of the
  native `accessibilityPerformEscape()` handler.


#### MouseEvent Object Properties

| **MouseEvent** | **`button`**        | **`target`/`srcElement`** | **`which` (deprecated)** |
| -------------- | ------------------- | ------------------------- | ------------------------ |
| click          | 1                   | TBD                       | 1                        |
| contextmenu    | 2 (secondary click) | TBD                       | 3 (legacy right click)   |

Note: Only send the deprecated `which` property if the user agent would normally send it on a non-synthesized mouse event.

Note: The `target` and `srcElement` properties should match the most likely element in the case of a non-synthesized MouseEvent (a real mouse click). Since AT focus targets and pointer event targets do not always align one-to-one, this event property is currently TBD. For example, users agents might attempt to synthesize a pointer event x/y position near the center of the element in AT focus. If hit-testing at that x/y position does not return a descendant of the element in AT focus, user agents might synthesize the event on the element directly in AT focus.



#### KeyboardEvent Object Properties

| **KeyEvent** | **`key`**    | **`code`**   | **`location`**                | **`target`/`srcElement`** |
| ------------ | ------------ | ------------ | ----------------------------- | ------------------------- |
| Escape       | "Escape"     | "Escape"     | DOM_KEY_LOCATION_STANDARD (0) | `document.activeElement`  |
| Left         | "ArrowLeft"  | "ArrowLeft"  | DOM_KEY_LOCATION_STANDARD (0) | `document.activeElement`  |
| Up           | "ArrowUp"    | "ArrowUp"    | DOM_KEY_LOCATION_STANDARD (0) | `document.activeElement`  |
| Right        | "ArrowRight" | "ArrowRight" | DOM_KEY_LOCATION_STANDARD (0) | `document.activeElement`  |
| Down         | "ArrowDown"  | "ArrowDown"  | DOM_KEY_LOCATION_STANDARD (0) | `document.activeElement`  |

The `target` and `srcElement` properties should match `document.activeElement`, which is either the currently focused element or `document.body`.


#### Deprecated KeyboardEvent Object Properties (Optional)

Only send these deprecated properties if the user agent would normally send them on non-synthesized keyboard events.

| **KeyEvent** | **`charCode`** | **`keyCode`** | **`keyIdentifier`**                 | **`keyLocation`** | **`which`** |
| ------------ | -------------- | ------------- | ----------------------------------- | ----------------- | ----------- |
| Escape       | 0              | 27            | U+001B (Unicode Character 'ESCAPE') | 0                 | 27          |
| Left         | 0              | 37            | `Left`                              | 0                 | 37          |
| Up           | 0              | 38            | `Up`                                | 0                 | 38          |
| Right        | 0              | 39            | `Right`                             | 0                 | 39          |
| Down         | 0              | 40            | `Down`                              | 0                 | 40          |


Note: These event property tables are intended to assist implementors during the incubation process. This is not intended as a normative specification.


#### Speculative: New InputEvent types

Note: This section is speculative, as there is now no immediate plan to include InputEvents 
for Assistive Technology Actions.

いくつかの[`Input Event`](https://www.w3.org/TR/uievents/#inputevent) タイプを追加する:

- `increment`
- `decrement`
- `dismiss`
- `scrollPageUp`
- `scrollPageDown`

これらのイベントは上記の表に記載された合成キーボードイベントと共に支援技術のイベントから引き起こされ、また上記の関連する支援技術のイベントに対応した有効なターゲットの文脈の中で発生したときに合成される。

例えば、もしある文脈で支援技術を使用していないユーザーが `Escape` キーを押した場合、キーが押されたシーケンスで `dismiss` タイプを伴った `input` イベントが発火する。

もし同じユーザーが `<input type="range">` _または_ （計算された `slider` ロールを含む）`slider` ロールを持った要素上にフォーカスがある `Up` キーを押した場合、キーが押されたシーケンスで `increment` タイプを伴った `input` イベントがフォーカスされた要素で発火される。

#### ユースケース3: 支援技術からのイベントをリッスンする

例えば:

- ユーザーは音声制御ソフトウェアを利用しており、ウェブページの中のどこかのボタンの名前を読み上げられる。音声制御ソフトウェアはアクセシビリティツリーの中に該当する名前があるボタンを見つけ、クリックすることで _アクション_ を送信することができる。
- 同じユーザーはあるページをスクロールダウンする音声コマンドを発行する。その音声制御ソフトウェアはウェブページのルート要素を見つけ、スクロール _アクション_ を送信する。
- モバイルスクリーンリーダーユーザーは、スライダーに移動し、範囲に基づいたコントロールを増加させるジェスチャーを実行することができる。スクリーンリーダーはアクセシビリティツリーの中のスライダー要素にブラウザーに増加 _アクション_ を送信する。

現在、ブラウザーはネイティブHTML要素の組み込みサポートを実装することによって、アクセシビリティアクションを一部実装している。（例えばネイティブHTMLの `<input type="range">` はすでに増加・減少アクションをサポートし、テキストボックスは値を設定したりテキストを挿入するアクションをサポートしている。）

しかしウェブページの著者がカスタムエレメント上のアクセシビリティアクションをリッスンする方法が無い。
例えば、`slider` [ロールを持ったカスタムスライダー](#%E3%83%A6%E3%83%BC%E3%82%B9%E3%82%B1%E3%83%BC%E3%82%B91-%E9%9D%9E%E5%8F%8D%E6%98%A0%E3%81%AE%E3%83%87%E3%83%95%E3%82%A9%E3%83%AB%E3%83%88%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B7%E3%83%93%E3%83%AA%E3%83%86%E3%82%A3%E3%83%97%E3%83%AD%E3%83%91%E3%83%86%E3%82%A3%E3%82%92%E3%82%A6%E3%82%A7%E3%83%96%E3%82%B3%E3%83%B3%E3%83%9D%E3%83%BC%E3%83%8D%E3%83%B3%E3%83%88%E3%81%AB%E8%A8%AD%E5%AE%9A%E3%81%99%E3%82%8B) 上ではiOSのVoiceOverでは増加・現象のためにスワイプアクションを促す提案がされるが、いかなるウェブAPIもそのセマンティックイベントを扱うことができない。

開発者はセマンティックイベントを補足するようなキーボードイベントまたは入力イベントをリッスンすることができるようになるだろう。

例えば、著者はカスタムスライダーを実装するために [ARIA Authoring Practices guide](https://www.w3.org/TR/wai-aria-practices-1.1/#slider_kbd_interaction)で推奨されている `Up` と `Down` キーイベントを扱うことができ、同じように支援技術のイベントも扱うことができる。

```js
customSlider.addEventListener('keydown', (event) => {
  switch (event.code) {
  case "ArrowUp":
    customSlider.value += 1;
    return;
  case "ArrowDown":
    customSlider.value -= 1;
    return;
});
```

#### Spec/implementation status

Not yet specced or implemented anywhere.

### 仮想アクセシビリティノード

**仮想アクセシビリティノード** は著者に、支援技術に特定のDOM要素に直接関係のない *仮想* のアクセシビリティノードに触れさせられることができる。

このメカニズムは、著者がカスタム描画APIのアクセシビリティをよりきめ細かくコントロールするためにネイティブアクセシビリティAPIによく存在する。

- `attachAccessibleRoot()` を実行することで `AccessibleNode` が `Node` に関連付けられる。
  - リターンされた `AccessibleNode` は仮想アクセシビリティツリーのルートを形成する。
  - そのノードのDOM子要素は、`AccessibleRoot` が付与されると、暗黙のうちに無視され、DOMの子要素と仮想アクセシブルノードが混在することはない。
- `ShadowRoot` のように、一つの要素は一つの関連付けられた `AccessibleRoot` しか持たない。
- `AccessibleNode` だけが子に `AccessibleNodes` 持ち、 `AccessibleNode` は `AccessibleNode` のみ子として持つ。

#### ユースケース4; DOMでない仮想のノードをアクセシビリティツリーに追加する

例えば、`<canvas>` 要素から構築された複雑なUIを表すには:

```js
// canvas ベースのスプレッドシートのセマンティクスを実現する
canvas.attachAccessibleRoot();
let table = canvas.accessibleRoot.appendChild(new AccessibleNode());
table.role = "table";
table.colCount = 10;
table.rowcount = 100;
let headerRow = table.appendChild(new AccessibleNode());
headerRow.role = "row";
headerRow.rowindex = 0;
// などなど
```

仮想ノードは通常位置と寸法を明示的に設定する必要がある。

```js
cell.offsetLeft = "30px";
cell.offsetTop = "20px";
cell.offsetWidth = "400px";
cell.offsetHeight = "300px";
cell.offsetParent = table;
```

もし `offsetParent` を設定しない場合、アクセシブルノードの親を基準として解釈される。

ノードをフォーカスできるようにする場合、 `focusable` 属性を設定することができる。これは `tabIndex=-1` をDOM要素に設定するのに似ている。

```js
virtualNode.focusable = true;
```

仮想アクセシブルノードはデフォルトではフォーカスできない。

最後に、アクセシビリティノードにフォーカスするには、 `focus()` メソッドを実行する。

```js
virtualNode.focus();
```

仮想アクセシブルノードがフォーカスされたとき、DOMのフォーカスは変わらない。アクセシビリティノードのフォーカスは、支援技術や他のアクセシビリティAPIクライアントに伝わるが、DOMイベントは発火せず document.activeElementも変わらない。

DOM要素のフォーカスが変わった時、アクセシブルフォーカスも追従し、DOM要素に関連付けられたアクセシブルノードがフォーカスされる。

#### Spec/implementation status

Not yet specced or implemented anywhere.

### `ComputedAccessibleNode` によるアクセシビリティツリーの完全な確認

This API is still being considered.

It may be approached initially as a testing-only API.

#### ユースケース5: 計算されたツリーを確認する

**計算されたアクセシビリティツリー** APIは著者に、完全な計算されたアクセシビリティツリーにアクセスすることを許可する。それぞれのDOM要素に関連付けられたアクセシビリティノードのすべての計算されたプロパティに加え、仮想ノードを含む計算された木構造を走査可能にする。

このことは次のことを可能にする:
- ページ、または要素のセマンティックプロパティが確からしいかを確認するプログラムによるテストを書くこと
- ブラウザーベースの信頼できる支援技術の構築。例えば、アクセシビリティツリーを利用したスクリーンリーダー、スクリーンルーペ、または他の支援機能を持ったブラウザー拡張やページ内ツール。
- アクセシビリティプロパティが（ARIAなどを通じて）要素に正しく適用されたかを検査する。例えば、ブラウザが特定のバージョンのARIAに対応しているかを検査できる。
- アクセシビリティツリーの問題をコンソールベースでデバッグしたりチェックしたりする。
- アクセシビリティツリーの状態に反応する。例えば要素に表されたロールを検査したり、アクセシブルなヘルプテキストを変更したりなど。

#### Spec/implementation status

A purely experimental implementation exists in Blink,
via the command-line flag `--enable-blink-features="AccessibilityObjectModel"`.

This adds a method to `Window`, `getComputedAccessibleNode(node)`,
which returns the computed accessible properties for the given node.

This implementation is not reliable and may be removed at any point.

#### なぜ最終的に計算されたプロパティにアクセスするのか

**一貫性**
現在、アクセシビリティツリーはブラウザ間で標準化されていない。それぞれのアクセシビリティツリーの計算は実装がわずかに異なる。

このAPIが役に立つには、ブラウザ間で一貫して動作することが必要で、開発者はそれぞれに特別なコードを書く必要がない。

我々はツリーがどのように計算され表現されるかについて合意ができるよう十分な時間をかけていきたいと考えている。

**パフォーマンス**
多くのアクセシビリティプロパティを計算するにはレイアウトが必要となる。ウェブ制作者に単純なプロパティアクセスと同期してアクセシビリティプロパティの計算された値を調べることを許可すると、パフォーマンスのボトルネックに混乱をもたらしてしまう。

このことから `accessibleNode` インターフェースの一部を意味しない非同期的なメカニズムを作成したいと考えている。

**ユーザー体験**
前の3つと比較して計算されたアクセシビリティツリーにアクセスすることはユーザーに与える影響が最小限となる。[優先順位の構成要素](https://www.w3.org/TR/html-design-principles/#priority-of-constituencies)の考え方の元、このことに最後に取り組むことは理にかなっている。

### このAPIの対象者

このAPIは主に、ウェブアプリの多くを動かしているJavaScriptフレームワークやウィジェットライブラリーを制作、メンテナンスしている比較的少数の開発者に興味を持たれるだろう。

それらのフレームワークやライブラリーにとってアクセシビリティは、可能な限り幅広く多様な文脈で使用されるようになるため、重要な目標の一つである。

低レベルのAPIによって彼らは制約やバグを回避し、彼らの制作したコンポーネントを使用する開発者のためにクリーンな高レベルのインターフェースを提供できるようになる。

このAPIはウェブプラットフォームの境界を押し広げるような巨大なフラッグシップウェブアプリの開発者をも対象としている。

これらのアプリの開発チームでは、Canvasのような低レベルAPIを利用してでもパフォーマンスを向上させたがっていることがしばしばある。

そのような開発チームだと、アクセシビリティを優先するだけのリソースもあるが、既存のAPIでは非常に面倒である。

### `AccessibleNode`に何が起こったのか?

当初の意図としては、これまでのユースケースを読み書き可能なAPIにまとめるものだった。それはそれぞれのDOMの `Element` に、アクセシビリティプロパティを読み込んだり書き込んだりできる `AccessibleNode` が関連付けられているという点でDOMに似ていた。

これはドキュメントオブジェクトモデルと似たようなアクセシビリティオブジェクトモデルと命名された。

しかし、議論が進むにつれこのモデルにはいくつかの問題があることがわかってきた。
- アクセシビリティツリーを計算することは、それを変更するために必要にはならない。書き込むための `AccessibleNode` を取得することは、計算済のプロパティを取得することに依存してはならない
- 計算されたアクセシビリティツリーを表示するにはブラウザ間で計算方法の標準化が必要となる
- もし `Element` の `AccessibleNode` の計算済のプロパティを表示しない場合、このオブジェクトの「プロパティの入れ物」を超えるという目的がぼやける
- ARIAプロパティと `AccessibleNode` プロパティの優先度の決定について、明確で「正しい」答えがなかった
- 同様に、`AccessibleNode` を排他的に使用して関係性を表現することは分かりにくかった

これらの問題により、APIをもともと対処するつもりだったユースケースに基づいて再評価、そして簡素化することとなった。

## 次のステップ

アクセシビリティオブジェクトモデルの開発は、現在の主要なブラウザベンダーを代表した編集者のチームがリードしている。

GitHub上で問題を報告することができる。

https://github.com/WICG/aom/issues

### インキュベーション

[Web Platform Incubator Community Group (WICG)](https://www.w3.org/community/wicg/)の一部としてこの仕様の開発は継続するつもりだが、時が経てば自身のコミュニティグループに開発が移るかもしれない。

このグループの活動は、[ARIA](https://www.w3.org/TR/wai-aria/)などの[Web Accessibility Initiative](https://www.w3.org/WAI/)の現在の活動からは全体的にほとんど独立している。

ARIAがWeb上のアクセシビリティプロパティのための構造的なマークアップとセマンティクスを定義している際には、しばしば支援技術のベンダーやネイティブプラットフォームのAPIと調整をしなければならないが、AOMは単に開発者のためにより低レベルの制御ができる並列のJavaScriptのAPIを提供し、ウェブプラットフォームのギャップを埋めるにとどまり、新しいセマンティクスを導入することは無い。

## 謝辞

価値のあるフィードバックや助言、ツールを提供してくれた次の方々に多大な感謝を。

- Alex Russell
- Anne van Kesteren
- Bogdan Brinza
- Chris Fleizach
- Chris Hall
- Cynthia Shelley
- David Bolter
- Domenic Denicola
- Elliott Sprehn
- Ian Hickson
- Joanmarie Diggs
- Marcos Caceres
- Meredith Lane
- Nan Wang
- Robin Berjon
- Rossen Atanassov
- Ryosuke Niwa
- Tess O'Connor

マイクロソフトの Bogdan Brinza と Cynthia Shelley は現在は積極的に参加していないが、この仕様の最初の草案に貢献した。

# 付録

## 背景: 支援技術とアクセシビリティツリー

この仕様の中で支援技術は、アプリケーションの既存のUIを補強したり置き換えたりする第三者のアプリケーションのことを指す。よく知られた例の一つとしてスクリーンリーダーがある。スクリーンリーダーは視覚的でポインターベースのUIを、聴覚出力（音声、およびトーン）と、キーワード、ジェスチャーの両方、またはいずれかによる入力メカニズムに置き換える。

多くの支援技術は、アクセシビリティAPIを通じてウェブページとやりとりする。たとえば、Windows上の[UIAutomation](https://msdn.microsoft.com/en-us/library/windows/desktop/ee684009.aspx)や、OS Xの[NSAccessibility](https://developer.apple.com/library/mac/documentation/AppKit/Reference/NSAccessibility_Protocol_Reference/)などがある。これらのAPIを利用するとアプリケーションのインターフェースを示すオブジェクトのツリーを公開することができる。アプリケーションウィンドウのことを示すルートノードから、個々のインタラクティブな要素まで様々なレベルのノードがグループ化されている。これは **アクセシビリティツリー** と呼ばれている。

支援技術がアクセシビリティツリーを使って代替インターフェースを作り、ユーザーの命令で発火したインタラクションイベントが支援技術にルーティングされ、支援技術のユーザーはほぼこのAPIを通じてアプリケーションとやりとりをする。

![アプリケーションUIからアクセシビリティツリー、支援技術、ユーザーまでの流れ](images/a11y-tree.png)

代替インターフェースの _出力_ （例えば、音声やトーン、[点字ディスプレイ](https://en.wikipedia.org/wiki/Refreshable_braille_display)の更新、[スクリーンルーペ](https://en.wikipedia.org/wiki/Screen_magnifier)のフォーカスの移動)、また _入力_ （例えば、キーボードショートカット、ジェスチャー、点字ルーティングキー、[スイッチ機器](https://en.wikipedia.org/wiki/Switch_access)、音声入力など）は完全に支援技術の責務で、アプリケーションの役割ではない。

例えば、OS Xのネイティブアプリケーションを使用する[VoiceOver](https://www.apple.com/voiceover/info/guide/)ユーザーは、「control、option、スペースバー」キーの組み合わせ押す。それはスクリーンリーダーが現在いるUI要素をクリックするということを示す。

![UIの要素からアクセシビリティノード、支援技術、ユーザー、ユーザーのキー操作、アクセシビリティAPIのアクションをUI要素に戻す往復](images/a11y-tree-example.png)

これらのキー押下はアプリケーションに渡されることはないが、スクリーンリーダーが受け取り、[`accessibilityPerformPress()`](https://developer.apple.com/reference/appkit/nsaccessibilitybutton/1525542-accessibilityperformpress?language=objc)関数を該当のUI要素を示すアクセシビリティノード上で実行する。アプリケーションはプレスアクションを処理し、通常はクリックイベントを処理するコードにルーティングされる。

アクセシビリティAPIはテストや自動化でもよく利用される。それらは堅牢で包括的な方法でアプリケーションの状態を調査、またプロセスの外からUIを操作する方法を提供する。通常、アクセシビリティAPIの主な動機は障害を持つユーザーのための支援技術だが、これらのAPIが一般的で多くの用途があることを理解することは重要である。

### アクセシビリティノードプロパティ

アクセシビリティツリーの各ノードは **アクセシビリティノード** と呼ばれる。

アクセシビリティノードは常に一つのセマンティックな目的を示す **ロール** を持つ。
これは単に他のノードを包含することを示すグループ化するロールであってもよいし、または `"button"` のようなインタラクティブなロールであってもよい。

![アクセシビリティツリーの中のアクセシビリティノード。ロール、名前、状態とプロパティを示している。](images/a11y-node.png)

ユーザーは支援技術を介して、様々なレベルでアクセシビリティツリーを探索することができる。かれらは、ランドマーク要素のようなユーザーのページのセクションへの移動を助けるグループ化されたノードとやり取りすることができたり、ボタンのようなインタラクティブなノードとやり取りすることができる。

それらの双方の場合において、文脈の中でのノードの役割を示すため、ノードは通常 **ラベル** （しばしば **名前** と呼ばれる）を必要とする。例えば、ボタンは「OK」や「メニュー」というラベルを持つ。

アクセシビリティノードはまた現在の **値** （範囲であれば `"10"` だったり、テキスト入力であれば `"ジェーン"` であったり）のような他のプロパティを持ったり、（チェックボックスの `"checked"` または `"focused"`など）の状態の情報を持つことができる。

インタラクティブなアクセシビリティノードは、それらに対して実行される特定の **アクション** も持つことができる。例えば、ボタンは `"press"` アクションを公開するし、スライダーは `"increment"` と `"decrement"` アクションを公開する。

これらのプロパティとアクションはノードの _セマンティクス_ と呼ばれる。各アクセシビリティAPIにおいて少しずつ違いはるが、概念的にはほとんど似ている。

## 背景: DOMツリー、アクセシビリティツリー、そしてプラットフォームのアクセシビリティAPI

ウェブはアクセシブルなアプリケーションを制作するのに多くのサポートを提供しているが、それらは *宣言的* なAPIを通じてのみである。

DOMツリーは並行して主にページの視覚的な表現とアクセシビリティツリーに変換される。アクセシビリティツリーは、一つまたは複数の _プラットフォーム固有の_ アクセシビリティAPIを介してアクセスされる。

![HTMLがDOMツリーに変換され、されに視覚的なUIとアクセシビリティツリーに変換される](images/DOM-a11y-tree.png)

異なるプラットフォーム間で複数のアクセシビリティAPIをサポートしているブラウザもあれば、特定のアクセシビリティのみをサポートしているブラウザもある。しかし、少なくとも１つのネイティブアクセシビリティAPIをサポートするブラウザはセマンティックの情報構造ツリーを公開するためのメカニズムを持っている。このAPIの目的のため、実装の詳細は気にせずそれらのAPIを参照する。

### ネイティブHTMLをアクセシビリティツリーにマッピングする

ネイティブHTML要素はアクセシビリティAPIに暗黙的にマップされる。例えば `<img>` 要素は自動的に `image` ロールを持つアクセシビリティノードにマップされ、 `alt` 属性（があれば）によってラベリングされる。

![<img>ノードが、ページ、またアクセシビリティノード上の image として変換される](images/a11y-node-img.png)

### ARIA

また、[ARIA](https://www.w3.org/TR/wai-aria-1.1/)を利用することで開発者が属性によって要素に注釈をつけ、要素のデフォルトのロールやセマンティックプロパティを上書きすることができるが、アクセシビリティアクションは公開できない。

![<div role=checkbox aria-checked=true> は視覚的なUIとDOMノードに変換される](images/a11y-node-ARIA.png)

どちらの場合も、DOMノードとアクセシビリティツリー内のノードは1対1で対応しており、対応するアクセシビリティノードのセマンティクスにたいする最小限の細やかな制御ができる。

## 付録: `AccessibleNode` の命名

仮想アクセシビリティツリー内の一つのノードを示すクラスの名前を `AccessibleNode` とした。

この名前を選ぶ際、簡潔さと明確さ、そして普遍性の間のバランスを取ろうとした。

- 簡潔さ: 名前は可能な限り短くある必要がある
- 明確さ: 名前は、分かりにくい略語や短縮形を利用せず、APIの機能を反映している必要がある
- 普遍性: 名前は仕様の範囲を制限したり狭め過ぎたりしてはいけない

以下に、真剣に提案されたすべての名前のそれぞれの長所と短所を簡潔にまとめた。

まだ以下の名前への投票や違う名前の提案は歓迎しているが、まずは既存の提案とその懸念点を慎重に考慮してほしい。すでに大筋で合意には至っており、完璧な命名にしようとするよりはむしろ世に出すよう努めていきたい。

提案された名前           | 長所                                   | 短所
-----------------------|---------------------------------------|-------
`Aria`                 | 短い; すでにアクセシビリティと関連している   | ARIAは仕様の名前であり、アクセシビリティツリー内のノードの名前でないため混乱する
`AriaNode`             | 短い; すでにアクセシビリティと関連している   | AOMはARIA属性のみを公開することを暗示していて制限的すぎる
`A11ement`             | 短い; `Element` に近い                  | 読みにくい; 数字が含まれている; 要素と必ずしも関連付けられていない; 分かりにくい
`A11y`                 | 非常に短い; DOMの関連団体について主張しない  | 読みにくい; 数字が含まれている; 分かりにくい
`Accessible`           | 一つの完全な単語である; タイプが難しくない   | 名詞ではない
`AccessibleNode`       | 非常に明確; 読むのが簡単                  | 長い; 混乱する可能性がある (他の `Node` はアクセシブルではない?)
`AccessibleElement`    | 非常に明確                              | さらに長い; 混乱する (他の `Element` はアクセシブルではない?)
`AccessibilityNode`    | 非常に明確                              | 非常に長い; 初めてのときに 'accessibility' を正確にタイプできる人はこの星にいない
`AccessibilityElement` | 非常に明確                              | 笑えるほど長い; まだ 'accessibility' とタイプしないといけない


## Appendix: Partial proposed IDL for virtual accessibility nodes

```IDL
// An AccessibleNode represents a virtual accessible node.
interface AccessibleNode {
    attribute DOMString? role;
    attribute DOMString? name;

    attribute DOMString? autocomplete;
    // ... all other ARIA-equivalent attributes

    // Non-ARIA equivalent attributes necessary for virtual nodes only
    attribute DOMString? offsetLeft;
    attribute DOMString? offsetTop;
    attribute DOMString? offsetWidth;
    attribute DOMString? offsetHeight;
    attribute AccessibleNode? offsetParent;

    // Only affects accessible focus
    boolean focusable;

    // Tree walking
    readonly attribute AccessibleNode? parent;
    readonly attribute ComputedAccessibleNode? firstChild;
    readonly attribute ComputedAccessibleNode? lastChild;
    readonly attribute ComputedAccessibleNode? previousSibling;
    readonly attribute ComputedAccessibleNode? nextSibling;

    // Actions
    void focus();

    // Tree modification
    AccessibleNode insertBefore(AccessibleNode node, Node? child);
    AccessibleNode appendChild(AccessibleNode node);
    AccessibleNode replaceChild(AccessibleNode node, AccessibleNode child);
    AccessibleNode removeChild(AccessibleNode child);
};

```

```idl
partial interface Element {
  AccessibleNode attachAccessibleRoot();
}
```

## Appendix: partial proposed IDL for `ComputedAccessibleNode`

```idl
interface ComputedAccessibleNode {
    // Same set of attributes as AccessibleNode, but read-only
    readonly attribute DOMString? role;
    readonly attribute DOMString? name;

    readonly attribute DOMString? autocomplete;
    // ... all other ARIA-equivalent attributes

    // Non-ARIA equivalent attributes
    readonly attribute DOMString? offsetLeft;
    readonly attribute DOMString? offsetTop;
    readonly attribute DOMString? offsetWidth;
    readonly attribute DOMString? offsetHeight;
    readonly attribute AccessibleNode? offsetParent;
    readonly boolean focusable;

    readonly attribute AccessibleNode? parent;
    readonly attribute ComputedAccessibleNode? firstChild;
    readonly attribute ComputedAccessibleNode? lastChild;
    readonly attribute ComputedAccessibleNode? previousSibling;
    readonly attribute ComputedAccessibleNode? nextSibling;
};

```

```idl
partial interface Window {
  [NewObject] ComputedAccessibleNode getComputedAccessibleNode(Element el);
}
```
