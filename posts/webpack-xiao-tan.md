---
title: webpack 小谈
date: 2019-06-14 17:41:20
tags: [webpack]
published: true
hideInList: false
feature: /post-images/webpack-xiao-tan.jpg
---
最近多看了一些这方面的知识，算是入了门，在这里记录一些。

### code splitting

代码分割在wenpack 中共有三种方法：

* entry 配置多个入口文件
	方便，快捷，但缺点也很明显，首先每个 chunk 都可能含有重复的模块，其次这样做的细粒度不够，并不能做到代码层面的分割。
		
* splitChunks 抽取公有代码
	splitChunks 可以帮助我们解决重复模块的问题。
		
* 动态加载
	webpack 提供了两种方式来实现动态加载：
	* import()
		动态 import 导入是 es7 的语法，返回的是一个 promise ， 可以和 async 搭配服用，这也是 webpack 推荐的动态加载方式，需要注意的是，需要额外的 babel 插件 plugin-syntax-dynamic-import
	* require.ensure
		这是 webpack 实现的功能，但官方已不推荐使用了。
		
	在这里我们还需要了解一下 css 的代码分割，在 css 层面目前能做到的，就是抽取生成单独的 css 文件。目前有两个plugin 可以帮助我们很好的实现这一点： `extract-text-webpack-plugin@next` 和 `mini-css-extract-plugin`。
	
### Tree Shaking

* css tree shaking
实现 css 的 tree shaking 需要借助 `purifycss-webpack` 这个插件，它会帮助我们清除无用的 css 代码

	```javascript
	const PurifyCss = require('purifycss-webpack')
	const glob = require('glob-all')

	plugins: [
			new PurifyCss({
					paths: glob.sync([
							path.resolve(__dirname, './src/*.html'),
							path.resolve(__dirname, './src/*.js')
					])
			})
	]
	```
	
* js tree shaking
在 webpack 中想要实现 tree shaking 是一件很不容易的事情，在webpack 中我们是靠 UglifyJS 来清除无用代码的，然而 UglifyJS 只能根据代码是否有副作用而不是程序流分析来删除代码。本来我们是可以知道自己的代码是否有副作用的，但是通过 babel 转译后，我们自己也无法保证了。想要了解详情可以看看[这里](https://juejin.im/post/5a5652d8f265da3e497ff3de)。
	这里我们只能做到自己应该做的：
	* 我们开发的是依赖库
		* 使用 rollup 进行打包并且提供 ES6 版本，在 package.json 中提供 module 字段。
		* 打包单独的组件文件，让用户通过目录去加载。
	* 我们开发的是工程项目
		* webpack4 以后只需将 mode 设置为 production ，即可引入 UglifyJS 插件。
		* 在 optimization 中，设置：
			```javascript
			optimization: {
					usedExports: true
			}
			```
		
### DllPlugin

我们引入的第三方库在一般情况下都不会有更改，所以可以使用插件将库打包成 dll 文件，在需要时直接引用。
* 设置 dll config 文件
```javascript
module.exports = {
	mode: 'production',
	entry: {
		react: ['react']
	},
	output: {
		path: path.resolve(__dirname, './dll'),
		filename: '[name].vendor.js',
		library: '[name]_[hash]'     // manifest 里第三方库的名称
		libraryTarget: 'this'
	},
	plugins: [
		new webpack.DllPlugin({
			path: path.resolve(__dirname, './dll/[name]-manifest.json'),
			name: '[name]_[hash]'
		})
	]
}
```

* 在 package.json 中添加命令
	```javascript
	"build:dll": "webpack --config webpack.dll.config.js"
	```
	
* 使用 dll 文件
```javascript
 new AddAssetHtmlWebpackPlugin({     // 将 dll 文件引入 html
	 filepath: path.resolve(__dirname, './dll/react.vendor.js') // 对应的 dll 文件路径
 }),
 new webpack.DllReferencePlugin({
	 manifest: path.resolve(__dirname, './dll/react-manifest.json')
 })
```

### happypack

happypack 可以启用多个进程帮助我们并发处理打包任务，所以会快一些。

```javascript
const os = require('os');
const HappyPackThreadPool = HappyPack.ThreadPool({ size: os.cpus().length });
rules: [
	{
			test: /\.jsx?$/,
			exclude: /node_modules/,
			use: [
					{
							loader: 'happypack/loader?id=usedBabel'
					}
			]
	},
]
new HappyPack({
	id: 'usedBabel',
	loaders: ['babel-loader'],
	threadPool: HappyPackThreadPool
}),
```

### PWA

webpack 主要是用了 google workbox 的 loader ，只需要一些简单的配置，就可以实现 PWA ，当然根据项目的实际情况，我们需要具体分析。

```javascript
const WorkboxPlugin = require('workbox-webpack-plugin')
const prodConfig = {
  plugins: [
    new WorkboxPlugin.GenerateSW({
      clientsClaim: true,
      skipWaiting: true
    })
  ]
}

// 在入口文件添加
if ('serviceWorker' in navigator) {
  window.addEventListener('load', () => {
    navigator.serviceWorker
      .register('/service-worker.js')
      .then(registration => {
        console.log('service-worker registed')
      })
      .catch(error => {
        console.log('service-worker registed error')
      })
  })
}
```

#### 以上

> 有些人的傲慢，来自于平时工作的严谨与负责




