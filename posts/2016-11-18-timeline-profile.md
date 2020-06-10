---
title: 利用 Chrome DevTools 进行网页性能分析
date: Invalid date
tags: [blog]
published: true
hideInList: false
feature: 
---

前端发展到现在，不仅要将设计师的原型图完美展现，还要考虑到响应式、性能优化等诸多方面,所以性能分析是现在每个前端工程师的基础技能，从而来优化自己的页面，
达到最佳用户体验的目的，但有时面对诸多的前端性能分析工具，我们会不知所措，其实很多时候，Chrome 开发者工具就可以帮助我们解决问题。它的 Timeline 和 Profiles 
版块可以帮助我们很好的分析页面性能。

> 以下的 Chrome Devtools 均需要在隐身模式下服用，来保证功能的完备性和准确性。尽可能缩短记录时间和不必要的操作，关闭浏览器缓存。

## Timeline (版本 54.0.2840.71)


> Use the Chrome DevTools Timeline panel to record and analyze all the activity in your application as it runs. It's the best place to start investigating perceived performance issues in your application

![](http://121.42.215.169/blog_images/timeline-10.png)

Timeline 仪表盘包括四个窗口：

![](http://121.42.215.169/blog_images/timeline-annotated.png)

* Controls：开始记录，停止记录，配置在记录中需要捕获的信息。
* Overview：性能总结分析页面。
* Flame Chart：一个可视化的 CPU 堆栈跟踪。
* Details： 当事件被选择时，这个窗口将展示这个事件的详细信息，当没有事件被选择时，展示所选时间段的信息。

Overview 窗口 

Overview 窗口包含三个曲线图：

![](http://121.42.215.169/blog_images/overview-annotated.jpg)

* FPS：Frames Per Second ，绿色的线条越高，FPS 越高。在 FPS 曲线上面的红色块表示长帧，即 FPS 低于 30。
* CPU：CPU 资源，这个窗口显示各种类型事件所消耗的 CPU 资源。
* NET：每种颜色代表一种资源。

各个线条的颜色如下：

* HTML 文件是蓝色。
* scripts 是黄色。
* stylesheets 是紫色。
* Media 文件是绿色。
* 混杂资源是灰色。

Flame Chart 窗口

![](http://121.42.215.169/blog_images/timeline-show.png)

* Interactions ： 在 recording 过程中发生的交互操作，比如 MouseDown。
* Main：记录主线程
* Raster：线程光栅图
* GPU：记录 GPU

Details 窗口：

当我们选择一个事件，Details 窗口展示的是这个事件的附加信息。除过Summary 标签，它展示的是各个事件类型的详细情况。

我们利用下面的程序来进行一次 Timeline 的操作实例

我们以 Chrome Developers 中的一个例子来操作 Timeline 面板

```javascript
var x = [];

function createSomeNodes() {
    var div,
        i = 100,
        frag = document.createDocumentFragment();
    for (;i > 0; i--) {
        div = document.createElement("div");
        div.appendChild(document.createTextNode(i + "-"+ new Date().toTimeString()));
        frag.appendChild(div);
    }
    document.getElementById("nodes").appendChild(frag);
}
function grow() {
    x.push(new Array(1000000).join('x'));
    createSomeNodes();
    setTimeout(grow,1000);
}
grow();
```

> When grow is invoked it will start creating div nodes and appending them to the DOM. It will also allocate a big array and append it to an array referenced by a global variable. This will cause a steady increase in memory that can be found using the tools mentioned above.
Garbage collected languages usually show a pattern of oscillating memory use. This is expected if code is running in a loop performing allocations, which is the usual case. We will be looking for periodic increases in memory that do not fall back to previous levels after a collection.


运行这个脚本网页后，我们打开 Timeline, record 记录 10s 运行情况后停止。同时记得记录结束后给 grow 函数加上断点，防止页面崩溃。
在 overview 中选择一个时间段，观察 flame chart 中的大致情况，出现红色长块，说明这里出现了长帧。点击红色长块，可以在 Details 窗口查看详细信息。flame chart 的 main 里面有更加详细的事件顺序和消耗时间。同样点击后在 Details 窗口可显示详细信息。

从下面的线性图中我们可以清晰的看出内存在不断的增加，这会导致页面最终崩溃。

![](http://121.42.215.169/blog_images/timeline-node.png)

可以看出，Nodes 和 JS Heap 都在不断上升，中间下降的那段是因为 js 的垃圾回收机制起的作用，但很明显，内存还是在周期性的泄露，这是一个危险的标志。那么我们如何来找到和解决这个问题呢？

## Profiles (版本 54.0.2840.71)

> Use the Profiles panel if you need more information than the Timeline provide, for instance to track down memory leaks.

![](http://121.42.215.169/blog_images/take-heap-snapshot.png)

在 Profiles 面板上有四个配置类型：

* Record JavaScript CPU Profile：监控函数执行期花费的时间
* Take Heap Snapshot：保存内存快照
* Record Allocation Timeline：帮助你在 JS 堆里追踪内存泄露
* Record Allocation Profile：通过 JS 函数帮助你查看内存分配

同样，我们用一个例子来说明问题：

``` javascript
<!DOCTYPE html>
<html>
<body>
<script type="text/javascript">
                var ClassA = function(name){
                        this.name = name;
                        this.func = null;
                };
                var a = new ClassA("a");
                var b = new ClassA("b");
                b.func = bind(function(){
                        console.log("I am " + this.name);
                }, a);
                b.func();  //输出 I am a

                a = null;        //释放a
                //b = null;        //释放b
                //模拟上下文绑定
                function bind(func, self){
                                return function(){
                                return func.apply(self);
                        };
                }; 
</script>
</body>
</html>
```

Record JavaScript CPU Profile:

![](http://121.42.215.169/blog_images/profile-cpu.png)

这个面板的功能比较简单，从上图可以看到点击按钮执行的各个函数执行的时间，顺序，包含关系和CUP变化等

Take Heap Snapshot:

![](http://121.42.215.169/blog_images/profile-snapshot.png)

这里解释几个名词：

* Constructor：表示所有通过该构造函数生成的对象
* Distance：对象到达 GC 根的最短距离
* Objects Count：对象的实例数
* Shallow size：对应构造函数生成的对象的 shallow sizes（直接占用内存）总数
* Retained size：展示了对应对象所占用的最大内存

关于GC根的解释有点麻烦：它指的是一个对象指针最开始指向的内存。可以用一张图来理解：

![](http://121.42.215.169/blog_images/progile-gc.jpg)

构成这张关系网的元素有两种：
Nodes：节点，对应一个对象，用创建该对象的构造方法来命名
Edges：连接线，对应着对象间的引用关系，用对象属性名来命名
根据上面的定义解释，图中的对象 6 到CG根的距离就是 3 。

![](http://121.42.215.169/blog_images/profile-comparison.png)

可以看到有两个ClassA对象，这与我们的本意不相符，我们释放了a，应该只存在一个ClassA对象b才对。

当 b.func = null 时

![](http://121.42.215.169/blog_images/profile-snapshot-extra.png)

从上面两个图可以看出这两个对象中，一个是 b，另一个并不是 a，因为 a 这个引用已经置空了。第二个 ClassA 对象是bind里的闭包的上下文 self，self 与 a 引用同一个对象。虽然 a 释放了，但由于 b 没有释放，或者 b.func 没有释放，使得闭包里的 self 也一直存在。要释放 self，可以执行 b=null 或者 b.func=null

![](http://121.42.215.169/blog_images/profile-snapshot-2.png)

这里需要对 shallow size 和 retained size 做一下解释：

>直接占用内存指的是对象本身占用的内存，因为对象在内存中会通过两种方式存在着，一种是被一个别的对象保留（我们可以说这个对象依赖别的对象）或者被Dom对象这样的原生对象隐含保留。在这里直接占有内存指的就是前一种。(通常来讲，数组和字符串会保留更多的直接占有内存)。而最大内存(Retained size)就是该对象依赖的其他对象所占用的内存

我们会发现在快照的 JS 方法中有红黄两种颜色，黄色即直接占用内存，红色即间接占用内存，我们需要关注这两种情况的内存回收情况。

一个快照有四个试图类型：

* Summary(概要) – 通过构造函数名分类显示对象；
* Comparison(对照) – 显示两个快照间对象的差异；
* Containment(控制) – 探测堆内容；
* Statistic(图形表)-用图表的方式浏览内存使用概要

通过 Comparison ，我们可以观察两个试图之间增加和删除的对象。

顺带提一句，Retainers 视图显示出了该对象被哪些对象引用了

Record Allocation Timeline:

关于这个面板，我们使用以下代码来进行演示：

```javascript
var x = [];

function grow() {
  x.push(new Array(1000000).join('x'));
}

document.getElementById('grow').addEventListener('click', grow);
```

![](http://121.42.215.169/blog_images/new-allocations.png)

每一个蓝条的出现都预示着内存泄露的可能，选择其中一个蓝条进行分析

![](http://121.42.215.169/blog_images/zoomed-allocation-timeline.png)



Record Allocation Profile:

![](http://121.42.215.169/blog_images/allocation-profile.png)

![](http://121.42.215.169/blog_images/object-details.png)

> 性能分析很重要，前端的前辈们在很长一段时间里处于黑暗的年代中。现在已经初现光明，所以我们是很幸运的。
