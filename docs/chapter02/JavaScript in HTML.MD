### script标签
一般来说，浏览器解析页面是从上往下一行一行解析的，当遇到script标签（没有设置defer和async）时，会先执行js代码，暂停html的解析,执行完js代码后才会继续继续页面
#### 属性
- type 表示代码块中脚本语言的内容类型（也称 MIME 类型）。按照惯 例，这个值始终都是"text/javascript"，尽管"text/javascript"和"text/ecmascript" 都已经废弃了。JavaScript 文件的 MIME 类型通常是"application/x-javascript"，不过给 type 属性这个值有可能导致脚本被忽略。在非 IE 的浏览器中有效的其他值还有 "application/javascript"和"application/ecmascript"。`如果这个值是 module，则代码会被当成ES6模块，而且只有这时候代码中才能出现 import 和 export 关键字`。
- async 不阻塞html解析，一旦下载完成则立即执行，在load之前执行
- defer 不阻塞html解析，延迟到页面完全解析后执行，在DOMContentLoaded和load之前执行
- charset 字符集
- src 引入的外部资源路径
- integrity 允许比对接收到的资源和指定的加密签名以验证子资源完整性（SRI，Subresource Integrity）。如果接收到的资源的签名与这个属性指定的签名不匹配，则页面会报错，脚本不会执行。这个属性可以用于确保内容分发网络（CDN，Content Delivery Network）不会提供恶意内容。
- crossorigin 配置相关请求的CORS（跨源资源共享）设置。默认不使用CORS。crossorigin= "anonymous"配置文件请求不必设置凭据标志.crossorigin="use-credentials"设置凭据标志，意味着出站请求会包含凭据。

按照惯例，外部 JavaScript 文件的扩展名是.js。**这不是必需的，因为浏览器不会检查所包含 JavaScript 文件的扩展名**。`这就为使用服务器端脚本语言动态生成JavaScript 代码，或者在浏览器中将 JavaScript扩展语言（如TypeScript，或React的 JSX）转译为JavaScript提供了可能性`。**不过要注意，服务器经常会根据文件扩展来确定响应的正确 MIME 类型**。如果不打算使用.js 扩展名，一定要确保服务器能返回正确的 MIME 类型。

### 文档模式

通过doctype可以切换文档模式，主要体现在css上的区别，给JavaScript也带来副作用。

怪诞模式：没有写doctype

标准模式:  严格的文档类型声明 + `<!doctype html>`

准标准模式：过渡性和框架集的文档类型声明

也可以说，标准模式是除怪诞模式的其他文档模式

### noscript 标签
适用于浏览器不支持或禁用JavaScript