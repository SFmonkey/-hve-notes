---
title: Webpack Essay
date: 2020-03-25 08:56:36
tags: [webpack,Essay]
published: true
hideInList: false
feature: 
---
### 概要

webpack 是前端最常用的**模块打包工具**了，关于它的知识，我们从使用和原理两方面来讨论下。

* Webpack 使用
	* 基础配置
	* webpack 优化
* Webpack 原理
	* Loader 和 Plugin 的区别
	* 如何保证各个 loader 按照预想方式工作
	* Webpack 构建流程
	* 文件监听原理
	* Webpack 热更新原理
	* 代码分割的本质与意义
	* Babel 原理
* Webpack 实现
	* 实现一个简单的 Webpack
	* 实现一个Loader
	* 实现一个Plugin

### 使用

#### 基础配置

* Loader

	* JavaScript 相关
	
		* babel-loader
			
			将 ES6 + 的语法转换成 ES5
			
		* ts-loader
		
			将 Typescript 转换成 Javascript
		
		* awesome-typescript-loader
			
			将 TypeScript 转换成 JavaScript， 性能优于 ts-loader
	
		* eslint-loader

			通过 ESLint 检查 Javascript 代码
			
		* tslint-loader

			通过 tslint 检查 TypeScript 代码
			
		* vue-loader

			加载 Vuejs 单文件组件
			
	* CSS 相关

		* sass-loader

			将 SCSS/SASS 代码转换成 CSS
			
		* postcss-loader
			
			扩展 CSS 语法，使用下一代 CSS，例如可以配合 autoprefixer 插件自动补齐 CSS3 前缀
			
		* css-loader

			加载 CSS，支持模块化，压缩，文件导入等特性
			
		* style-loader

			将 CSS 代码通过 style 标签的方式加载进 html
			
	* 文件相关

		* file-loader

			将文件输出到一个文件夹中，在代码中通过相对 URL 去引用输出的文件（处理图片和字体）

		* url-loader

			与 file-loader 类似，区别是用户可以设置一个阈值，小于阈值时返回文件的 base64 形式编码（处理图片和字体）

		* raw-loader

			将文件内容通过字符串形式输出（utf-8）

	* 优化相关

		* cache-loader

			可以在一些性能开销较大的 Loader 之前添加，目的是将结果缓存在磁盘里

		* thread-loader

			与 cache-loader 使用效果与位置相同
				
* Plugins

	* 通用插件
	
		* clean-webpack-plugin 
		
			目录清理
			
		* html-webpack-plugin

			简化 HTML 文件创建（可以指定 template 以及自动引入入口文件）
			
		* webpack.ProvidePlugin

			定义全局变量，不必到处 require 或 import
			
		* HappyPack

			多进程打包
			
		* webpack.DefinePlugin

			定义环境变量（在项目代码中可能存在需要环境变量的情况）
	
	* 开发插件
			
		* HotModuleReplacementPlugin

			webpack 的热更新又称热替换（HMR），这个机制可以做到不用刷新浏览器而将新变更的模块替换掉旧的模块
			
	* 生产插件

		* mini-css-extract-plugin
			
			分离样式文件，CSS 提取为独立文件，支持按需加载
			
		* workbox-webpack-plugin

			为网页应用增加离线缓存功能
			
		* purifycss-webpack

			利用该插件进行 css treeshaking
			
		* webpack.DllReferencePlugin

			根据 DllPlugin 生成的 manifest 文件，对 bundle 文件进行映射
			
		* add-asset-html-webpack-plugin

		  将指定的静态文件写入 html 

		* webpack-bundle-analyzer

			可视化 webpack 输出文件的体积（业务组件，第三方组件）
			
* 其他配置
			
	* 文件指纹：文件指纹是打包输出的文件名的后缀，一共分三种
		* Hash
			和整个项目的构建有关，只要项目文件有修改，整个项目构建的hash值就会更改
			
			设置 file-loader 的name，使用hash
			
		* Chunkhash
			和 webpack 打包的 chunk 有关，不同的 entry 会生出不同的 chunkhash
			
			设置 output 的 filename， 用 chunkhash
			
		* Contenthash
			根据文件内容来定义 hash，文件内容不变，则 contenthash 不变
			
			设置 MiniCssExtractPlugin 的 filename， 使用 contenthash
			
	* optimization
		
