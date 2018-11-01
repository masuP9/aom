# アクセシビリティオブジェクトモデル

**著者:**

* Alice Boxhall, Google, aboxhall@google.com
* James Craig, Apple, jcraig@apple.com
* Dominic Mazzoni, Google, dmazzoni@google.com
* Alexander Surkov, Mozilla, surkov.alexander@gmail.com

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**

  - [はじめに](#introduction)
  - [モチベーションとなるユースケース](#motivating-use-cases)
  - [アクセシビリティオブジェクトモデル](#the-accessibility-object-model)
    - [ARIA属性を反映する](#reflecting-aria-attributes)
    - [要素の参照を反映する](#reflecting-element-references)
      - [Use case 2: Setting relationship properties without needing to use IDREFs](#use-case-2-setting-relationship-properties-without-needing-to-use-idrefs)
    - [Custom ElementsのAPI](#custom-elements-apis)
      - [ユースケース1: 非反映のデフォルトアクセシビリティプロパティをウェブコンポーネントに設定する](#use-case-1-setting-non-reflected-default-accessibility-properties-for-web-components)
        - [customElements.define() を利用したデフォルトセマンティクス](#default-semantics-via-customelementsdefine)
        - [`ElementInternals` オブジェクトを利用した動的なインスタンス単位のセマンティクス](#per-instance-dynamic-semantics-via-the-createdcallback-reaction)
    - [User action events from Assistive Technology](#user-action-events-from-assistive-technology)
      - [New InputEvent types](#new-inputevent-types)
      - [Use case 3: Listening for events from Assistive Technology](#use-case-3-listening-for-events-from-assistive-technology)
    - [Virtual Accessibility Nodes](#virtual-accessibility-nodes)
      - [Use case 4: Adding non-DOM nodes (“virtual nodes”) to the Accessibility tree](#use-case-4-adding-non-dom-nodes-virtual-nodes-to-the-accessibility-tree)
    - [Full Introspection of an Accessibility Tree - `ComputedAccessibleNode`](#full-introspection-of-an-accessibility-tree---computedaccessiblenode)
      - [Use case 5:  Introspecting the computed tree](#use-case-5--introspecting-the-computed-tree)
      - [Why is accessing the computed properties being addressed last?](#why-is-accessing-the-computed-properties-being-addressed-last)
    - [Audience for the proposed API](#audience-for-the-proposed-api)
    - [`AccessibleNode`に何が起こったのか?](#what-happened-to-accessiblenode)
  - [Next Steps](#next-steps)
    - [Incubation](#incubation)
  - [Additional thanks](#additional-thanks)
- [付録](#appendices)
  - [Background: assistive technology and the accessibility tree](#background-assistive-technology-and-the-accessibility-tree)
    - [Accessibility node properties](#accessibility-node-properties)
  - [Background: DOM tree, accessibility tree and platform accessibility APIs](#background-dom-tree-accessibility-tree-and-platform-accessibility-apis)
    - [Mapping native HTML to the accessibility tree](#mapping-native-html-to-the-accessibility-tree)
    - [ARIA](#aria)
  - [Appendix: `AccessibleNode` naming](#appendix-accessiblenode-naming)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## はじめに

このJavaScript APIを作る努力の目的は、開発者にHTMLページのアクセシビリティツリーの変更（ついには探索まで）を許可するためである。

## モチベーションとなるユースケース

既存のAPIの背景は[付録](#appendices)で見つけることができる。

ウェブ上でできることの境界を押し広げているウェブアプリケーションは、APIが不十分なためそれらをアクセシブルにするため苦闘している。特に、ブラウザとやりとりするネイティブAPIと比較して表現力に劣っている。

1. ページの著者が上書きできる[ウェブコンポーネント](https://developer.mozilla.org/en-US/docs/Web/Web_Components)のデフォルトアクセシビリティプロパティの設定
    - 現在、ウェブコンポーネントのデフォルトセマンティクスを定義するにはARIAを使用しなければならない。
    これにより、真に詳細な実装であるARIA属性がDOMに「リーク」してしまう。
    - このことは必要では _なく_ ウェブコンポーネントに限られる _かも_ 知れない。
2. IDREFs を必要としない[関係属性](https://www.w3.org/TR/wai-aria-1.1/#attrs_relationships)の設定
    - 現在、いくつかのARIAの関係を示すには、著者が一意のIDを関係の対象となりうる要素に指定しなければならない
    - [`aria-activedescendant`](https://www.w3.org/TR/wai-aria-1.1/#aria-activedescendant)のような場合、
    それはUIに応じて、数百ないしは数千もの要素のうち、一つを参照するかもしれない。
    この要求は、多くの余分なDOMの属性が必要となり、これらのAPIを複雑にする。
3. 支援技術からのイベントをリスニングする
   - 現在、組み込み要素 _だけ_ がイベントに反応することができ、
   通常、["simulated click"](https://developer.android.com/reference/android/view/accessibility/AccessibilityEvent.html#TYPE_VIEW_CLICKED) や ["increment"](https://developer.apple.com/documentation/objectivec/nsobject/1615076-accessibilityincrement) のようなユーザーアクションによって引き起こされる。
4. アクセシビリティツリーにDOMでないノード（仮想ノード）を追加する
   - 例えば、`<canvas>` 要素で構築された複雑なUIや、`<video>` 要素を用いたリモートデスクトップのストリーミングなどを表現するなど
   - そのためには、少なくとも要素と同じようなアクセシビリティプロパティや、他の仮想ノードとの親/子/その他の関係性、位置や次元を表す必要がある。
5. 計算済のアクセシビリティツリーの確認
   - 開発者は現在、ARIAやその他のアクセシブルプロパティがどのように適用されているかを調べたりテストする方法を持っていない。

## アクセシビリティオブジェクトモデル

アクセシビリティオブジェクトモデル (AOM)は、上記のユースケースに取り組むためのHTMLや関連する標準に対する一連の変更である。

(注: 以前のバージョンのAOMに慣れ親しんでいれば、[`AccessibleNode`に何が起こったのか?](#what-happened-to-accessiblenode))と疑問を持つかもしれない。

### ARIA属性を反映する

ARIA 属性をHTML要素に[反映する](https://html.spec.whatwg.org/multipage/common-dom-interfaces.html#reflect)。

これは今[ARIA 1.2の仕様](https://www.w3.org/TR/wai-aria-1.2/#idl-interface)の一部となった。

```js
el.role = "button";
el.ariaPressed = "true";  // aria-pressed は3つのステートを持つ属性
el.ariaDisabled = true;   // aria-disabled は true/false を持つ属性
```

### 要素の参照を反映する

ARIAプロパティを直接反映、`aria-labelledby` のような関係属性の文字列に反映させる。

```js
el.ariaDescribedBy = "id1";
```

結果は

```html
<div aria-describedby="id1">
```

要素を参照する非反映のプロパティでAPIを拡張することを提案

```js
el.ariaDescribedByElements = [labelElement1, labelElement2];
el.ariaActiveDescendantElement = ownedElement1;
```

これにより、関係に属するそれぞれの要素にグローバルに固有のID属性を割り当てることなく要素間のセマンティクスの関係性を示すことができる。

さらにこれは、`ShadowRoot`を使用した著者がShadow DOMの境界を超えて関係性を明示することを可能にする

このAPIはWHATWG HTML仕様の変更として提案されている。

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

著者は代わりにこの関係性をプログラムで示すことができる。

```js
const input = comboBox.shadowRoot.querySelector("input");
const optionList = comboBox.querySelector("custom-optionlist");
input.activeDescendantElement = optionList.firstChild;
```

This would allow the relationship to be expressed naturally.

このことにより、関係が自然に表現されるようになる。

このAPIはWHATWG HTMLリポジトリの[issue #3513](https://github.com/whatwg/html/issues/3515#issuecomment-413716944)で議論されている。

### Custom ElementsのAPI

Custom Elementの著者が `customElements.define()` オプションを用いて静的なデフォルトセマンティクス、または設定されたコールバックを用いて動的に要素ごとのセマンティクスを提供できることを提案する。

#### ユースケース1: 非反映のデフォルトアクセシビリティプロパティをウェブコンポーネントに設定する

今、ウェブコンポーネントを制作するライブラリの著者はネイティブ要素にとっては暗黙であるセマンティクスを表すためにARIA属性を"生やす"ことを強いられている。

```html
<!-- ページの著者はCustom elementをネイティブ要素を使用するように使用する -->
<custom-tablist>
  <custom-tab selected>Tab 1</custom-tab>
  <custom-tab>Tab 2</custom-tab>
  <custom-tab>Tab 3</custom-tab>
</custom-tablist>

<!-- Custom elements がセマンティクスを表すために余分な属性を"生やす"ことを強制される -->
<custom-tablist role="tablist">
  <custom-tab selected role="tab" aria-selected="true" aria-controls="tabpanel-1">Tab 1</custom-tab>
  <custom-tab role="tab" aria-controls="tabpanel-2">Tab 2</custom-tab>
  <custom-tab role="tab" aria-controle="tabpanel-3">Tab 3</custom-tab>
</custom-tablist>
```

##### customElements.define() を利用したデフォルトセマンティクス

著者は `CustomElementRegistry.define()` メソッドに渡された `ElementDefinitionOptions` オブジェクトを利用して、不変のデフォルトセマンティクスをカスタム要素に提供することもできる。

`ElementDefinitionOptions` オブジェクトに設定されたプロパティは、カスタム要素をアクセシブルなオブジェクトにマッピングする際に、デフォルトの値として利用される。

注: これは "不変なクラス変数" を作るのに類似している。これらのセマンティクスプロパティはカスタム要素の定義に関連付けられていて、カスタム要素のインスタンスには関連していない。

定義されたセマンティクスは *すべての* カスタム要素のインスタンスに適用される。

例えば、カスタムタブコントロールを作成する著者は、タブ、タブリスト、タブパネルの3つのカスタム要素を個々に定義することができる。

```js
class TabListElement extends HTMLElement { ... }
customElements.define("custom-tablist", TabListElement,
                      { role: "tablist", ariaOrientation: "horizontal" });

class TabElement extends HTMLElement { ... }
customElements.define("custom-tab", TabElement,
                      { role: "tab" });

class TabPanelElement extends HTMLElement { ... }
customElements.define("custom-tabpanel", TabPanelElement,
                      { role: "tabpanel" });
```

`<custom-tab>`がアクセシビリティツリーにマッピングされるとき、タブのロールがデフォルトでマッピングされる。

これは `button` 要素がデフォルトでボタンのロールを持ってアクセシビリティオブジェクトにマッピングされるのに似ている。

##### `ElementInternals` オブジェクトを利用した動的なインスタンス単位のセマンティクス

これは[W3C Web Componentsプロジェクトの一部として議論されている](https://github.com/w3c/webcomponents/issues/758).

カスタム要素の著者は `ElementInternals` オブジェクトを、ユーザーインタラクションに応じてカスタム要素のインスタンスのセマンティクスの状態を変更するのに利用できる。

`ElementInternals` オブジェクトにセットされたプロパティはアクセシビリティオブジェクトに要素をマッピングされる際に利用される。

注: これは "インスタンス変数" を設定するのに類似している。セマンティクスプロパティのコピーはカスタム要素のインスタンス毎に作られる。それぞれに定義されたセマンティクスは関連するカスタム要素のインスタンスオブジェクトにのみ関連付けられる。

```js
class CustomTab extends HTMLElement {
  #internals = null;
  #tablist = null;
  #tabpanel = null;

  constructor() {
    super();
    this.#internals = customElements.createInternals(this);
    this.#internals.role = "tab";
  }

  // カスタム "active" 属性を監視する
  static get observedAttributes() { return ["active"]; }

  connectedCallback() {
    this.#tablist = this.parentElement;
  }

  setTabPanel(tabpanel) {
    if (tabpanel.localName !== "custom-tabpanel" || tabPanel.id === "")
      return;  // 静かに失敗する

    this.#tabpanel = tabpanel;
    tabpanel.setTab(this);
    this.#internals.ariaControls = tabPanel;    // 反映されない
  }

  // 属性に反映するカスタムプロパティの setters/getters

  attributeChangedCallback(name, oldValue, newValue) {
    switch(name) {
      case "active":
        let active = (newValue != null);
        this.#tabpanel.shown = active;

        // カスタム "active" 属性が変更された時、
        // アクセシブルな "selected" ステートを同期し続ける
        this.#internals.ariaSelected = (newValue !== null);

        if (selected)
          this.#tablist.setSelectedTab(this);  // 他のタブが "active" で無いことを保証
        break;
    }
  }
}

customElements.define("custom-tab", CustomTab, { role: "tab", needsElementInternals: true });
```

これらの要素を使用する著者は通常通りARIAを用いてデフォルトセマンティクスを上書きすることができる。

例えば、著者は `<custom-tablist>` 要素の見た目を縦並びに変更できる。彼らはそれを示すために `aria-orientation` 属性を追加することで、カスタム要素に定義されているデフォルトセマンティクスを上書きすることができる。

```html
<custom-tablist aria-orientation="vertical" class="vertical-tablist">
  <custom-tab selected>Tab 1</custom-tab>
  <custom-tab>Tab 2</custom-tab>
  <custom-tab>Tab 3</custom-tab>
</div>
```

著者が提供するロールがデフォルトのロールを上書きするので、それぞれの場合においてマッピングされるロールは著者が提供するロールに基づく。

仮に著者によって提供されたセマンティクスが Custom Element のセマンティクスと競合する場合でも著者が提供するセマンティクスが優先される。

### User action events from Assistive Technology

To preserve the privacy of assistive technology users,
events from assistive technology will typically cause a synthesised DOM event to be triggered:

| **AT event**     | **Targets**                                                                        | **DOM event**                                                                   |
|------------------|------------------------------------------------------------------------------------|---------------------------------------------------------------------------------|
| `click`          | *all elements*                                                                     | `click`                                                                         |
| `focus`          | *all elements*                                                                     | `focus`                                                                         |
| `select`         | Elements whose mapped role is `cell` or `option`                                   | `click`                                                                         |
| `scrollIntoView` | (n/a)                                                                              | No event                                                                        |
| `dismiss`        | *all elements*                                                                     | Keypress sequence for `Escape` key                                              |
| `contextMenu`    | *all elements*                                                                     | `contextmenu`                                                                   |
| `scrollByPage`   | *all elements*                                                                     | Keypress sequence for `PageUp` or `PageDown` key, depending on scroll direction |
| `increment`      | Elements whose mapped role is `progressbar`, `scrollbar`, `slider` or `spinbutton` | Keypress sequence for `Up` key                                                  |
| `decrement`      | Elements whose mapped role is `progressbar`, `scrollbar`, `slider` or `spinbutton` | Keypress sequence for `Down` key                                                |
| `setValue`       | Elements whose mapped role is `combobox`,`scrollbar`,`slider` or `textbox`         | TBD                                                                             |

#### New InputEvent types

We will also add some new [`InputEvent`](https://www.w3.org/TR/uievents/#inputevent) types:

* `increment`
* `decrement`
* `dismiss`
* `scrollPageUp`
* `scrollPageDown`

These will be triggered via assistive technology events,
along with the synthesised keyboard events listed in the above table,
and also synthesised when the keyboard events listed above
occur in the context of a valid target for the corresponding assistive technology event.

For example,
if a user not using assistive technology presses the `Escape` key in any context,
an `input` event with a type of `dismiss` will be fired at the focused element
along with the keypress sequence.

If the same user pressed `Up` while page focus was on
a `<input type="range">` *or* an element with a role of `slider`
(either of which will have a computed role of `slider`),
an `input` event with a type of `increment` will be fired at the focused element
along with the keypress sequence.

#### Use case 3: Listening for events from Assistive Technology

For example:

* A user may be using voice control software and they may speak the name of a
  button somewhere in a web page.
  The voice control software finds the button matching that name in the
  accessibility tree and sends an *action* to the browser to click on that button.
* That same user may then issue a voice command to scroll down by one page.
  The voice control software finds the root element for the web page and sends
  it the scroll *action*.
* A mobile screen reader user may navigate to a slider, then perform a gesture to
  increment a range-based control.
  The screen reader sends the browser an increment *action* on the slider element
  in the accessibility tree.

Currently, browsers implement partial support for accessible actions 
either by implementing built-in support for native HTML elements 
(for example, a native HTML `<input type="range">` 
already supports increment and decrement actions,
and text boxes already support actions to set the value or insert text).

However, there is no way for web authors to listen to accessible actions on
custom elements.  
For example, the 
[custom slider above with a role of `slider`](#use-case-1-setting-non-reflected-default-accessibility-properties-for-web-components)
prompts a suggestion on VoiceOver for iOS 
to perform swipe gestures to increment or decrement, 
but there is no way to handle that semantic event via any web API.

Developers will be able to listen for keyboard events
or input events to capture that semantic event.

For example, to implement a custom slider,
the author could handle the `Up` and `Down` key events
as recommended in the [ARIA Authoring Practices guide](https://www.w3.org/TR/wai-aria-practices-1.1/#slider_kbd_interaction),
and this would handle the assistive technology event as well.

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

### Virtual Accessibility Nodes

**Virtual Accessibility Nodes** will allow authors
to expose "virtual" accessibility nodes,
which are not associated directly with any particular DOM element,
to assistive technology.

This mechanism is often present in native accessibility APIs,
in order to allow authors more granular control over the accessibility
of custom-drawn APIs.

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

- Calling `attachAccessibleRoot()` causes an `AccessibleNode` to be associated with a `Node`.
  - The returned `AccessibleNode` forms the root of a virtual accessibility tree.
  - The Node's DOM children are implicitly ignored for accessibility once an `AccessibleRoot` is attached - there is no mixing of DOM children and virtual accessible nodes.
- Like `ShadowRoot`, an element may only have one associated `AccessibleRoot`.
- Only `AccessibleNode`s may have `AccessibleNodes` as children, 
  and `AccessibleNode`s may only have `AccessibleNode`s as children.

#### Use case 4: Adding non-DOM nodes (“virtual nodes”) to the Accessibility tree 

For example, to express a complex UI built out of a `<canvas>` element:

```js
// Implementing a canvas-based spreadsheet's semantics
canvas.attachAccessibleRoot();
let table = canvas.accessibleRoot.appendChild(new AccessibleNode());
table.role = 'table';
table.colCount = 10;
table.rowcount = 100;
let headerRow = table.appendChild(new AccessibleNode());
headerRow.role = 'row';
headerRow.rowindex = 0;
// etc. etc.
```

Virtual nodes will typically need to have location and dimensions set explicitly:

```js
cell.offsetLeft = "30px";
cell.offsetTop = "20px";
cell.offsetWidth = "400px";
cell.offsetHeight = "300px";
cell.offsetParent = table;
```

If offsetParent is left unset, 
the coordinates are interpreted relative to the accessible node's parent.

To make a node focusable, the `focusable` attribute can be set. 
This is similar to setting tabIndex=-1 on a DOM element.

```js
virtualNode.focusable = true;
```

Virtual accessible nodes are not focusable by default.

Finally, to focus an accessible node, call its focus() method.

```js
virtualNode.focus();
```

When a virtual accessible node is focused, 
input focus in the DOM is unchanged. 
The focused accessible node is reported to assistive technology
and other accessibility API clients, 
but no DOM events are fired and document.activeElement is unchanged.

When the focused DOM element changes, accessible focus follows it:
the DOM element's associated accessible node gets focused.

### Full Introspection of an Accessibility Tree - `ComputedAccessibleNode`

```idl
partial interface Window {
  [NewObject] ComputedAccessibleNode getComputedAccessibleNode(Element el);
}
```

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

#### Use case 5:  Introspecting the computed tree

The **Computed Accessibility Tree** API will allow authors to access
the full computed accessibility tree -
all computed properties for the accessibility node associated with each DOM element,
plus the ability to walk the computed tree structure including virtual nodes.

This will make it possible to:
  * write any programmatic test which asserts anything
    about the semantic properties of an element or a page.
  * build a reliable browser-based assistive technology -
    for example, a browser extension which uses the accessibility tree
    to implement a screen reader, screen magnifier, or other assistive functionality;
    or an in-page tool.
  * detect whether an accessibility property
    has been successfully applied
    (via ARIA or otherwise)
    to an element -
    for example, to detect whether a browser has implemented a particular version of ARIA.
  * do any kind of console-based debugging/checking of accessibility tree issues.
  * react to accessibility tree state,
    for example, detecting the exposed role of an element
    and modifying the accessible help text to suit.

#### Why is accessing the computed properties being addressed last?

**Consistency**
Currently, the accessibility tree is not standardized between browsers:
Each implements accessibility tree computation slightly differently.
In order for this API to be useful,
it needs to work consistently across browsers,
so that developers don't need to write special case code for each.

We want to take the appropriate time to ensure we can agree
on the details for how the tree should be computed
and represented.

**Performance**
Computing the value of many accessible properties requires layout.
Allowing web authors to query the computed value of an accessible property
synchronously via a simple property access
would introduce confusing performance bottlenecks.

We will likely want to create an asynchronous mechanism for this reason,
meaning that it will not be part of the `accessibleNode` interface.

**User experience**
Compared to the previous three phases,
accessing the computed accessibility tree will have the least direct impact on users.
In the spirit of the [Priority of Constituencies](https://www.w3.org/TR/html-design-principles/#priority-of-constituencies),
it makes sense to tackle this work last.

### Audience for the proposed API

This API is will be primarily of interest to
the relatively small number of developers who create and maintain
the JavaScript frameworks and widget libraries that power the vast majority of web apps.
Accessibility is a key goal of most of these frameworks and libraries,
as they need to be usable in as broad a variety of contexts as possible.
A low-level API would allow them to work around bugs and limitations
and provide a clean high-level interface that "just works" 
for the developers who use their components.

This API is also aimed at developers of large flagship web apps that
push the boundaries of the web platform. 
These apps tend to have large development teams 
who look for unique opportunities to improve performance 
using low-level APIs like Canvas. 
These development teams have the resources to make accessibility a priority too, 
but existing APIs make it very cumbersome.

### `AccessibleNode`に何が起こったのか?

Initially, our intention was to combine these use cases into a read/write API
analogous to the DOM,
wherein each DOM `Element` would have an associated `AccessibleNode`
allowing authors to read and write accessible properties.
This was named the Accessibility Object Model,
analogous to the Document Object Model.

However, as discussions progressed it became clear that there were some issues with this model:
- Computing the accessibility tree should not be necessary in order to modify it -
getting an `AccessibleNode` to write to 
should thus not depend on getting the computed properties.
- Exposing the computed accessibility tree requires standardisation across browsers
of how the tree is computed.
- If we are not exposing computed properties on an `Element`'s `AccessibleNode`,
it's unclear what the purpose of this object is beyond a "bag of properties".
- Determining the order of precedence of ARIA properties
and `AccessibleNode` properties did not have an obvious "correct" answer.
- Similarly, using exclusively `AccessibleNode`s to express relationships was confusing.

These issues prompted a reassessment, 
and a simplification of the API based around the original set of use cases
we were committed to addressing.

## Next Steps

The Accessibility Object Model development is led by a team of editors
that represent several major browser vendors.

Issues can be filed on GitHub:

https://github.com/WICG/aom/issues

### Incubation

We intend to continue development of this spec as part of the
[Web Platform Incubator Community Group (WICG)](https://www.w3.org/community/wicg/).
Over time it may move into its own community group.

Our intent is for this group's work to be almost entirely orthogonal to the
current work of the [Web Accessibility Initiative](https://www.w3.org/WAI/)
groups such as [ARIA](https://www.w3.org/TR/wai-aria/). While ARIA defines
structural markup and semantics for accessibility properties on the web,
often requiring coordination with assistive technology vendors and native platform
APIs, the AOM simply provides a parallel JavaScript API that provides
more low-level control for developers and fills in gaps in the web platform,
but without introducing any new semantics.

## Additional thanks

Many thanks for valuable feedback, advice, and tools from:

* Alex Russell
* Bogdan Brinza
* Chris Fleizach
* Cynthia Shelley
* David Bolter
* Domenic Denicola
* Elliott Sprehn
* Ian Hickson
* Joanmarie Diggs
* Marcos Caceres
* Nan Wang
* Robin Berjon
* Tess O'Connor

Bogdan Brinza and Cynthia Shelley of Microsoft were credited as authors of an
earlier draft of this spec but are no longer actively participating.

# 付録

## Background: assistive technology and the accessibility tree

Assistive technology, in this context, refers to a third party application
which augments or replaces the existing UI for an application.
One well-known example is a screen reader,
which replaces the visual UI and pointer-based UI
with an auditory output (speech and tones)
and a keyboard and/or gesture-based input mechanism.

Many assistive technologies interact with a web page via accessibility APIs, such as
[UIAutomation](https://msdn.microsoft.com/en-us/library/windows/desktop/ee684009.aspx)
on Windows, or
[NSAccessibility](https://developer.apple.com/library/mac/documentation/AppKit/Reference/NSAccessibility_Protocol_Reference/)
on OS X.
These APIs allow an application to expose a tree of objects representing the application's interface,
typically with the root node representing the application window,
with various levels of grouping node descendants down to individual interactive elements.
This is referred to as the **accessibility tree**.

An assistive technology user interacts with the application almost exclusively via this API,
as the assistive technology uses it both to create the alternative interface,
and to route user interaction events triggered by the user's commands to the assistive technology.

![Flow from application UI to accessibility tree to assistive technology to user](images/a11y-tree.png)

Both the alternative interface's *output*
(e.g. speech and tones,
updating a [braille display](https://en.wikipedia.org/wiki/Refreshable_braille_display),
moving a [screen magnifier's](https://en.wikipedia.org/wiki/Screen_magnifier) focus)
and *input*
(e.g. keyboard shortcuts, gestures, braille routing keys,
[switch devices](https://en.wikipedia.org/wiki/Switch_access), voice input)
are completely the responsibility of the assistive technology,
and are abstracted away from the application.

For example, a [VoiceOver](https://www.apple.com/voiceover/info/guide/) user
interacting with a native application on OS X
might press the key combination
"Control Option Spacebar" to indicate that they wish to click the UI element which the screen reader is currently visiting.

![A full round trip from UI element to accessibility node to assistive technology to user to user keypress to accessibility API action method back to UI element](images/a11y-tree-example.png)

These keypresses would never be passed to the application,
but would be interpreted by the screen reader,
which would then call the
[`accessibilityPerformPress()`](https://developer.apple.com/reference/appkit/nsaccessibilitybutton/1525542-accessibilityperformpress?language=objc)
function on the accessibility node representing the UI element in question.
The application can then handle the press action;
typically, this routes to the code which would handle a click event.

Accessibility APIs are also popular for testing and automation.
They provide a way to examine an application's state and manipulate its UI from out-of-process,
in a robust and comprehensive way.
While assistive technology for users with disabilities
is typically the primary motivator for accessibility APIs,
it's important to understand that these APIs are quite general
and have many other uses.

### Accessibility node properties

Each node in the accessibility tree may be referred to as an **accessibility node**.
An accessibility node always has a **role**, indicating its semantic purpose.
This may be a grouping role,
indicating that this node merely exists to contain and group other nodes,
or it may be an interactive role,
such as `"button"`.

![Accessibility nodes in an accessibility tree, showing roles, names, states and properties](images/a11y-node.png)

The user, via assistive technology, may explore the accessibility tree at various levels.
They may interact with grouping nodes,
such as a landmark element which helps a user navigate sections of the page,
or they may interact with interactive nodes,
such as a button.
In both of these cases,
the node will usually need to have a **label** (often referred to as a **name**)
to indicate the node's purpose in context.
For example, a button may have a label of "Ok" or "Menu".

Accessibility nodes may also have other properties,
such as the current **value**
(e.g. `"10"` for a range, or `"Jane"` for a text input),
or **state** information
(e.g. `"checked"` for a checkbox, or `"focused"`).

Interactive accessibility nodes may also have certain **actions** which may be performed on them.
For example, a button may expose a `"press"` action, and a slider may expose
`"increment"` and `"decrement"` actions.

These properties and actions are referred to as the *semantics* of a node.
Each accessibility API expresses these concepts slightly differently,
but they are all conceptually similar.

##  Background: DOM tree, accessibility tree and platform accessibility APIs

The web has rich support for making applications accessible,
but only via a *declarative* API.

The DOM tree is translated, in parallel,
into the primary, visual representation of the page,
and the accessibility tree,
which is in turn accessed via one or more *platform-specific* accessibility APIs.

![HTML translated into DOM tree translated into visual UI and accessibility tree](images/DOM-a11y-tree.png)

Some browsers support multiple accessibility APIs across different platforms,
while others are specific to one accessibility API.
However, any browser that supports at least one native accessibility API
has some mechanism for exposing a tree structure of semantic information.
We refer to that mechanism, regardless of implementation details,
as the **accessibility tree** for the purposes of this API.

### Mapping native HTML to the accessibility tree

Native HTML elements are implicitly mapped to accessibility APIs.
For example, an  `<img>` element will automatically be mapped
to an accessibility node with a role of `"image"`
and a label based on the `alt` attribute (if present).

![<img> node translated into an image on the page and an accessibility node](images/a11y-node-img.png)


### ARIA

Alternatively, [ARIA](https://www.w3.org/TR/wai-aria-1.1/)
allows developers to annotate elements with attributes to override
the default role and semantic properties of an element -
but not to expose any accessible actions.

![<div role=checkbox aria-checked=true> translated into a visual presentation and a DOM node](images/a11y-node-ARIA.png)

In either case there's a one-to-one correspondence
between a DOM node and a node in the accessibility tree,
and there is minimal fine-grained control over the semantics of the corresponding accessibility node.

## Appendix: `AccessibleNode` naming

We have chosen the name `AccessibleNode` for the class representing one
node in the virtual accessibility tree.

In choosing this name, we have tried to pick a balance between brevity,
clarity, and generality.

* Brevity: The name should be as short as possible.
* Clarity: The name should reflect the function of the API,
  without using opaque abbreviations or contractions.
* Generality: The name should not be too narrow and limit the scope of the spec.

Below we've collected all of the serious names that have been proposed
and a concise summary of the pros and cons of each.

Suggestions for alternate names or votes for one of the other names below
are still welcome, but please try to carefully consider the existing suggestions and
their cons first. Rough consensus has already been achieved and we'd rather work
on shipping something we can all live with rather than trying to get the perfect
name.

Proposed name          | Pros                                                      | Cons
-----------------------|-----------------------------------------------------------|-------
`Aria`                 | Short; already associated with accessibility              | Confusing because ARIA is the name of a spec, not the name of one node in an accessibility tree.
`AriaNode`             | Short; already associated with accessibility              | Implies the AOM will only expose ARIA attributes, which is too limiting
`A11ement`             | Short; close to `Element`                                 | Hard to pronounce; contains numbers; not necessarily associated with an element; hard to understand meaning
`A11y`                 | Very short; doesn't make assertions about DOM association | Hard to pronounce; contains numbers; hard to understand meaning
`Accessible`           | One full word; not too hard to type                       | Not a noun
`AccessibleNode`       | Very explicit; not too hard to read                       | Long; possibly confusing (are other `Node`s not accessible?)
`AccessibleElement`    | Very explicit                                             | Even longer; confusing (are other `Element`s not accessible?)
`AccessibilityNode`    | Very explicit                                             | Extremely long; nobody on the planet can type 'accessibility' correctly first try
`AccessibilityElement` | Very explicit                                             | Ludicrously long; still requires typing 'accessibility'
