## Can I Use Accessibility Object Model (AOM)

様々なブラウザーにおけるAOMの実装状況を追跡。

## AOMを有効にする方法

**Chrome**:
*`AccessibleNode`/`ComputedAccessibleNode` に関連する機能:*

```--enable-blink-features=AccessibilityObjectModel```

*ウェブプラットフォームに関連する機能:*
`chrome://flags`に遷移し `enable-experimental-web-platform-features` を利用可能にする。

**Safari Technology Preview**:
```開発 > 実験的な機能 > Accessibility Object Model```

**Firefox**:
```about:config accessibility.AOM.enabled = true```

## 概要

最終更新: 2019年7月9日

| | Chrome | Safari | Firefox |
| --- | --- | --- | --- |
| Phase 1: ARIA属性をDOMノードに反映 | **Yes**, experimental-web-platform-features フラグが必要 | **Yes** | **No** |
| Phase 1: IDREF属性に要素の参照を反映 | **No** | **No** | **No** |
| Phase 1: `ElementInternals` 上のカスタム要素のセマンティクス  | **No** | **No** | **No** |
| Phase 2: 支援技術のアクションのフォールバックイベントを生成 | **No** | **No** | **No** |
| Phase 2: 新しい入力イベントタイプ | **No** | **No** | **No** |
| Phase 3: 仮想のアクセシビリティノードを構築 | **Yes**, 古い文法 | **No** | **No** |
| Phase 4: 計算済のアクセシビリティツリーへ問い合わせる | **Yes**, 古い文法 | **No** | **Yes**, 古い文法 |

### フェーズ1: ARIA属性をDOMノードに反映

*Safariで利用可能; Chromeでは `--experimental-web-platform-features` フラグが必要*

```js
element.role = "button";
element.ariaLabel = "Click Me";
```

### フェーズ1: IDREF属性に要素の参照を反映

*未実装*

仕様:
https://whatpr.org/html/3917/common-dom-interfaces.html#reflecting-content-attributes-in-idl-attributes:element

```js
element.ariaActiveDescendantElement = otherElement;
element.ariaLabelledByElements = [ anotherElement, someOtherElement ];
```

### フェーズ1: `ElementInternals` 上のカスタム要素のセマンティクス

*Chromeで現在実装中: https://chromestatus.com/feature/5962105603751936*

仕様: 
https://whatpr.org/html/4658/custom-elements.html#native-accessibility-semantics-map

```js
class CustomTab extends HTMLElement {
  constructor() {
    super();
    this._internals = customElements.createInternals(this);
    this._internals.role = "tab";
  }

  // カスタム "active" 属性を監視
  static get observedAttributes() { return ["active"]; }

  connectedCallback() {
    this._tablist = this.parentElement;
  }

  setTabPanel(tabpanel) {
    if (tabpanel.localName !== "custom-tabpanel" || tabPanel.id === "")
      return;  // 静かに失敗する

    this._tabpanel = tabpanel;
    tabpanel.setTab(this);
    this._internals.ariaControls = tabPanel;    // 反映されない
  }

  // ... setters/getters for custom properties which reflect to attributes

  attributeChangedCallback(name, oldValue, newValue) {
    switch(name) {
      case "active":
        let active = (newValue != null);
        this._tabpanel.shown = active;

        // When the custom "active" attribute changes,
        // keep the accessible "selected" state in sync.
        this._internals.ariaSelected = (newValue !== null);

        if (selected)
          this._tablist.setSelectedTab(this);  // ensure no other tab has "active" set
        break;
    }
  }
}
```

### Phase 2: 支援技術のアクションのフォールバックイベントを生成

(まだ仕様がなく実装されていない)

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

### Phase 2: 新しい入力イベントタイプ

(まだ仕様がなく実装されていない)

```js
customSlider.addEventListener("increment", function(event) {
  customSlider.value += 1;
});

customSlider.addEventListener("decrement", function(event) {
  customSlider.value -= 1;
});
```

### フェーズ3: 仮想のアクセシビリティノードを構築

*Chrome (古い文法, `--enable-blink-features=AccessibilityObjectModel` を適用)*:

```
var listitem = new AccessibleNode();
listitem.role = "listitem";
listitem.offsetParent = list.accessibleNode;
listitem.offsetTop = 32;
listitem.offsetLeft = 0;
listitem.offsetWidth = 200;
listitem.offsetHeight = 16;

// future syntax may be: list.attachAccessibleRoot().appendChild
list.accessibleNode.appendChild(listitem);  
```

### フェーズ4: 計算されたアクセシビリティツリーへ問い合わせる

*Chrome (未確定の記法, `--enable-blink-features=AccessibilityObjectModel` を適用)*:

```
var c = await window.getComputedAccessibleNode(element);
console.log(c.role);
console.log(c.label);
```

*Firefox (古い記法)*:
```
console.log(element.accessibleNode.computedRole);
```
