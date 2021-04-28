# BOM

BOM 的核心是 window 对象，表示浏览器的实例。window 对象在浏览器中有两重身份，一个是ECMAScript 中的 `Global 对象`，另一个就是浏览器窗口的 `JavaScript 接口`

## window

在全局变量中使用`var`定义变量或者function定义方法，这些变量都会成为window的属性

### 窗口关系

要理解窗口关系，就要理解`顶层窗口`，`父窗口`，`当前窗口`,分别对应window.top, window.parent, window.self
`top 对象`始终指向最上层（最外层）窗口，即浏览器窗口本身。而 `parent 对象`则始终指向当前窗口的父窗口。如果当前窗口是最上层窗口，则 parent 等于 top（都等于 window）。最上层的 window如果不是通过 window.open()打开的，那么其 name 属性就不会包含值。
还有一个 `self 对象`，它是终极 window 属性，始终会指向 window。实际上，self 和 window 就是同一个对象。之所以还要暴露 self，就是为了和 top、parent 保持一致。

### 窗口位置

window 对象的位置可以通过不同的属性和方法来确定。现代浏览器提供了 `screenLeft` 和`screenTop` 属性，用于表示窗口相对于屏幕左侧和顶部的位置 ，返回值的单位是 CSS 像素。可以使用 `moveTo()`和 `moveBy()`方法移动窗口。这两个方法都接收两个参数，其中 moveTo()接收要移动到的新位置的**绝对坐标** x 和 y；而 moveBy()则接收**相对**当前位置在两个方向上移动的像素数

### 像素比

像素分为css像素（也叫逻辑像素）和物理像素（也叫设备像素）。物理像素的单位是pt,是绝对单位；逻辑像素的单位是px，是相对单位。像素比 = 物理像素 / css像素，表示一个css像素需要多少个物理像素去渲染，简写为dpr。分辨率可以理解为一个屏幕有多少个物理像素点，在出厂时就已经固定下来。

让我们记住以上的概念，现在我们来举个例子好好理解：比如一个屏幕的分辨率是1920*1080，它表示这块屏幕就是1920*1080个物理像素点。默认情况下dpr = 1,即一个css像素占用一个物理像素，所以我们在打印screen.width和screen.height分别为1920和1080,它们的单位均为px。然后我们在设置里调整缩放与布局为150%，这个其实就是dpr，即现在dpr = 1.5, 相当于一个css像素的宽高变为一个物理像素的1.5倍，那么1920*1080个像素点的屏幕有多少个css像素呢，不难计算是 (1920 / 1.5)* (1080 / 1.5), 即1280 * 720 px,打印screen.width和screen.height分别为1280和720。

举个例子，手机屏幕的`物理分辨率`可能是 1920×1080，但因为其像素可能非常小，所以浏览器就需要将其分辨率降为较低的`逻辑分辨率`，比如 640×320。这个物理像素与 CSS 像素之间的转换比率由window.devicePixelRatio 属性提供。对于分辨率从 1920×1080 转换为 640×320 的设备，window. devicePixelRatio 的值就是 3。这样一来，12 像素（CSS 像素）的文字实际上就会用 36 像素的物理像素来显示。

window.devicePixelRatio 实际上与每英寸像素数（DPI，dots per inch）是对应的。DPI 表示单
位像素密度，而 window.devicePixelRatio 表示物理像素与逻辑像素之间的缩放系数。

### 窗口大小

- document.documentElement.clientHeight 和 document.documentElement.clientWidth 返回页面视口的宽度和高度，不包含滚动条
- window.innerHeight 和 window.innerWidth 返回浏览器页面视口的宽高，不包含工具栏
- window.outerHeight 和 window.outerWidth 返回浏览器页面视口的宽高
- document.body.clientWidth 和 document.body.clientHeight 兼容ie
- 用resizeTo()和resizeBy()方法调整窗口大小

```javascript
  let pageWidth = window.innerWidth, 
  pageHeight = window.innerHeight; 
  if (typeof pageWidth != "number") { 
    if (document.compatMode == "CSS1Compat"){ // 标准模式
      pageWidth = document.documentElement.clientWidth; 
      pageHeight = document.documentElement.clientHeight; 
    } else { 
      pageWidth = document.body.clientWidth; 
      pageHeight = document.body.clientHeight; 
    } 
  }
```

