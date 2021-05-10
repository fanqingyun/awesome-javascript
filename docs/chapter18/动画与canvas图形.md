# 动画和canvas

## requestAnimationFrame

requestAnimationFrame 将回调函数加入队列，可以看成一个异步任务

```javascript
function test() {
console.log('requestAnimationFrame')
}
setTimeout(console.log, 0, 'settimeout0')
setTimeout(console.log, 1000, 'settimeout1s')
requestAnimationFrame(test)
Promise.resolve('promise').then(console.log)
console.log('normal')

// normal
// promise
// settimeout0
// requestAnimationFrame
// settimeout1s
```

## canvas

### 创建一个2d上下文

```javascript
let canvas = document.createElement('canvas')
// 传入2d
let ctx = canvas.getContext('2d')
// 我们所有的操作都是在ctx上
```

```markdown
canvas: canvas
direction: "ltr"
fillStyle: "#000000"
filter: "none"
font: "10px sans-serif"
globalAlpha: 1
globalCompositeOperation: "source-over"
imageSmoothingEnabled: true
imageSmoothingQuality: "low"
lineCap: "butt"
lineDashOffset: 0
lineJoin: "miter"
lineWidth: 1
miterLimit: 10
shadowBlur: 0
shadowColor: "rgba(0, 0, 0, 0)"
shadowOffsetX: 0
shadowOffsetY: 0
strokeStyle: "#000000"
textAlign: "start"
textBaseline: "alphabetic"
__proto__: CanvasRenderingContext2D
arc: ƒ arc()
arcTo: ƒ arcTo()
beginPath: ƒ beginPath()
bezierCurveTo: ƒ bezierCurveTo()
canvas: canvas
clearRect: ƒ clearRect()
clip: ƒ clip()
closePath: ƒ closePath()
createImageData: ƒ createImageData()
createLinearGradient: ƒ createLinearGradient()
createPattern: ƒ createPattern()
createRadialGradient: ƒ createRadialGradient()
direction: (...)
drawFocusIfNeeded: ƒ drawFocusIfNeeded()
drawImage: ƒ drawImage()
ellipse: ƒ ellipse()
fill: ƒ fill()
fillRect: ƒ fillRect()
fillStyle: (...)
fillText: ƒ fillText()
filter: (...)
font: (...)
getContextAttributes: ƒ getContextAttributes()
getImageData: ƒ getImageData()
getLineDash: ƒ getLineDash()
getTransform: ƒ getTransform()
globalAlpha: (...)
globalCompositeOperation: (...)
imageSmoothingEnabled: (...)
imageSmoothingQuality: (...)
isPointInPath: ƒ isPointInPath()
isPointInStroke: ƒ isPointInStroke()
lineCap: (...)
lineDashOffset: (...)
lineJoin: (...)
lineTo: ƒ lineTo()
lineWidth: (...)
measureText: ƒ measureText()
miterLimit: (...)
moveTo: ƒ moveTo()
putImageData: ƒ putImageData()
quadraticCurveTo: ƒ quadraticCurveTo()
rect: ƒ rect()
resetTransform: ƒ resetTransform()
restore: ƒ restore()
rotate: ƒ rotate()
save: ƒ save()
scale: ƒ scale()
setLineDash: ƒ setLineDash()
setTransform: ƒ setTransform()
shadowBlur: (...)
shadowColor: (...)
shadowOffsetX: (...)
shadowOffsetY: (...)
stroke: ƒ stroke()
strokeRect: ƒ strokeRect()
strokeStyle: (...)
strokeText: ƒ strokeText()
textAlign: (...)
textBaseline: (...)
transform: ƒ transform()
translate: ƒ translate()
constructor: ƒ CanvasRenderingContext2D()
Symbol(Symbol.toStringTag): "CanvasRenderingContext2D"
```

### toDataURL

将一张画布转为一个dataURL路径，可以传给img标签的src属性展示
