# Promise和异步函数

## SetTImeout和setInterval

## Promise

```javascript
  let p = new Promise((resolve, reject) => setTimeout(resolve, 1000)); 
  // 在 console.log 打印Promise实例的时候，还不会执行超时回调（即 resolve()）
  setTimeout(console.log, 0, p);
  // Promise {<pending>}

  let p = new Promise((resolve, reject) => setTimeout(resolve, 1000)); 
  // 在 console.log 打印Promise实例的时候，执行回调（即 resolve()）
  setTimeout(console.log, 1000, p);
  // Promise {<fulfilled>: undefined}
```

### Promise.resolve() 和Promise.reject()

Promise并非一开始就是pendding状态，也可以使用Promise.resolve()实例化一个解决状态的Promise,下面两个Promise实例实际上是一样的：

```javascript
  let p1 = new Promise((resolve, reject) => resolve()); 
  let p2 = Promise.resolve(); 
```

### thenable接口

就是看有没有实现名称为`then`的方法

### Promise.prototype.then

Promise.prototype.then()是为Promise实例添加处理程序的主要方法。这个 then()方法接收最多两个参数：onResolved 处理程序和 onRejected 处理程序。这两个参数都是可选的，如果提供的话，则会在Promise分别进入“兑现”和“拒绝”状态时执行。`Promise.prototype.then()方法返回一个新的Promise实例`,这个新Promise实例基于 onResovled 处理程序的返回值构建。换句话说，该处理程序的返回值会通过Promise.resolve()包装来生成新Promise。如果没有提供这个处理程序，则 Promise.resolve()就会包装上一个Promise解决之后的值。如果没有显式的返回语句，则 Promise.resolve()会包装默认的返回值 undefined。

- 若调用 then()时不传处理程序，则原样向后传

```javascript
  let p1 = new Promise(resolve => {
    resolve(1)
  })
  let p2 = p1.then()
// Promise {<fulfilled>: 1}

  let p3 = p1.then(22) // 22不是方法，原样后传
 // Promise {<fulfilled>: 1}

```

- 如果有显式的返回值，则 Promise.resolve()会包装这个

```javascript
  let p1 = new Promise(resolve => {
    resolve()
  })
  let p2 = p1.then(() => new Error(1))
  // Promise {<fulfilled>: Error: 1 at <anonymous>:1:20}

  let p3 = new Promise((resolve, reject) => {
    reject()
  })
  let p4 = p3.then(undefined, () => 'bar')
  // Promise {<fulfilled>: "bar"}

// 无论是返回new Error还是最开始reject, then方法有返回值时都是用Promise.resolve()包装的
```

- 抛出异常会返回拒绝的期约

```javascript
  let p1 = new Promise(resolve => {
    resolve()
  })
  let p2 = p1.then(() => { throw 'error' })
  // Promise {<rejected>: "error"}

// 注意这里与上面例子返回错误值的区别，返回错误值不会触发拒绝行为，而会把错误对象包装在一个解决的期约中
```

- Promise.resolve()保留返回的期约,即如果then的返回值是一个期约

```javascript
let p1 = new Promise((resolve, reject) => {
    resolve()
  })
p8 = p1.then(() => new Promise(() => {})); // Promise {<pending>}
p9 = p1.then(() => Promise.reject()); // Promise {<rejected>: undefined}
```

### Promise.prototype.catch

其实就是Promise.prototype.then(null, onRejected)的语法糖,

```javascript
p.then(null, onRejected); // rejected 
p.catch(onRejected); // rejected 
```

`Promise.prototype.catch()返回一个新的期约实例`

### Promise.prototype.finally

这个处理程序在期约转换为解决或拒绝状态时都会执行。这个方法可以避免 onResolved 和 onRejected 处理程序中出现冗余代码。但 onFinally 处理程序没有办法知道期约的状态是解决还是拒绝，所以这个方法主要用于添加清理代码,返回一个新的期约，一般情况下期约的状态是父期约的传递，除了以下情况

1. finally方法里抛出了错误: 返回一个状态为rejected的期约
2. finally方法的返回值是一个期约: Promise.resolve()保留返回的期约

### 非重入期约方法

跟在添加这个处理程序的代码之后的同步代码一定会在处理程序之前先执行。即使期约一开始就是与附加处理程序关联的状态，执行顺序也是这样的。这个特性由 JavaScript 运行时保证，被称为“非重入”（non-reentrancy）特性。下面的例子演示了这个特性：

```javascript
  // 创建解决的期约
  let p = Promise.resolve(); 
  // 添加解决处理程序
  // 直觉上，这个处理程序会等期约一解决就执行
  p.then(() => console.log('onResolved handler')); 
  // 同步输出，证明 then()已经返回
  console.log('then() returns'); 
  // 实际的输出：
  // then() returns 
  // onResolved handler 


  let synchronousResolve; 
  // 创建一个期约并将解决函数保存在一个局部变量中
  let p = new Promise((resolve) => { 
  synchronousResolve = function() { 
  console.log('1: invoking resolve()'); 
  resolve(); 
  console.log('2: resolve() returns'); 
  }; 
  }); 
  p.then(() => console.log('4: then() handler executes')); 
  synchronousResolve(); 
  console.log('3: synchronousResolve() returns'); 
  // 实际的输出：
  // 1: invoking resolve() 
  // 2: resolve() returns 
  // 3: synchronousResolve() returns 
  // 4: then() handler executes 
```

非重入适用于 onResolved/onRejected 处理程序、catch()处理程序和 finally()处理程序

### 期约连锁和期约合成

