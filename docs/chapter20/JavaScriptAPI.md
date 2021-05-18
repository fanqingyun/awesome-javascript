# JavaScriptAPI

## Atomics与SharedArrayBuffer

Atomics API通过强制同一时刻只能对缓冲区执行一个操作，可以让多个上下文(像主线程和web worker线程就不在同一个上下文，它们不能直接通信，必须通过消息完成)安全地读写一个SharedArrayBuffer。
SharedArrayBuffer与ArrayBuffer具有同样的API。二者的主要区别是ArrayBuffer必须在不同执行上下文间切换，SharedArrayBuffer则可以被任意多个执行上下文同时使用。

```javascript
  const workerScript = ` 
  self.onmessage = ({data}) => { 
  const view = new Uint32Array(data); 
  // 执行 1 000 000 次加操作
  for (let i = 0; i < 1E6; ++i) { 
  // 线程不安全加操作会导致资源争用
  view[0] += 1; 
  // // 线程安全的加操作：把view[0] += 1;注释掉，打开Atomics.add(view, 0, 1);则能得到期望结果4000001
  // Atomics.add(view, 0, 1);
  } 
  self.postMessage(null); 
  }; 
  `; 
  const workerScriptBlobUrl = URL.createObjectURL(new Blob([workerScript])); 
  // 创建容量为 4 的工作线程池
  const workers = []; 
  for (let i = 0; i < 4; ++i) { 
  workers.push(new Worker(workerScriptBlobUrl)); 
  } 
  // 在最后一个工作线程完成后打印出最终值
  let responseCount = 0; 
  for (const worker of workers) { 
  worker.onmessage = () => { 
  if (++responseCount == workers.length) { 
  console.log(`Final buffer value: ${view[0]}`); 
  } 
  }; 
  } 
  // 初始化 SharedArrayBuffer 
  const sharedArrayBuffer = new SharedArrayBuffer(4); 
  const view = new Uint32Array(sharedArrayBuffer); 
  view[0] = 1; 
  // 把 SharedArrayBuffer 发送到每个工作线程
  for (const worker of workers) { 
  worker.postMessage(sharedArrayBuffer); 
  } 
  //（期待结果为 4000001。实际输出可能类似这样：）
  // Final buffer value: 2145106
```

## 跨上下文消息

跨文档消息，有时候也简称为XDM（cross-document messaging），是一种在不同执行上下文（如不同工作线程或不同源的页面）间传递信息的能力。例如，www.wrox.com上的页面想要与包含在内嵌窗格中的p2p.wrox.com上面的页面通信。在XDM之前，要以安全方式实现这种通信需要很多工作。XDM以安全易用的方式规范化了这个功能。
**XDM的核心是postMessage()方法**

postMessage()方法接收3个参数：消息、表示目标接收源的字符串和可选的可传输对象的数组（只与工作线程相关）。第二个参数对于安全非常重要，其可以限制浏览器交付数据的目标。

`注意: 跨上下文消息用于窗口之间通信或工作线程之间通信。本节主要介绍使用postMessage()与其他窗口通信。关于工作线程之间通信、MessageChannel和BroadcastChannel，可以参考第27章。`

postMessage使用例子：
假设窗口index.html为父窗口，来自<http://172.25.160.1:9002>, 窗口src.html为子窗口，来自<http://172.25.160.1:9001>，父窗口要和子窗口通信，
1，获得子窗口的引用，如果子窗口是以iframe的方式嵌入在父窗口，则可通过contentWindow，比如document.getElementById("xxx").contentWindow; 或者通过window.open的返回值获得子窗口的引用，我们把这个引用命名为otherWin
2, otherWin 向自身发送消息，即postMessage的第二个参数跟otherWin的来源一致，为<http://172.25.160.1:9001>，所以就有otherWin.postMessage('发送给子窗口的消息', <http://172.25.160.1:9001>)
这2个步骤的伪代码可以写为：

```javascript
let otherWin = window.open('http://172.25.160.1:9001/src.html')
或
let otherWin = document.getElementById("xxx").contentWindow
otherWin.postMessage('发送给子窗口的消息', 'http://172.25.160.1:9001')
```

