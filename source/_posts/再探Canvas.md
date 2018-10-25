---
title: 再探Canvas
date: 2018-8-2 15:32:39
tags:
---
本章将继续归整 `Canvas` 余下的属性。
<!-- more -->


#### 绘制文本

```js
// 普通样式效果
fillText(text, x, y, [, maxWidth]);
// 在指定位置(x, y)处填充文本 text，可选绘制的最大宽度。

// 文本线条为空心
strokeText(text, x, y, [, maxWidth]);
// 在指定位置(x, y)处绘制文本 text，可选绘制的最大宽度。

// 文本样式：

// 文字样式，默认 '10px sans-serif'，值同 css font 语法。
font = value;

// 文本对齐，默认 'start'，可选start, end, left, right, center
textAlign = value

// 文本基线对齐方式，默认 alphabetic
// 可选 top, hanging, middle, alphabetic, ideographic, bottom
textBaseLine = value

// 文本方向，默认 'inherit'，可选 ltr, rtl, inherit
direction = value
```

关于基线标准可以参考下图：
![baseline](http://www.whatwg.org/specs/web-apps/current-work/images/baselines.png)

#### 预测量文本宽度

```js
measureText()

// eg:

var text = ctx.measureText("foo")
console.log(text.width); // -> 16px
```

#### 使用图片

这个是 `Canvas` 最有意思的一个特性。
可以用于动态的图像合成或者作为图形的背景，以及游戏界面(`Sprites`)等等。浏览器支持的任意格式的外部图片都可以使用，比如`PNG、GIF`或者`JPEG`。 你甚至可以将同一个页面中其他`canvas`元素生成的图片作为图片源。

**绘制图片**

```js
// image 图片源
// x,y 表示在 canvas 中的绘制坐标
drawImage(image, x, y);
```

**绘制图片-缩放**

```js
// image 图片源
// x,y 表示在 canvas 中的绘制坐标
// width,height 表示图片在 canvas 中的绘制宽度
drawImage(image, x, y, width, height);
```

**绘制图片-切片**

```js
// image 图片源
// sx, sy, sWidth, sHeight 图片切片的位置和大小
// dx, dy, dWidth, dHeight 切片目标显示的位置和大小
drawImage(image, sx, sy, sWidth, sHeight, dx, dy, dWidth, dHeight)
```

如下图：
![draw](https://mdn.mozillademos.org/files/225/Canvas_drawimage.jpg)

#### 变形

`状态的保存和恢复`
在了解变形之前，先了解下两个在你开始绘制复杂图形时必不可少的方法。

```js
save();

restore();
```

`save` 和 `restore` 方法是用来保存和恢复 `canvas` 状态的，都没有参数。

`Canvas` 的状态就是当前画面应用的所有样式和变形的一个快照。
`Canvas` 状态存储在栈中，每当 `save()` 方法被调用后，当前的状态就被推送到栈中保存。
一个绘画状态包括：

* 当前应用的变形(即移动，旋转和缩放)
* `strokeStyle, fillStyle, globalAlpha, lineWidth, lineCap, lineJoin, miterLimit, shadowOffsetX, shadowOffsetY, shadowBlur, shadowColor, globalCompositeOperation` 的值
* 当前的裁切路径(`clipping path`)

每一次调用 `restore` 方法，上一个保存的状态就从栈中弹出，所有设定都恢复。
通过[官方示例](https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial/Transformations)可以更好的理解 `save` 和 `restore` 的用法。

#### 移动

用来移动 `canvas` 和它的原点到一个不同的位置。

```js
// x 是左右偏移量，y 是上下偏移量
translate(x, y)
```

#### 旋转

以原点为中心旋转 `canvas`。旋转的中心点始终是 `canvas` 的原点，如果要改变它，我们需要用到 `translate` 方法。

```js
// 顺时针旋转 angle 弧度
rotate(angle)
```

#### 缩放

```js
// x，y分别表示横轴和纵轴的缩放因子
scale(x, y)
```

#### 变形

```js
transform(m11, m12, m21, m22, dx, dy);

// m11：水平方向上的缩放
// m12：水平方向上的倾斜偏移
// m21：竖直方向上的倾斜偏移
// m22：竖直方向上的缩放
// dx：水平方向上的移动
// dy：竖直方向上的移动


// 重置当前变形
resetTransform()；
```

#### 合成与裁剪

**合成**

```js
globalCompositeOperation = type
```

点击[这里](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API/Tutorial/Compositing/Example)查看具体类型及效果

**裁剪**

```js
clip()
```

点击[这里](https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial/Compositing)查看具体使用方式及效果


#### 像素操作

**ImageData 对象**

* width：图片宽度
* height：图片高度
* data：`Uint8ClampedArray` 类型的一维数组

**创建 ImageData 对象**

```js
// 1. 创建一个新的，具有特定尺寸的 ImageData 对象
var imageData = ctx.createImageData(width, height);

// 2. 你也可以创建一个被anotherImageData对象指定的相同像素的ImageData对象。
var imageData2 = ctx.createImageData(anotherImageData);
```

**获取场景像素数据**

```js
// 
var imageData = ctx.getImageData(left, top, width, height);
```

**写入像素数据**

```js
//
ctx.putImageData(imageData, dx, dy);
```
`dx`和`dy`参数表示你希望在场景内左上角绘制的像素数据所得到的设备坐标。

例如，为了在场景内左上角绘制`myImageData`代表的图片，你可以写如下的代码：

```js
//
ctx.putImageData(myImageData, 0, 0);
```

**反锯齿**

反锯齿默认是启用的，我们可能想要关闭它以看到清楚的像素。

```js
//
imageSmoothingEnabled = bool
```

#### 保存图片

`HTMLCanvasElement` 提供一个`toDataURL()`方法，此方法在保存图片的时候非常有用。它返回一个包含被类型参数规定的图像表现格式的`数据链接(base64)`。

```js
//
//
//
//
// 添加注释，解决博客bug
//
//
//
// 默认设定，创建一个 PNG 图片
canvas.toDataURL('image/png')

// quality 图片质量，取值 [0, 1]
canvas.toDataURL('image/jpeg', quality);

// 不常用，创建了一个在画布中的代表图片的Blob对像
// 具体信息：https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLCanvasElement/toBlob
canvas.toBlob(callback, type, encoderOptions)

```

### 结尾

以上就是 `canvas` 最常用的所有特性了。
另外，关于 `Canvas` 性能优化 你可以点击[这里](https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial/Optimizing_canvas)查看。
