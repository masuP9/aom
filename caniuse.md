## Can I Use Accessibility Object Model (AOM)

様々なブラウザーにおけるAOMの実装状況を追跡。

## AOMを有効にする方法

*Chrome*: `--enable-blink-features=AccessibilityObjectModel`

*Safari Technology Preview*: `Develop > Experimental Features > Accessibility Object Model`

*Firefox*: `about:config accessibility.AOM.enabled = true`

## 概要

Last updated: 2018年5月8日

| | Chrome | Safari | Firefox |
| --- | --- | --- | --- |
| フェーズ1: ARIA属性をDOMノードに反映 | **はい**、古い構文 | **いいえ** | **いいえ** |
| フェーズ2: ATからの入力イベントをリッスンする | **はい**、古い構文、6イベント | **はい**、8イベント | **いいえ** |
| フェーズ3: 仮想のアクセシビリティノードを構築 | **はい**、古い構文 | **いいえ** | **いいえ** |
| フェーズ4: 計算されたアクセシビリティツリーへ問い合わせる | **はい**、古い構文 | **いいえ** | **はい**、古い構文 |

### フェーズ1: ARIA属性をDOMノードに反映

*Chrome*:

```
element.accessibleNode.role = "button";
element.accessibleNode.label = "Click Me";
element.accessibleNode.labeledBy = new AccessibleNodeList();
element.accessibleNode.labeledBy.add(other.accessibleNode);
...
```

### フェーズ2: ATからの入力イベントをリッスンする

*Chrome*:

```
element.accessibleNode.addEventListener('accessibleclick', ...);
element.accessibleNode.addEventListener('accessiblecontextmenu', ...);
element.accessibleNode.addEventListener('accessiblefocus', ...);
element.accessibleNode.addEventListener('accessiblescrollintoview', ...);
element.accessibleNode.addEventListener('accessibleincrement', ...);
element.accessibleNode.addEventListener('accessibledecrement', ...);
```

*Safari Technology Preview*

```
element.addEventListener('accessibleclick', ...);
element.addEventListener('accessiblecontextmenu', ...);
element.addEventListener('accessiblefocus', ...);
element.addEventListener('accessiblescrollintoview', ...);
element.addEventListener('accessibleincrement', ...);
element.addEventListener('accessibledecrement', ...);
element.addEventListener('accessibledismiss', ...);
element.addEventListener('accessiblesetvalue', ...);
```

### フェーズ3: 仮想のアクセシビリティノードを構築

*Chrome*:

```
var listitem = new AccessibleNode();
listitem.role = "listitem";
listitem.offsetParent = list.accessibleNode;
listitem.offsetTop = 32;
listitem.offsetLeft = 0;
listitem.offsetWidth = 200;
listitem.offsetHeight = 16;
list.accessibleNode.appendChild(listitem);
```

### フェーズ4: 計算されたアクセシビリティツリーへ問い合わせる

*Chrome*:

```
var c = await window.getComputedAccessibleNode(element);
console.log(c.role);
console.log(c.label);
```

*Firefox*:
```
console.log(element.accessibleNode.role);
console.log(element.accessibleNode.label);
```
