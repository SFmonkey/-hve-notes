---
title: 碎片化知识总结
date: Invalid date
tags: [blog]
published: true
hideInList: false
feature: 
---

这两天在写一些网页的时候,发现自己得到了一些琐碎的知识点,这种东西,还是赶快写下来比较好

## box-sizing

这个属性对我们平时开发的帮助之大,不言而喻,在这里涉及一个盒子的真正大小问题

    border + padding + width = 盒子的宽度
    border + padding + height = 盒子的高度

所以,我们在写一些盒子时,实际的大小往往偏大,这让我们还原设计时的工作量变的多了起来. box-sizing 帮助我们解决了这个问题,在这个属性下,如果我们设置了 padding 和 border,盒子的 content 将自动减小,来保证盒子的总大小是我们设置的 width 和 height 值.

我们常用的属性值是 border-box ,同样的,可以设置为 content-box , 这是浏览器的默认值,也就是 content 的宽高度. inherit 值用来继承父级元素的属性值.

> 该属性兼容 IE8+ ,FF 需要加 -moz- 前缀, 低版本的 ios 和 android 需要加 -webkit- (手机浏览器是 webkit 内核当道).

## 解决缩放图片的问题

很多时候,我们都会遇到图片的缩放问题.

1.在要保证不变形的情况下,我们只能将图片按照原宽高比进行缩放,在有的时候考虑到自适应的情况,我们可以利用 width 和 padding 的百分比来进行缩放.
2.利用 background-size 属性, cover 为等比扩容填满元素 , contain 等比缩小适应元素尺寸, auto 是图片自身的大小.

## 三栏式布局

我们在这里默认两边宽度不变

1.绝对定位法

左右两栏采用绝对定位,分别固定于页面的左右两侧,中间的主体栏用左右 margin 值撑开距离.

2.负 margin (双飞翼)

首先,中间的主体要使用双层标签.外层 div 宽度 100% 显示,并且浮动,
内层 div 为真正的主体内容,含有左右的 margin 值,左栏与右栏都是采用 margin 负值定位的, 左栏左浮动, margin-left 为 -100%,由于前面的 div 宽度100% , 所以这里的 -100% margin 值正好使左栏 div 定位到了页面的左侧,右侧栏也是做浮动,其 margin-left 也是负值,大小为其本身的宽度.

3.自身浮动法

左栏左浮动,右栏右浮动,主体直接放后面,加上margin 值 ,就实现了自适应.

## css 浮动

之所以写这个完全是因为看到了一篇好文,觉得很有必要记录

> 假如某个 div 元素A 是浮动的,如果 A 元素上一个元素也是浮动的,那么 A 元素会跟随在上一个元素的后边(如果一行方不下这两个元素,那么A 元素会被挤到下一行);如果 A 元素上一个元素是标准流中的元素,那么A 的相对垂直位置不会改变,也就是说A 的顶部总是和上一个元素的底部对齐. div 的顺序是 HTML 代码中 div 的顺序决定的.靠近页面边缘的一端是前,远离页面边缘的一端是后.

## 利用 css 绘制各种图形

1.圆

```css
#circle{
  width:100px;
  height:100px;
  background: red;
  -moz-border-radius:50px;
  -webkit-border-radius:50px;
  border-radius: 50px;
}
```

2.椭圆

```css
#oval{
  width: 200px;
  height: 100px;
  background: red;
  -moz-border-radius:100px/50px;
  -webkit-border-radius:100px/50px;
  border-radius: 100px/50px
}
```

3.三角形

```css
#triangle-up{
  width: 0;
  height: 0;
  border-left: 50px solid transparent;
  border-right: 50px solid transparent;
  border-bottom: 100px solid red;
}
```

4.平行四边形

```css
#parallelogram{
  width:150px;
  height: 100px;
  -webkit-transform:skew(20deg);
  -moz-transform:skew(20deg);
  -o-transform:skew(20deg);
  background: red;
}
```

5.半圆和四分之一圆

```css
#quartercircle{
  width: 200px;
  height: 200px;
  background: red;
  -webkit-border-radius:200px 0 0 0;
}
```

```css
#quartercircle {
width: 200px;
height: 200px;
background-color: red;
-webkit-border-radius: 0 0 200px 200px;
}
```

## centering in css

1.水平居中

(1) inline or inline-* elements

```
text-align:center
```

(2) block level element

```
margin:0 auto
```

(3) more than one block level element

in a row

```css
.inline-block-center{
  text-align:center
}

.inline-block-center div{
  display:inline-block;
  text-align:left
}

.flex-center{
  display: flex;
  justify-content: center
}
```

stacked on top of each other

```
margin:0 auto
```

2.垂直居中

(1) inline or inline-* elements

single inline

```css
.link{
  padding-top: 30px;
  padding-bottom: 30px
}

.link{/*height 固定*/
  height:30px;
  line-height:30px
}
```

multiple lines

```css
.center-table {
  display: table;
  height: 250px;
  background: white;
  width: 240px;
  margin: 20px;
}
.center-table p {
  display: table-cell;
  margin: 0;
  background: black;
  color: white;
  padding: 20px;
  border: 10px solid white;
  vertical-align: middle;
}

.flex-center-vertically {
  display: flex;
  justify-content: center;
  flex-direction: column;
  height: 400px;
}

.ghost-center {
  position: relative;
}
.ghost-center::before {
  content: " ";
  display: inline-block;
  height: 100%;
  width: 1%;
  vertical-align: middle;
}
.ghost-center p {
  display: inline-block;
  vertical-align: middle;
}
```

(2) block elements

know width and height

```css
.parent {
  position: relative;
}
.child {
  position: absolute;
  top: 50%;
  height: 100px;
  margin-top: -50px; /* account for padding and border if not using box-sizing: border-box; */
}
```

do not know

```css
.parent {
  position: relative;
}
.child {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
}
```

flex

```css
.parent {
  display: flex;
  flex-direction: column;
  justify-content: center;
}
```

3.水平垂直居中

know width and height

```css
.parent {
  position: relative;
}

.child {
  width: 300px;
  height: 100px;
  padding: 20px;

  position: absolute;
  top: 50%;
  left: 50%;

  margin: -70px 0 0 -170px;
}
```

```css
.parent {
  position: relative;
}
.child {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
```

```css
.parent {
  display: flex;
  justify-content: center;
  align-items: center;
}
```

参考资料:
[让CSS布局更加直观：box-sizing](http://web.jobbole.com/82331/)

[深入理解CSS中的行高](http://www.cnblogs.com/rainman/archive/2011/08/05/2128068.html)

[我熟知的三种三栏网页宽度自适应布局方法](http://www.zhangxinxu.com/wordpress/2009/11/%E6%88%91%E7%86%9F%E7%9F%A5%E7%9A%84%E4%B8%89%E7%A7%8D%E4%B8%89%E6%A0%8F%E7%BD%91%E9%A1%B5%E5%AE%BD%E5%BA%A6%E8%87%AA%E9%80%82%E5%BA%94%E5%B8%83%E5%B1%80%E6%96%B9%E6%B3%95/)

[经验分享：CSS浮动(float,clear)通俗讲解](http://www.cnblogs.com/iyangyuan/archive/2013/03/27/2983813.html)

[The Shapes of CSS](https://css-tricks.com/examples/ShapesOfCSS/)

[CSS3:border-radius隐藏的威力](http://www.xincss.com/?p=221)

[Centering in CSS: A Complete Guide](https://css-tricks.com/centering-css-complete-guide/)
