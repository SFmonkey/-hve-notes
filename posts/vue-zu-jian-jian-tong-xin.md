---
title: Vue 组件间通信
date: 2020-01-01 13:15:36
tags: []
published: true
hideInList: false
feature: /post-images/vue-zu-jian-jian-tong-xin.jpg
---
vue 组件间通信的方式有很多，有平时常常用到的，也有一些比较边界冷门的，所以有必要总结归纳一下，形成知识图谱，方便记忆。

我们可以用传递层级来区分记忆：

* 父传子
* 子传父
* 兄弟传递
* 任意组件间
* 祖先传后代

### 父传子

1. props
2. $parent
3. $attrs与$listener

### 子传父

1. $emit
2. $children
3. ref

### 兄弟传递

1. $parent
2. $root

### 任意组件间

1. bus （常用 vue 实例，bus 的特点是谁监听谁触发）
2. vuex

### 祖先传后代

1. provide/inject


#### 参考资料
[Vue 组件间通信六种方式（完整版）](https://juejin.im/post/5cde0b43f265da03867e78d3)