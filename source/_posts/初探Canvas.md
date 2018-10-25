---
title: 初探Canvas
date: 2018-7-27 15:17:28
tags:
---
`Canvas` 是一个浏览器自带的图形绘制元素，随着高版本浏览器的普及率，`Canvas` 已经广泛的应用于日常网站。
<!-- more -->

### 什么是 `Canvas`

`Canvas` 最早是由 `Apple` 引入的，用于 `OS X Dashboard` 和 `Safari`。`Canvas` 是 `HTML5` 新增的一个元素，可用于通过使用 `JavaScript` 中的脚本来绘制图形。

**举个栗子：**

```html
<canvas id="canvas"></canvas>
```

```js
var canvas = document.getElementById('canvas');
var ctx = canvas.getContext('2d');
```

#### `Canvas` 的属性

实际上，`<canvas>` 标签只有两个属性—— `width` 和 `height`。
当没有设置宽度和高度的时候，`canvas` 会初始化宽度为`300px`和高度为`150px`。
`<canvas>` 元素可以像任何一个普通的图像一样（有`margin，border，background`等等属性）被设计。然而，这些样式不会影响在canvas中的实际图像。

### 使用 `Canvas` 绘制图形

#### 绘制矩形

```js
// 绘制一个填充矩形。
fillRect(x, y, width, height);

// 绘制一个矩形的边框
strokeRect(x, y, width, height);

// 清除指定矩形区域
clearRect(x, y, width, height);
```

更简单的方式：

```js
rect(x, y, width, height);

// 需要注意的是：当该方法执行的时候，moveTo()方法自动设置坐标参数（0,0）
```

#### 绘制路径

一个路径，甚至一个子路径，都是闭合的。使用路径绘制图形需要一些额外的步骤。

* 创建路径起始点。
* 使用画图命令去画出路径。
* 把路径封闭。
* 一旦路径生成，就能通过描边或填充路径区域来渲染图形。

```js
// 新建一条路径
beginPath();

// 闭合路径。如果图形是已经闭合了的，即当前点为开始点，该函数什么也不做。
// 当调用 fill() 时，会自动 closePath(); 但 stroke() 并不会。
closePath();

// 绘制图形轮廓
stroke();

// 填充路径内容区域
fill();
```

#### 移动笔触

将绘制的点移动到另一个位置。

```js
moveTo(x, y)
```

#### 绘制直线

```js
lineTo(x, y)
```

#### 绘制圆弧

```js
arc(x, y, radius, startAngle, endAngle, anticlockwise);

// 解释：
// 画一个以 x, y 为圆心，以 radius 为半径的圆弧(圆)
// 从 startAngle弧度 开始 至 endAngle弧度 结束，
// anticlockwise: bool; false=顺时针, true=逆时针

// eg:
arc(75,75,35,0,Math.PI,false);
arc(60,65,5,0,Math.PI*2,true);
```

#### 二次贝塞尔曲线及三次贝塞尔曲线

```js
// 二次
quadraticCurveTo(cp1x, cp1y, x, y);
// cp1x,cp1y 表示控制点，x,y 表示结束点位置

// 三次
bezierCurveTo(cp1x, cp1y, cp2x, cp2y, x, y)
// cp1x,cp1y 表示第一个控制点，cp2x,cp2y 表示第二个控制点，x,y 表示结束点
```

如下图所示：
![bezier](https://mdn.mozillademos.org/files/223/Canvas_curves.png)

#### `path2D` 对象

为了简化代码和提高性能，`Path2D` 对象 已可以在较新版本的浏览器中使用，用来缓存或记录绘画命令，这样你将能快速地回顾路径。

```js
// 创造一个 Path2D 对象
new Path2D();  // -> 空的 Path 对象
new Path2D(path);  // -> 克隆 Path 对象
new Path2D(svg);  // -> 从 Svg 建立 Path 对象

// 添加一个 path
Path2D.addPath(path, [, transform]);
```

举个栗子：

```js
// 先创建一个 path 对象，并画一个矩形
// var p1 = new Path2D("M10 10 h 80 v 80 h -80 Z");
var p1 = new Path2D();
p1.rect(10, 10, 100, 100);

// 创建 p2 克隆 p1, 并画圆
var p2 = new Path2D(p1);
p2.moveTo(220, 60);
p2.arc(170, 60, 50, 0, 2 * Math.PI);

// 什么都没有
ctx.stroke();

// 只有矩形
ctx.stroke(p1);

// 成功画出一个矩形和圆形
ctx.stroke(p2);
```

#### 色彩

`color` 的取值方式与 `css` 中取值相同。
```js
// 填充颜色
fillStyle = color;

// 描边颜色
strokeStyle = color;
```

#### 全局图形透明度

```js
// 全局透明度默认为 1， 取值范围为 [0.0, 1];
globalAlpha = 1;
```

#### 线型

```js
// 设置线条宽度, 默认为 1，必须为正数
lineWidth = value;

// 设置线条末端样式，默认为butt，[butt, round, square]
lineCap = type;

// 设置线条与线条接合处样式，默认为miter， [round, bevel, miter]
lineJoin = type;

// 限制当两条线相交时交接处最大长度；所谓交接处长度（斜接长度）是指线条交接处内角顶点到外角顶点的长度。
// 我没看懂
miterLimit = value;

// 返回一个包含当前虚线样式，长度为非负偶数的数组
getLineDash();

// 设置当前虚线样式
setLineDash(segments: array);

// 设置虚线样式的起始偏移量
lineDashOffset = value;
```

#### 渐变

```js
// 线性渐变
createLinearGradient(x1, y1, x2, y2);
//渐变的起点 (x1,y1) 与终点 (x2,y2)

// 径向渐变
createRadialGradient(x1, y1, r1, x2, y2, r2);
// x1, y1, r1，坐标为(x1, y1)，半径为 r1 的圆
// x2, y2, r2，坐标为(x2, y2)，半径为 r2 的圆

// 设置完渐变后再上色
// position 取值 [0, 1], color 同 css
gradient.addColorStop(position, color);
```

举个栗子：

```js
var lg = ctx.createLinearGradient(0, 0, 100, 100);
lg.addColorStop(0, 'white')
lg.addColorStop(1, 'black')

// 径向渐变同理
```

#### 图案样式

通过 `createPattern` 实现图案。

```js
createPattern(image, type);

// image 是一个 Image 对象的引用 或 另一个 Canvas 对象
// type 取 repeat，repeat-x，repeat-y 和 no-repeat 中的一个
```

#### 阴影

```js
// 阴影在 x 轴的延伸距离
shadowOffsetX = value;
// 阴影在 y 轴的延伸距离
shadowOffsetY = value;
// 阴影模糊成都
shadowBlur = value;
// 阴影的颜色
shadowColor = color;
```

#### 填充规则

```js
// 默认值：nonzero
// 可选值：nonzero || evenodd

ctx.fill('nonzero')
```
下一章会继续介绍 `Canvas` 的其它属性。

