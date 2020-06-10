---
title: js 隐式转换
date: 2019-05-08 21:46:18
tags: []
published: true
hideInList: false
feature: 
---
对于弱类型语言，我们会经常遇到类型转换的情景， 如果搞不懂原理，会在遇到类型转换的时候只能凭借过往的经验处理，但心里还是没有一个清晰的认知。

js 的类型转换多发生在 `算数运算符`，`关系运算符`，`判断语句` 中，在这些场景中，类型会按照特定的规则，进行类型转换。我们来说说这些规则：

首先在算数运算符中，除了 + 号以外，其他的如 - * / ，都需要将运算符两边的类型转换为 Number，js 引擎通过 ToNumber 方法来进行类型转换。

如果算数运算符是 + 的话，如果有复杂类型，我们先将其转换为简单类型，在简单类型中（null, undefined, boolean,string,number）, 如果存在 string，我们将算数运算符两边转换为 string 对待。否则，将类型向 number 转换。

其次是关系运算符，我们将运算符两边转换为基本类型后，如果两个操作数都是数值，则进行数值比较，如果两个操作数都是字符串，则进行按位比较。如果操作数有数值，则进行另一操作数数值转换。因为可能会有字符串与数值比较的情况，字符串转换为 NaN，NaN 与数字比较时，总是返回 false 。

这里我们再讲一下 == 。双等运算符判断稍显复杂，如果类型相等，则直接比较，复杂类型需要看引用地址是否相同。如果类型不相等，如果有复杂类型，则先进行拆箱操作，转换为基本类型，接着就成了基本类型的比较了。第一种是数字和字符之间的比较，如果左操作数中是数字，则将另一操作数转换为数字，如果是字符，则自己转为数字。第二种是如果不是字符和数字的比较，其中有一个为boolean，则将boolean转换为数字。第三种，如果两个为null或者undefined,则返回true，当然如果只有一个，那就不说了，不要迷糊了。详细参见：

[JavaScript隐式转换与拆箱](https://github.com/ck18781145809/blog/issues/2)

这里还有几道练习题，可以试试看：

[前端进阶系列（第3期）：常见的面试题 — 隐式类型转换](https://www.hankewins.com/blog/%E5%89%8D%E7%AB%AF%E8%BF%9B%E9%98%B6%E7%B3%BB%E5%88%97%EF%BC%88%E7%AC%AC3%E6%9C%9F%EF%BC%89%EF%BC%9A%E5%B8%B8%E8%A7%81%E7%9A%84%E9%9D%A2%E8%AF%95%E9%A2%98-%E2%80%94-%E9%9A%90%E5%BC%8F%E7%B1%BB%E5%9E%8B%E8%BD%AC%E6%8D%A2/)