以上2个步骤的代码都是写在父窗口里的.
3, src.html里通过window.addEventListener('message', (event) => {}) 接收消息，event.origin表示消息是从哪里发过来的，本例子中是从父窗口过来的，所以值为<http://172.25.160.1:9002>; event.data表示接收的信息，这里为 '发送给子窗口的消息'; event.source表示父窗口的代理，指向父窗口，子窗口接收到消息后要反馈给父窗口可以event.source.postMessage('接收到了', event.origin)，这里是不是可以得出一个结论，要想postMessage发送成功，要么把第二个参数设置为*，要么是执行postMessage方法的对象的来源和postMessage方法的第二个参数一致。比如otherWin指向子窗口，而子窗口来自<http://172.25.160.1:9001>， 则postMessage的第二个参数为<http://172.25.160.1:9001>；event.source指向父窗口，而父窗口来自<http://172.25.160.1:9002>，则postMessage的第二个参数为<http://172.25.160.1:9002>，即event.origin.
4, 父窗口要接收到子窗口的反馈，同样通过window.addEventListener('message', (event) => {}) 接收消息
注意点：otherWin.postMessage('发送给子窗口的消息', 'http://172.25.160.1:9001')可以放在异步任务里，避免来不及发送消息报错
`Failed to execute 'postMessage' on 'DOMWindow': The target origin provided ('http://172.25.160.1:9001') does not match the recipient window's origin ('http://172.25.160.1:9002').`

## Encoding API

这部分主要涉及文本编码和文本解码，分别有批量和流2种方式，因此对应有4个API, TextEncoder、TextEncoderStream、TextDecoder和TextDecoderStream。主要用于实现字符串与定型数组之间的转换。

### 文本编码

#### 批量编码TextEncoder

```javascript
let te = new TextEncoder()
let rs = te.encoder('abc')
// Uint8Array(3) [97, 98, 99],返回一个定型数组
let mrs = te.encoder('☺')
// Uint8Array(3) [226, 152, 186]

// 将文本写入定型数组,返回一个字典，字典包含read和written属性，分别表示成功从源字符串读取了多少字符和向目标数组写入了多少字符。如果定型数组的空间不够，编码就会提前终止，返回的字典会体现这个结果
let intArr = new Uint8Array(3)
te.encodeInto('☺', intArr)
// {read: 1, written: 3}
intArr = new Uint8Array(2)
te.encodeInto('123', intArr)
// {read: 2, written: 2}
intArr = new Uint8Array(2)
te.encodeInto('☺', intArr)
// {read: 0, written: 0}

te.encoding // 'utf-8', 只读属性
```

#### 流编码TextEncoderStream

TextEncoderStream其实就是TransformStream形式的TextEncoder。将解码后的文本流通过管道输入流编码器会得到编码后文本块的流

```JavaScript
async function* chars() { 
 const decodedText = 'foo'; 
 for (let char of decodedText) { 
 yield await new Promise((resolve) => setTimeout(resolve, 1000, char)); 
 } 
} 
const decodedTextStream = new ReadableStream({ 
 async start(controller) { 
 for await (let chunk of chars()) { 
 controller.enqueue(chunk); 
 } 
 controller.close(); 
 } 
}); 
const encodedTextStream = decodedTextStream.pipeThrough(new TextEncoderStream()); 
const readableStreamDefaultReader = encodedTextStream.getReader(); 
(async function() { 
 while(true) { 
 const { done, value } = await readableStreamDefaultReader.read(); 
 if (done) { 
 break; 
 } else { 
 console.log(value); 
 } 
 } 
})(); 

// async 会把yield或return 的结果包装成一个promise
```

### 文本解码

#### 批量解码

