# 03-css

## 1.盒子模型

> margin, border, padding, content

* 标准盒子：width = content-width；box-sizing: content-box
* 怪异盒子：width = border-width + padding + content-width；box-sizing: border-box

## 2.flex

指定 flex 容器：
> display: flex | inline-flex

* flex-direction: row | column | row-reverse | column-reverse(顺序反转)
* flex-wrap: nowrap | wrap | wrap-reverse(行翻转);
* justify-content: flex-start | flex-end | center | space-between | space-around;主轴上的对齐方式
* align-items: flex-start | flex-end | center | baseline(基线，以文字底部为主) | stretch(默认，以flex-direction:row 为例，如果未设置高度或者 auto，撑满整个容器高度);交叉轴的对齐方式
* align-content: flex-start | flex-end | center | space-between | space-around | stretch;(定义了多根轴线的对齐方式，如果项目只有一根轴线，那么该属性将不起作用，如果 flex-wrap: wrap，则起作用)

flex 子项：

* order: <integer>; 排序
* flex-basis: <length> | auto;(定义了在分配多余空间之前，项目占据的主轴空间，浏览器根据这个属性，计算主轴是否有多余空间)
* flex-grow: <number>; (默认为 0，定义项目的放大比例)
* flex-shrink: <number>; (定义了项目的缩小比例)
* flex: <number>; (flex-grow, flex-shrink 和 flex-basis的简写)
* align-self: auto | flex-start | flex-end | center | baseline | stretch;(允许单个项目有与其他项目不一样的对齐方式)

flex: 0 1 auto; 放大 0，缩小 1, 自动占用主轴空间。

## 3.css 优先级

> important > 内联 > id选择器 > 类选择器 > 标签选择器

### 3.1 浏览器怎样解析 css 选择器的？

> 从右向左解析。

## 4.避免 css 全局污染

* 使用 css module (生成唯一的类名)

## 5.动态设置样式

* 前缀 or react-context

## 6.CSS3 有哪些新特性？

* RGBA 和透明度
* background-image、background-size、background-repeat
* 文字阴影 text-shadow
* 圆角 border-radius
* 边框图片 border-image
* 盒子阴影 box-shadow

## 7.display:none 和 visibility:hidden 区别？

* display:none 不显示元素，不占空间，会导致重绘和回流
* visibility: hidden 隐藏元素，占空间

## 8.对 BFC 的理解

> BFC是指一个独立的渲染区域，只有Block-level Box参与。它规定了内部的Block-level Box如何布局，并且与这个区域外部毫不相干.。

定义：浮动元素和绝对定位元素，非块级盒子的块级容器（例如 inline-blocks, table-cells, 和 table-captions），以及overflow值不为"visiable"的块级盒子，都会为他们的内容创建新的BFC（Block Fromatting Context， 即块级格式上下文）。

BFC的触发条件：

* 根元素，即 html 元素
* 浮动元素（float 不为 none）、绝对定位元素（position: fixed/absolute）
* 行内块元素(display: inline-block)、表格单元格(display: table-cell)、表格标题(display: table-caption)
* overflow不为visible的块元素
* 网格元素(display: grid)

应用场景：

* 防止浮动导致父元素高度塌陷；
* 避免外边距折叠；（margin 重叠的原因就是这个）
* 两栏布局，防止文字环绕等

## 9.line-height 的理解？

> 行高是指一行文字的高度，具体说是两行文字间基线的距离。
> 多行文本垂直居中：需要设置display属性为inline-block。

## 10.怎么让 chrome 支持小于 12px 的文字？

* transform: scale(0.8);

## 11.浏览器刷新频率？

* 60hz，一秒刷新60次，最小间隔为 16.7ms = 1000 / 60

## 12.position

* absolute
* relative
* fixed
* static
* sticky
* initial
* unset

## 13.居中

* margin: 0 auto;
* display: flex; justify-content: center;
* display: table; margin: 0 auto;

## 14.高清问题 1px

> 产生原因：DPR(devicePixelRatio) 设备像素比，window.devicePixelRatio=物理像素 /CSS像素