### 视口的位置

scroll()、scrollTo()和 scrollBy()方法滚动页面

## 导航与打开新窗口

window.open()方法可以用于`导航到指定 URL`，也可以用于`打开新浏览器窗口`。这个方法接收 4个参数：要加载的 URL、目标窗口、特性字符串和表示新窗口在浏览器历史记录中是否替代当前加载页面的布尔值。通常，调用这个方法时只传前 3 个参数，最后一个参数只有在不打开新窗口时才会使用。如果 window.open()的第二个参数是一个已经存在的窗口或窗格（frame）的名字，则会在对应的窗口或窗格中打开 URL。下面是一个例子：

```javascript
// 与<a href="http://www.wrox.com" target="topFrame"/>相同
window.open("http://www.wrox.com/", "topFrame");
```

执行这行代码的结果就如同用户点击了一个 href 属性为"http://www.wrox.com"，target 属性为"topFrame"的链接。如果有一个窗口名叫"topFrame"，则这个窗口就会打开这个 URL；否则就会打开一个新窗口并将其命名为"topFrame"。第二个参数也可以是一个特殊的窗口名，比如_self、_parent、_top 或_blank。

- 如果 window.open()的第二个参数不是已有窗口，则会打开一个新窗口或标签页
- 第三个参数，即特性字符串，用于指定新窗口的配置。如果没有传第三个参数，则新窗口（或标签页）会带有所有默认的浏览器特性（工具栏、地址栏、状态栏等都是默认配置）。如果打开的不是新窗口，则忽略第三个参数。
- window.open()方法返回一个对新建窗口的引用。这个对象与普通 window 对象没有区别，只是为控制新窗口提供了方便
- 新创建窗口的 window 对象有一个属性 opener，指向打开它的窗口。

### 系统对话框

alert, confirm, prompt, print, find

## location

location 是最有用的 BOM 对象之一，提供了当前窗口中加载文档的信息，以及通常的导航功能。这个对象独特的地方在于，它既是 window 的属性，也是 document 的属性。也就是说，window.location 和 document.location 指向同一个对象。location 对象不仅保存着当前加载文档的信息，也保存着把 URL 解析为离散片段后能够通过属性访问的信息。这些解析后的属性在下表中有详细说明（location 前缀是必需的）

- URLSearchParams 查询字符串
- 操作地址: 以下3条语句能达到同样的效果,导航到<http://www.wrox.com>

```javascript
  location.assign("http://www.wrox.com"); 
  window.location = "http://www.wrox.com"; 
  location.href = "http://www.wrox.com"; 
```

- 除了 hash 之外，只要修改 location 的一个属性，就会导致页面重新加载新 URL。

- replace 传入一个url, 会替换当前的浏览器url,并且不会在浏览器中增加历史记录,因此也不能点击后台回到最开始的url
- reload 会重新加载当前页面,默认从浏览器缓存中加载,如果想强制从服务器重新加载，可以像下面这样给 reload()传个 true：
`location.reload(true)`

## navigator

navigator 对象实现了 NavigatorID 、 NavigatorLanguage 、 NavigatorOnLine 、NavigatorContentUtils 、 NavigatorStorage 、 NavigatorStorageUtils 、 NavigatorConcurrentHardware、NavigatorPlugins 和 NavigatorUserMedia 接口定义的属性和方法

- 检测插件 navigator.plugins 和 ActiveXObject
- 注册处理程序
现代浏览器支持 navigator 上的（在 HTML5 中定义的）registerProtocolHandler()方法。这个方法可以把一个网站注册为处理某种特定类型信息应用程序。随着在线 RSS 阅读器和电子邮件客户端的流行，可以借助这个方法将 Web 应用程序注册为像桌面软件一样的默认应用程序

## screen

## history

history 对象表示当前窗口首次使用以来用户的导航历史记录。因为 history 是 window 的属性，所以每个 window 都有自己的 history 对象。出于安全考虑，这个对象不会暴露用户访问过的 URL，但可以通过它在不知道实际 URL 的情况下前进和后退

### 导航

- go 前进或后退n页,传入整数或字符串
- back 后退一页
- forward 前进一页

### 历史状态管理