```javascript
const textDecoder = new TextDecoder(); 
// f 的 UTF-8 编码是 0x66（即十进制 102）
// o 的 UTF-8 编码是 0x6F（即二进制 111）
const encodedText = Uint32Array.of(102, 111, 111); 
const decodedText = textDecoder.decode(encodedText); 
console.log(decodedText); // "f o o " 
// 解码器是用于处理定型数组中分散在多个索引上的字符的，包括表情符号：
const textDecoder = new TextDecoder(); 
// ☺的 UTF-8 编码是 0xF0 0x9F 0x98 0x8A（即十进制 240、159、152、138）
const encodedText = Uint8Array.of(240, 159, 152, 138);
const decodedText = textDecoder.decode(encodedText); 
console.log(decodedText); // ☺
// 与 TextEncoder 不同，TextDecoder 可以兼容很多字符编码。比如下面的例子就使用了 UTF-16
// 而非默认的 UTF-8：
const textDecoder = new TextDecoder('utf-16'); 
// f 的 UTF-8 编码是 0x0066（即十进制 102）
// o 的 UTF-8 编码是 0x006F（即二进制 111）
const encodedText = Uint16Array.of(102, 111, 111); 
const decodedText = textDecoder.decode(encodedText); 
console.log(decodedText); // foo 
```

#### 流解码

TextDecoderStream其实就是TransformStream形式的TextDecoder。将编码后的文本流通过管道输入流解码器会得到解码后文本块的流

```javascript
async function* chars() { 
 // 每个块必须是一个定型数组
 const encodedText = [102, 111, 111].map((x) => Uint8Array.of(x)); 
 for (let char of encodedText) { 
 yield await new Promise((resolve) => setTimeout(resolve, 1000, char)); 
 } 
} 
const encodedTextStream = new ReadableStream({ 
 async start(controller) { 
 for await (let chunk of chars()) { 
 controller.enqueue(chunk); 
 } 
 controller.close(); 
 } 
}); 
const decodedTextStream = encodedTextStream.pipeThrough(new TextDecoderStream());  
const readableStreamDefaultReader = decodedTextStream.getReader(); 
(async function() { 
 while(true) { 
 const { done, value } = await readableStreamDefaultReader.read(); 
 if (done) { 
 break; 
 } else { 
 console.log(value); 
 } 
 } 
})(); 
// f 
// o 
// o 
// 文本解码器流经常与 fetch()一起使用，因为响应体可以作为 ReadableStream 来处理。比如：
const response = await fetch(url); 
const stream = response.body.pipeThrough(new TextDecoderStream());
const decodedStream = stream.getReader() 
for await (let decodedChunk of decodedStream) { 
 console.log(decodedChunk);
}
```

## File API与Blob API

### FileReader类型

例子参考file_blob.html, FileReader实例可以读取blob对象或file对象

### FileReaderSync类型

是FileReader的同步版本，在工作线程中使用

### Blob

某些情况下，可能需要读取部分文件而不是整个文件。为此，File对象提供了一个名为slice()的方法。slice()方法接收两个参数：起始字节和要读取的字节数。这个方法返回一个Blob的实例，而Blob实际上是File的超类。blob表示二进制大对象（binary larget object），是JavaScript对不可修改二进制数据的封装类型。包含字符串的数组、ArrayBuffers、ArrayBufferViews，甚至其他Blob都可以用来创建blob。Blob构造函数可以接收一个options参数，并在其中指定MIME类型。

对象URL有时候也称作Blob URL，是指引用存储在File或Blob中数据的URL。对象URL的优点是不用把文件内容读取到JavaScript也可以使用文件。只要在适当位置提供对象URL即可。要创建对象URL，可以使用window.URL.createObjectURL()方法并传入File或Blob对象。这个函数返回的值是一个指向内存中地址的字符串。因为这个字符串是URL，所以可以在DOM中直接使用,比如放在img的src里或a的href里当作下载链接。

组合使用HTML5拖放API与File API可以创建读取文件信息的有趣功能。在页面上创建放置目标后，可以从桌面上把文件拖动并放到放置目标。这样会像拖放图片或链接一样触发drop事件。被放置的文件可以通过事件的event.dataTransfer.files属性读到，这个属性保存着一组File对象，就像文本输入字段一样。

## 媒体元素

### video

### audio

## 拖放