目前主流的屏幕DPR=2 （iPhone 8）,或者3 （iPhone 8 Plus）。拿2倍屏来说，设备的物理像素要实现1像素，而DPR=2，所以css 像素只能是 0.5。一般设计稿是按照750来设计的，它上面的1px是以750来参照的，而我们写css样式是以设备375为参照的，所以我们应该写的0.5px就好了啊！ 试过了就知道，iOS 8+系统支持，安卓系统不支持。

解决方案：

* border: 0.5px; ios8+ 支持，安卓不支持。
* 边框图片; 颜色不可控，圆角模糊；
* 伪元素;

```css
.setOnePx{
  position: relative;
  &::after{
    position: absolute;
    content: '';
    background-color: #e5e5e5;
    display: block;
    width: 100%;
    height: 1px; /*no*/
    transform: scale(1, 0.5);
    top: 0;
    left: 0;
  }
}
.setBorderAll{
  position: relative;
  &:after{
    content:" ";
    position:absolute;
    top: 0;
    left: 0;
    width: 200%;
    height: 200%;
    transform: scale(0.5);
    transform-origin: left top;
    box-sizing: border-box;
    border: 1px solid #E5E5E5;
    border-radius: 4px;
  }
}
```

* 设置 viewport 的 scale 值。

```html
<html>
  <head>
    <title>1px question</title>
    <meta http-equiv="Content-Type" content="text/html;charset=UTF-8">
    <meta name="viewport" id="WebViewport" content="initial-scale=1, maximum-scale=1, minimum-scale=1, user-scalable=no">        
    <style>
      html {
        font-size: 1px;
      }
      * {
        padding: 0;
        margin: 0;
      }
      .top_b {
        border-bottom: 1px solid #E5E5E5;
      }

      .a,.b {
        box-sizing: border-box;
        margin-top: 1rem;
        padding: 1rem;                
        font-size: 1.4rem;
      }

      .a {
        width: 100%;
      }

      .b {
        background: #f5f5f5;
        width: 100%;
      }
    </style>
    <script>
      var viewport = document.querySelector("meta[name=viewport]");
      //下面是根据设备像素设置viewport
      if (window.devicePixelRatio == 1) {
        viewport.setAttribute('content', 'width=device-width,initial-scale=1, maximum-scale=1, minimum-scale=1, user-scalable=no');
      }
      if (window.devicePixelRatio == 2) {
        viewport.setAttribute('content', 'width=device-width,initial-scale=0.5, maximum-scale=0.5, minimum-scale=0.5, user-scalable=no');
      }
      if (window.devicePixelRatio == 3) {
        viewport.setAttribute('content', 'width=device-width,initial-scale=0.3333333333333333, maximum-scale=0.3333333333333333, minimum-scale=0.3333333333333333, user-scalable=no');
      }
      var docEl = document.documentElement;
      var fontsize = 32* (docEl.clientWidth / 750) + 'px';
      docEl.style.fontSize = fontsize;
    </script>
  </head>
  <body>
    <div class="top_b a">下面的底边宽度是虚拟1像素的</div>
    <div class="b">上面的边框宽度是虚拟1像素的</div>
  </body>
</html>
```

## 15.重绘与回流（重排）

> 回流必将引起重绘，重绘不一定会引起回流。

当 Render树 中部分或全部元素的尺寸、结构、或某些属性发生改变时，浏览器重新渲染部分或全部文档的过程称为回流。

* 改变窗口大小
* 改变字体
* 添加、修改样式
* 内容变化
* 伪类
* 设置style 等

当页面中元素样式的改变并不影响它在文档流中的位置时（例如：color、background-color、visibility等），浏览器会将新样式赋予给元素并重新绘制它，这个过程称为重绘。

## 16.水平垂直布局

* lineheight
* flex
* grid (和 flex 一样，但是兼容性不好)

## 17.层叠上下文

> 层叠上下文是HTML元素的三维概念，这些HTML元素在一条假想的相对于面向（电脑屏幕的）视窗或者网页的用户的z轴上延伸，HTML元素依据其自身属性按照优先级顺序占用层叠上下文的空间。

触发：

* 根元素 (HTML)
* opacity 属性值小于 1 的元素
* transform 属性值不为 "none"的元素
* z-index 值不为 "auto"的 绝对/相对定位，
* position: fixed

## 18.transform

* transform 不会触发重绘和回流，元素仍然占据原来的位置
* transform 会创建一个 GPU图层，使用 GPU渲染。
