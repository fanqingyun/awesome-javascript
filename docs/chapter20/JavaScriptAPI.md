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

Encoding API主要用于实现字符串与定型数组之间的转换。规范新增了4个用于执行转换的全局类：TextEncoder、TextEncoderStream、TextDecoder和TextDecoderStream。

## File API与Blob API

## 拖放

## Notifications API

## Page Visibility API

## Streams API

## 计时API

## Web组件

## Web Cryptography API

```javascript
/**

*/
```