拖放事件几乎可以让开发者控制拖放操作的方方面面。关键的部分是确定每个事件是在哪里触发的。有的事件在被拖放元素上触发，有的事件则在放置目标上触发。在某个元素被拖动时，会（按顺序）触发以下事件：
（1）dragstart
（2）drag
（3）dragend
在按住鼠标键不放并开始移动鼠标的那一刻，被拖动元素上会触发dragstart事件。此时光标会变成非放置符号（圆环中间一条斜杠），表示元素不能放到自身上。拖动开始时，可以在ondragstart事件处理程序中通过JavaScript执行某些操作。dragstart事件触发后，只要目标还被拖动就会持续触发drag事件。这个事件类似于mousemove，即随着鼠标移动而不断触发。当拖动停止时（把元素放到有效或无效的放置目标上），会触发dragend事件。**所有这3个事件的目标都是被拖动的元素**。默认情况下，浏览器在拖动开始后不会改变被拖动元素的外观，因此是否改变外观由你来决定。不过，大多数浏览器此时会创建元素的一个半透明副本，始终跟随在光标下方。在把元素拖动到一个有效的放置目标上时，会依次触发以下事件：
（1）dragenter
（2）dragover
（3）dragleave或drop
只要一把元素拖动到放置目标上，dragenter事件（类似于mouseover事件）就会触发。dragenter事件触发之后，会立即触发dragover事件，并且元素在放置目标范围内被拖动期间此事件会持续触发。当元素被拖动到放置目标之外，dragover事件停止触发，dragleave事件触发（类似于mouseout事件）。如果被拖动元素被放到了目标上，则会触发drop事件而不是dragleave事件。这些事件的目标是放置目标元素。

### dataTransfer对象

用于从被拖动元素向放置目标传递字符串数据。因为这个对象是event的属性，所以在拖放事件的事件处理程序外部无法访问dataTransfer。在事件处理程序内部，可以使用这个对象的属性和方法实现拖放功能。dataTransfer对象现在已经纳入了HTML5工作草案。dataTransfer对象有两个主要方法：getData()和setData()。顾名思义，getData()用于获取setData()存储的值。setData()的第一个参数以及getData()的唯一参数是一个字符串，表示要设置的数据类型："text"或"URL"，如下所示

## Notifications API

### 请求权限

```javascript
Notification.requestPermission() 
 .then((permission) => { 
 console.log('User responded to permission request:', permission); 
 }); 
 ```

"granted"值意味着用户明确授权了显示通知的权限。除此之外的其他值意味着显示通知会静默失
败。如果用户拒绝授权，这个值就是"denied"。一旦拒绝，就无法通过编程方式挽回，因为不可能再
触发授权提示

### 消息体格式

```javascript
// 可以通过 options 参数对通知进行自定义，包括设置通知的主体、图片和振动等：
new Notification('Title text!', {
 body: 'Body text!', 
 image: 'path/to/image.png', 
 vibrate: true 
}); 
```

### 生命周期

通知并非只用于显示文本字符串，也可用于实现交互。Notifications API提供了4个用于添加回调的生命周期方法：
❑ onshow在通知显示时触发；
❑ onclick在通知被点击时触发；
❑ onclose在通知消失或通过close()关闭时触发；
❑ onerror在发生错误阻止通知显示时触发。

## Page Visibility API

Web开发中一个常见的问题是开发者不知道用户什么时候真正在使用页面。如果页面被最小化或隐藏在其他标签页后面，那么轮询服务器或更新动画等功能可能就没有必要了。Page Visibility API旨在为开发者提供页面对用户是否可见的信息。这个API本身非常简单，由3部分构成。
❑ document.visibilityState值，表示下面4种状态之一。
  ■ 页面在后台标签页或浏览器中最小化了。
  ■ 页面在前台标签页中。
  ■ 实际页面隐藏了，但对页面的预览是可见的（例如在Windows 7上，用户鼠标移到任务栏图标上会显示网页预览）。
  ■ 页面在屏外预渲染。
❑ visibilitychange事件，该事件会在文档从隐藏变可见（或反之）时触发。
❑ document.hidden布尔值，表示页面是否隐藏。这可能意味着页面在后台标签页或浏览器中被最小化了。这个值是为了向后兼容才继续被浏览器支持的，应该优先使用document.visibilityState检测页面可见性。
要想在页面从可见变为隐藏或从隐藏变为可见时得到通知，需要监听visibilitychange事件。document.visibilityState的值是以下三个字符串之一：
❑ "hidden"
❑ "visible"
❑ "prerender"

## Streams API

### 可写流

### 可读流

### 转换流

## 计时API

## Web组件

这里所说的Web组件指的是一套用于增强DOM行为的工具，包括影子DOM、自定义元素和HTML模板

## Web Cryptography API