#### webpack 优化

* 提升webpack构建速度

	* 多进程/多实例构建
	
		HappyPack 和 thread-loader
	
	* 多进程压缩代码
		
		terser-webpack-plugin 开启 parallel 参数
		
	* 缩小打包作用域
		* exclude/inlcude （确定 loader 规则范围）
		* resolve.modules 指明第三方模块的绝对路径
		* resolve.mainFields 只采用 main 字段作为入口文件描述字段
		* resolve.extensions 尽可能减少后缀尝试的可能性
		* 合理使用alias

	* DLL

		使用 DllPlugin 进行分包，使用 DllReferencePlugin 对 manifest.json 引用，让一些基本不会改动的代码先打包成静态资源，避免反复编译
		
	* 充分利用缓存提升二次构建速度

		* babel-loader 开启缓存
		* terser-webpack-plugin 开启缓存
		* cache-loader

* 提升页面加载速度

	* 动态 Polyfill
		[babel polyfill](https://sfmonkey.github.io/post/babel-polyfill/)
	* Scope hoisting
		* 构建后的代码会存在大量闭包，造成体积增大，运行代码时创建的函数作用域变多，内存开销变大。Scope hoisting 将所有模块的代码按照引用顺序放在一个函数作用域里，然后适当的重命名一些变量以防止变量名冲突。
			```js
			//开启前
			[
					/* 0 */
					function(module, exports, require) {
							var module_a = require(1)
							console.log(module_a['default'])
					}

					/* 1 */
					function(module, exports, require) {
							exports['default'] = 'module A'
					}
			]

			//开启后,通过减少闭包和函数声明，来减少代码体积和内存开销
			[
					function(module, exports, require) {
							var module_a_defaultExport = 'module A'
							console.log(module_a_defaultExport)
					}
			]
			```
		* 必须是 ES6 的语法，因为有很多第三方库仍采用 CommonJs 语法，为了充分发挥 Scope hoisting 的作用，需要配置 mainFields 对第三方模块优先采用 main 指向 ES6 模块化语法
	* Tree shaking
		* 打包过程中检测工程中没有引入过的模块进行标记，在资源压缩时将它们从最终的 bundle 中去掉（[只能对 ES6 module 生效](https://juejin.im/post/5a5652d8f265da3e497ff3de)），开发中尽可能使用 ES6 module 的模块，提高 tree shaking 效率。
		* 禁用 babel-loader 的模块依赖解析，否则 webpack 接收到的就都是转换过的 CommonJS 形式的模块，无法进行 tree-shaking
		* 使用 PurifyCSS（不再维护）或者 uncss 去除无用 css 代码
	* 提取页面公共资源
		* 将基础包通过 externals 或者 dll 方式 CDN 引入，不打入 bundle 中
		* 使用 SplitChunksPlugin 进行代码分割
	* 图片压缩，配置 image-webpack-loader
	* 提取 CSS 文件，通过css-loader的 minimize 选项开启 cssnano 压缩 CSS
	
### webpack 原理

* Loader 和 Plugin 的区别

	**Loader 本质就是一个函数**，在该函数中对接收到的内容进行转换，返回转换后的结果。因为 webpack 只认识 JavaScript，所以 Loader 就成了翻译官，对其他类型的资源进行转译的预处理工作。
	
	**Plugin 就是插件，插件可以扩展 webpack 的功能**，在 webpack 运行的生命周期中会广播出许多事件，Plugin 可以监听这些事件，在合适的时机通过 webpack 提供的 API 改变输出结果。
	
	**Loader** 在 module.rules 中配置，作为模块的解析规则，类型为数组。每项都是一个 Object， 内部包含了 test，loader，options 等属性。
	
	**Plugin** 在 plugins 中单独配置，类型为数组，每一项是一个 Plugin 的实例，参数都通过构造函数传入。
	
* 如何保证各个 loader 按照预想方式工作

	rules 中的每个 Object 都可以配置一个key **enforce**，value 一般有两个 **pre** **post**，pre 代表在所有正常 loader 之前执行，post 在所有 loader 之后执行。
	
* Webpack 构建流程

	webpack 的运行流程是一个串行的过程，从启动到结束会依次执行以下流程：
	
	* 读取 webpack 配置
	
		* 从配置文件中读取参数

	* 分析入口文件

		* 分析依赖模块
		* 分析入口文件内容
		* 编译内容

	* 依赖模块

		* 分析依赖模块
		* 分析本文件内容
		* 编译内容
		
	* 生成 bundlejs

* 文件监听原理

	文件监听是指 **webpack --watch**  或者 config 中配置 **watch 属性为 true**

	缺点： 每次都需要手动更新浏览器
	
	原理： 轮询判断文件的最后编辑时间是否变化，如果某个文件发生变化，并不会立刻告诉监听者，而是先缓存起来，等 **aggregateTimeout** 后再执行。
	
	```js
	module.export = {    
		// 默认false,也就是不开启    
		watch: true,    
		// 只有开启监听模式时，watchOptions才有意义    
		watchOptions: {        
		// 默认为空，不监听的文件或者文件夹，支持正则匹配       
		ignored: /node_modules/,        
		// 监听到变化发生后会等300ms再去执行，默认300ms        
		aggregateTimeout:300,        
		// 判断文件是否发生变化是通过不停询问系统指定文件有没有变化实现的，默认每秒问1000次        
		poll:1000    
	}}
	```
	
	
* Webpack 热更新原理

	HMR 可以做到在不刷新浏览器的情况下用新变更的模块替换掉旧的模块
	
	HMR 的核心就是客户端从服务器拉取更新后的文件，准确的说是 chunk diff（chunk 需要更新的部分），**实际上 WDS 与浏览器之间维护了一个 Websocket，当本地资源发生变化时，WDS 会向浏览器发送更新通知**，并带上构建时的 hash，让客户端与上一次资源进行对比。**客户端对比出差异后会向 WDS 发送 Ajax 请求来获取更改内容**（文件列表，hash），这样客户端就可以再借助这些信息继续向 WDS 发送 jsonp 请求获取该 chunk 的增量更新。**客户端拿到增量更新后，由 HotMoudlePlugin 提供相关 API 以供开发者针对自身场景进行处理**，react-hot-loader 和 vue-loader 都是借助这些 API 实现的 HMR

	> WDS->websocket->客户端->Ajax->更新内容->HotMoudlePlugin->react-hot-loader/vue-loader->HMR

	* 代码分割的本质与意义

	代码分割本质其实就是在 **源代码直接上线** 和 **打包成唯一脚本** 这两种极端方案之间的一种更适合实际场景的中间状态。

* Babel 原理

	* 解析：将代码转换成 AST
		* 词法分析：将代码字符串分割为 token 流，即语法单元组成的数组
		* 语法分析：分析 token 流（上面生成的数组）并生成 AST
	* 转换：访问 AST 的节点进行变换操作生产新的 AST
	* 生成：以新的 AST 为基础生成代码

### Webpack 实现

#### 实现一个简单的 Webpack

原理：

1. 读取 webpack 配置
2. 分析入口模块
	* 获取入口依赖
	* 获取入口内容
	* 转译入口内容
3. 分析依赖模块
	* 获取模块依赖
	* 获取模块内容
	* 转译模块内容
4. 得到 bundle.js

实现：

```js
const path = require('path');
const fs = require('fs');
const babel = require('@babel/core');
const parser = require('@babel/parser');
const traverse = require('@babel/traverse').default;


class compiler {
    constructor(opt) {
        this.opt = opt;
    }

    run() {
        const {entry, output} = this.opt;

        const res = this.generateCode(entry);

        fs.writeFileSync(path.resolve(output.path, output.filename), res);
    }

    moduleAnalyze(filename) {
        const content = fs.readFileSync(filename, 'utf-8');
        // 分析内容
        const ast = parser.parse(content, {
            sourceType: "module"
        });

        const dependencies = {};
        // 分析依赖模块路径
        traverse(ast, {
            ImportDeclaration({node}){
                const dirname = path.dirname(filename);
                const genFile = './' + path.join(dirname, node.source.value);
                dependencies[node.source.value] = genFile;
            }
        });
        // 编译内容
        const {code} = babel.transformFromAst(ast, null, {
            presets: ['@babel/preset-env']
        });

        return {
            filename,
            dependencies,
            code
        }
    }

    graphAnalyze(entry) {
        // 分析入口模块
        const entryModule = this.moduleAnalyze(entry);
        const graphArr = [entryModule];

        // 分析依赖模块
        graphArr.forEach(v => {
            const {dependencies} = v;

            if (dependencies) {
                for(let key in dependencies) {
                    graphArr.push(this.moduleAnalyze(dependencies[key]));
                }
            }
        });
        
        const graph = {};

        graphArr.forEach(v => {
            graph[v.filename] = {
                dependencies: v.dependencies,
                code: v.code
            }
        });

        return graph;
    }

    // 生成结果代码
    generateCode(entry) {
        // 得到所有模块图谱
        const graph = JSON.stringify(this.graphAnalyze(entry));
        // 输出 bundle.js
        return `(function(graph){
            // 浏览器没有 require ，所以我们需要实现一个
            function require(module) {
                function localRequire(relativePath) {
                    return require(graph[module].dependencies[relativePath]);
                }

                var exports = {};
                (function(require, exports, code){
                    eval(code);
                })(localRequire, exports, graph[module].code);

                return exports;
            }

            require('${entry}')
        })(${graph})`;
    }
}

module.exports = compiler;
```

#### 实现一个Loader

```js
const chalk = require('chalk');

module.exports = function(content) {
    const warning = chalk.keyword('orange');
    const { notice } = this.query;
    let words = ['todo', 'fixme', 'review'];

    if (notice && Array.isArray(notice) && notice.length > 0) {
        words = notice;
    }

    const reg = new RegExp(`(\/\/\\s*(${words.join("|")}).*$)|(\/\\*[\\s\\S]*(${words.join("|")})[\\s\\S]*\\*\/)`, "igm");
    const res = content.match(reg)

    if (res && res.length > 0) {
        const len = res.length;

        this.emitWarning(warning(`

           
        ███╗   ██╗ ██████╗ ████████╗██╗ ██████╗███████╗
        ████╗  ██║██╔═══██╗╚══██╔══╝██║██╔════╝██╔════╝
        ██╔██╗ ██║██║   ██║   ██║   ██║██║     █████╗  
        ██║╚██╗██║██║   ██║   ██║   ██║██║     ██╔══╝  
        ██║ ╚████║╚██████╔╝   ██║   ██║╚██████╗███████╗
        ╚═╝  ╚═══╝ ╚═════╝    ╚═╝   ╚═╝ ╚═════╝╚══════╝
                                               
   
                                        
        There are ${len} ${len > 1 ? 'comments' : 'comment'} you might to notice in ${chalk.green(this.resourcePath)}
        
        `));
    }

    return content;
}
```

#### 实现一个Plugin

* 一个JavaScript类函数
* 在函数原型 (prototype)中定义一个注入compiler对象的apply方法
* apply函数中通过compiler插入指定的事件钩子,在钩子回调中拿到compilation对象
* 使用compilation操纵修改webapack内部实例数据
* 异步插件，数据处理完后使用callback回调

**Compiler 和 Compilation 的区别在于：Compiler 代表了整个 Webpack 从启动到关闭的生命周期，而 Compilation 只是代表了一次新的编译**

```js
class EndPlugin {
    constructor(options) {
        const {doneCb, failCb} = options;

        this.doneCallback = doneCb;
        this.failCallback = failCb;
    }

    apply(compiler) {
        compiler.plugin('done', stats => {
            this.doneCallback && this.doneCallback(stats);
        });

        compiler.plugin('failed', error => {
            this.failCallback && this.failCallback(error);
        });
    }
}

module.exports = EndPlugin;
```


### 参考资料

* [「吐血整理」再来一打Webpack面试题🔥(持续更新)](https://juejin.im/post/5e6f4b4e6fb9a07cd443d4a5#heading-16)
* [Webpack 4进阶--从前的日色变得慢 ，一下午只够打一次包](https://zhuanlan.zhihu.com/p/35407642)
* [手写一个webpack插件](https://segmentfault.com/a/1190000019010101)
* [玩转webpack之loader开发](https://imweb.io/topic/5d4a94a08db073cf44ca8cd0)
* [webpack loader和plugin编写](https://juejin.im/post/5bbf190de51d450ea52fffd3)


	
