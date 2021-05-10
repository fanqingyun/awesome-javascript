# DOM

文档对象模型（DOM，Document Object Model）是 HTML 和 XML 文档的编程接口。DOM 表示由多层节点构成的文档，通过它开发者可以添加、删除和修改页面的各个部分。DOM 现在是真正跨平台、语言无关的表示和操作网页的方式

## 节点层级

`任何 HTML 或 XML 文档都可以用 DOM 表示为一个由节点构成的层级结构。节点分很多类型，每种类型对应着文档中不同的信息和（或）标记，也都有自己不同的特性、数据和方法，而且与其他类型有某种关系。这些关系构成了层级，让标记可以表示为一个以特定节点为根的树形结构`

对这句话理解是,html可以用DOM来表示成一棵具有层级关系的树,这颗树上有不同类型的节点,而这些节点有它们各自的特性,数据和方法.

在了解节点有哪些类型之前,要知道,其中，document 节点表示每个文档的`根节点`。在这里，根节点的唯一子节点是<html>元素，我们称之为`文档元素`（`documentElement`）。文档元素是文档最外层的元素，所有其他元素都存在于这个元素之内。每个文档只能有一个文档元素。在 HTML 页面中，文档元素始终是<html>元素。在 XML 文档中，则没有这样预定义的元素，任何元素都可能成为文档元素。

HTML 中的每段标记都可以表示为这个树形结构中的一个节点。`元素节点`表示 HTML 元素，`属性节点`表示属性，`文档类型节点`表示文档类型，`注释节点`表示注释。DOM 中总共有 12 种节点类型，这些类型都继承一种基本类型: **Node类型**,因此用于Node类型的属性和方法

### Node类型

1. 属性:

- nodeType 是一个数值,Node类内置了12种节点类型nodeType值的常量
 Node.ELEMENT_NODE（1）
 Node.ATTRIBUTE_NODE（2）
 Node.TEXT_NODE（3）
 Node.CDATA_SECTION_NODE（4）
 Node.ENTITY_REFERENCE_NODE（5）
 Node.ENTITY_NODE（6）
 Node.PROCESSING_INSTRUCTION_NODE（7）
 Node.COMMENT_NODE（8）
 Node.DOCUMENT_NODE（9）
 Node.DOCUMENT_TYPE_NODE（10）
 Node.DOCUMENT_FRAGMENT_NODE（11）
 Node.NOTATION_NODE（12）

```javascript
  console.log(Node.ELEMENT_NODE) // 1
  var div = document.createElement('div')
  console.log(div.nodeType) // 1
```

- nodeName
- nodeValue
- childNodes, 包含一个 NodeList 的实例,子节点
- parentNode, 父节点
- previousSibling 和 nextSibling, 兄弟节点
- firstChild 和 lastChild 分别指向一个元素的childNodes的第一个元素和最后一个元素
- ownerDocument, 一个指向自己所在文档的指针

2. 方法:

- hasChildNodes() 是否拥有子节点
- 插入节点: insertBefore()与 appendChild()
- 替换节点: replaceChild()方法接收两个参数：要插入的节点和要替换的节点
- 删除节点: removeChild()
- 复制节点: cloneNode() 传入true表示深度复制, 包括子节点.如果不传或传入false,只复制自己
- normalize() 如果发现空文本节点，则将其删除；如果两个同胞节点是相邻的，则将其合并为一个文本节点

了解了Node这个基本类型,接下来让我们去了解它的"继承者" 们

| 节点类型     |  nodeType    |   nodeName   | nodeValue|
| ---- | ---- | ---- | ---- |
| Document     |    9  |  #document    | null |
|Element      |  1    | 元素的标签名     |      null     |
| Text     |  3    |  #text    |   文本内容    |
| Comment     |  8   |  #comment   |   注释的内容    |
| DocumentType     |  10   |  文档类型的名称   |   null    |
| CDATASection     |  4   |   #cdata-section    |  CDATA 区块的内容   |
| DocumentFragment     |  11   |   #document-fragment    |  null  |
| Attr     |  2   |   属性名   |  属性值  |

### Document

是HTMLDocument的实例

```javascript
document.__proto__   === HTMLDocument.prototype // true
```

- documentElement属性，指向<html>元素
- body 属性，直接指向<body>元素
- 所有主流浏览器都支持 document.documentElement 和 document.body。
- document.doctype 指向 文档声明元素
- URL 指向当前页面的网址，同location.href
- domain 包含页面的域名
- referrer 当前页面的来源
- 特殊集合： document 对象上还暴露了几个特殊集合，这些集合也都是 HTMLCollection 的实例。这些集合是访问文档中公共部分的快捷方式，列举如下。
 document.anchors 包含文档中所有带 name 属性的<a>元素。
 document.applets 包含文档中所有<applet>元素（因为<applet>元素已经不建议使用，所
以这个集合已经废弃）。
 document.forms 包含文档中所有<form>元素（与 document.getElementsByTagName ("form")
返回的结果相同）。
 document.images 包含文档中所有<img>元素（与 document.getElementsByTagName ("img")
返回的结果相同）。
 document.links 包含文档中所有带 href 属性的<a>元素。
这些特殊集合始终存在于 HTMLDocument 对象上，而且与所有 HTMLCollection 对象一样，其内
容也会实时更新以符合当前文档的内容。

### Element

- 元素的通用属性： id, title, className，dir, lang

```html

```
