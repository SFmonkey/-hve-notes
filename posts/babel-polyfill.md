---
title: babel polyfill
date: 2019-07-17 13:33:48
tags: [Babel]
published: true
hideInList: false
feature: 
---
在前端代码构建中，我们使用 babel 将 ES2015 + 的代码转换为低版本的代码，来保证各个浏览器端的正常运行。但是在没有任何插件的情况下，babel 只会帮我们转换语法，不会转换类和方法。

如果需要转换类和方法，就需要 polyfill 了。引入 polyfill ，主要有两种方法：

### @babel/polyfill

可以单独引入 @babel/polyfill, `import '@babel/polyfill'`，缺点很明显，**所有的 polyfill 都引进项目里了，可能会有几十K的大小，这个是没法忍的**

### @babel/preset-env

@babel/preset-env 搭配 @babel/polyfill 使用，可以根据 browserlists 来灵活的引用 polyfill。

@babel/preset-env 里面有个 option 是 useBuiltins：

* flase
	不启用 polyfill ，如果 `import '@babel/polyfill'`，会无视 browserlist 将所有的 polyfill 加载进来。
* entry
	启用，需要手动 `import '@babel/polyfill'`，这样会根据 browserlist 过滤出需要的 polyfill
* usage
	不需要手动 `import '@babel/polyfill'`，会根据 browserlist + 业务代码使用到的新 API 按需进行 polyfill
	
	**问题也有，那就是 polyfill 依旧是全局污染的，对于库或者组件代码，这样做是有风险的**
	
### @babel/plugin-transform-runtime (暂时有问题)

@babel/plugin-transform-runtime 需要搭配 @babel/runtime 使用，当然也是根据 preset 来判断。

@babel/plugin-transform-runtime 用于构建过程中，@babel/runtime 是实际导入项目代码的功能模块。

@babel/plugin-transform-runtime 做了三件事：

* 自动请求 @babel/runtime/regenerator 当你使用 generators/async 方法时
* 使用 core-js 假设用户使用了相关的 polyfill
* 移除内联的 helpers ，将使用的 helpers 抽成一个模块并 require 引入

好处是很明显的，此转换器将帮助你的代码创建沙盒环境，不会污染全局。

但问题也有，**@babel/plugin-transform-runtime 不会转换实例方法**，因为如果转换实例方法，势必要修改原型链，这和不污染全局的初衷相悖。而且在我个人看来，**根据 preset 来判断，也没有 bowerlists 那么精确**。

### polyfill.io

以上的方案其实都有牺牲使用最新浏览器的用户的体验，而 polyfill.io 只需引入一个文件，就会根据当前浏览器来加载需要的 polyfill 文件，**但是考虑到面对国内浏览器各种奇葩的 UA，会有一些风险存在**。

### 参考资料：
* [@babel/polyfill 与 @babel/plugin-transform-runtime 详解](https://github.com/SunshowerC/blog/issues/4)
* [babel能不能分析代码然后按需polyfill ？](https://juejin.im/post/5c09d6d35188256d9832df9d)
* [21 分钟精通前端Polyfill方案](https://segmentfault.com/a/1190000010106158)
* [babel-runtime使用与性能优化](https://juejin.im/entry/5b108f4c6fb9a01e5868ba3d)
* [@babel/plugin-transform-runtime](https://babeljs.io/docs/en/babel-plugin-transform-runtime#technical-details)
* [ES6和Babel你不知道的事儿](https://juejin.im/entry/5a290988f265da43062aa986)