把期约逐个地串联起来是一种非常有用的编程模式。之所以可以这样做，是因为每个期约实例的方法（then()、catch()和 finally()）都会返回一个新的期约对象，而这个新期约又有自己的实例方法。这样连缀方法调用就可以构成所谓的“期约连锁”。

``` javascript
  let p1 = new Promise((resolve, reject) => { 
  console.log('p1 executor'); 
  setTimeout(resolve, 1000); 
  }); 
  p1.then(() => new Promise((resolve, reject) => { 
  console.log('p2 executor'); 
  setTimeout(resolve, 1000); 
  })) 
  .then(() => new Promise((resolve, reject) => { 
  console.log('p3 executor'); 
  setTimeout(resolve, 1000); 
  })) 
  .then(() => new Promise((resolve, reject) => { 
  console.log('p4 executor'); 
  setTimeout(resolve, 1000); 
  })); 
  // p1 executor（1 秒后）
  // p2 executor（2 秒后）
  // p3 executor（3 秒后）
  // p4 executor（4 秒后）
```

#### Promise.all 和 Promise.race

接收一个可迭代参数，即一组期约。

1. Promise.all

- 如果至少有一个包含的期约待定，则合成的期约也会待定。如果有一个包含的期约拒绝，则合成的期约也会拒绝：

```javascript
  let p2 = Promise.all([ 
  Promise.resolve(), 
  Promise.reject(), 
  new Promise(() => {}),
  Promise.resolve() 
  ]);
  // 包含拒绝的期约,尽管也包含待定的期约，合成成的期约仍会拒绝
```

- 如果新期约的状态为rejected，则拒绝理由是传入参数中第一个状态为rejected的理由
- 只有传入的全部期约的成功解决，新的期约状态才为resolved

2. Promise.race
是一组集合中最先解决或拒绝的期约的镜像,新返回的期约状态与第一个落定的期约相同

### 期约取消 和期约进度通知

目前需要自行实现

## async和await

使用 async 关键字可以让函数具有异步特征，但总体上其代码仍然是同步求值的

- async 函数的执行结果是一个期约，其返回值的规则基本跟Promise一致，（Promise.resolve()会进行包装，抛出异常，返回一个期约）
- await 只能在async函数里执行
- async 函数返回的值如果实现了thenable方法，则返回的期约.then会执行该方法
- await 关键字会暂停执行异步函数后面的代码，让出 JavaScript 运行时的执行线程。这个行为与生成器函数中的 yield 关键字是一样的。await 关键字同样是尝试“解包”对象的值，然后将这个值传给表达式，再异步恢复异步函数的执行
- await 关键字期待（但实际上并不要求）一个实现 thenable 接口的对象，但常规的值也可以。如果是实现 thenable 接口的对象，则这个对象可以由 await 来“解包”。如果不是，则这个值就被当作已经解决的期约。

```javascript
  async function test(){
  const obj = {
  then(callback){
  callback(12)
  }
  }
  var t = await obj
  console.log(t)
  return t
  }
  console.log(test())
  // 12
  // Promise {<fulfilled>: 12}
```

await不仅仅是等待一个值，。JavaScript 运行时在碰到 await 关键字时，会记录在哪里暂停执行。等到 await 右边的值可用了，JavaScript 运行时会向消息
队列中推送一个任务，这个任务会恢复异步函数的执行。因此，即使 await 后面跟着一个立即可用的值，函数的其余部分也会被异步求值

```javascript
  async function foo() { 
  console.log(await Promise.resolve('foo')); 
  } 
  async function bar() { 
  console.log(await 'bar'); 
  } 
  async function baz() { 
  console.log('baz'); 
  } 
  foo(); 
  bar(); 
  baz(); 
  // baz 
  // bar 
  // foo
```

另外一个例子

```javascript
async function foo() { 
 console.log(2); 
 console.log(await Promise.resolve(8)); 
 console.log(9); 
} 
async function bar() { 
    console.log(4); 
    console.log(await 6); 
    console.log(7); 
  } 
console.log(1); 
foo(); 
console.log(3); 
bar(); 
console.log(5); 
// 1 
// 2 
// 3 
// 4 
// 5 
// 6 
// 7 
// 8 
// 9 
```

(1) 打印 1；
(2) 调用异步函数 foo()；
(3)（在 foo()中）打印 2；
(4)（在 foo()中）await 关键字暂停执行，向消息队列中添加一个期约在落定之后执行的任务；
(5) 期约立即落定，把给 await 提供值的任务添加到消息队列；
(6) foo()退出；
(7) 打印 3；
(8) 调用异步函数 bar()；
(9)（在 bar()中）打印 4；
(10)（在 bar()中）await 关键字暂停执行，为立即可用的值 6 向消息队列中添加一个任务；
(11) bar()退出；
(12) 打印 5；
(13) 顶级线程执行完毕；
(14) JavaScript 运行时从消息队列中取出解决 await 期约的处理程序，并将解决的值 8 提供给它；
(15) JavaScript 运行时向消息队列中添加一个恢复执行 foo()函数的任务；
(16) JavaScript 运行时从消息队列中取出恢复执行 bar()的任务及值 6；
(17)（在 bar()中）恢复执行，await 取得值 6；
(18)（在 bar()中）打印 6；
(19)（在 bar()中）打印 7；
(20) bar()返回；
(21) 异步任务完成，JavaScript 从消息队列中取出恢复执行 foo()的任务及值 8；
(22)（在 foo()中）打印 8；
(23)（在 foo()中）打印 9；
(24) foo()返回。
