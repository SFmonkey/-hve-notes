---
title: Vue Router Essay
date: 2020-03-09 08:44:00
tags: [Vue,Essay]
published: true
hideInList: false
feature: /post-images/vue-router-essay.jpeg
---
### 概要

Vue Router 的实现大致分为三个部分

* 插件机制
* router-link
* router-view

Vue Router 一共有三种模式， hash（默认），history 以及 abstract（支持 Nodejs 运行环境） ，我们这里暂且只谈 hash 模式。

### 插件机制

Vue Router 是作为**插件**的形式接入到 Vue 中的，所以我们在实现 Vue Router 的具体功能前，需要先实现 Vue 插件机制。

* 插件主要的我们需要实现一个 install 函数

	```js
	export const install = () => {
		Vue.mixin({
			beforeCreate() {
				if (this.$options.router) {
					Vue.prototype.$router = this.$options.router;
				}
			}
		});
		Vue.component('RouterView', RouterView);
		Vue.component('RouterLink', RouterLink);
	}
	```
	
	> 	这里之所以要混入一个 beforeCreate ，是想要判断一下 new Vue 时是否添加了 Router 实例，从而添加全局 Router 实例
	
* 当然为了应对 Vue 可能 Script 标签引入的方式，我们还需要判断处理下这种情况

	```js
	if (typeof window !== undefined && window.vue) {
		install(window.vue);
	}
	```
	
### router-link

* **Router-LInk** 底层其实也就是个 a 标签，因为是 hash 模式，to 属性我们会直接传给 a 标签的 href。

* 核心代码如下

	```js
	vue.component('RouterLink', {
		props: {
				to: String, // 还可能是 Object
				required: true
		},
		//这里不能使用 template ，因为我们引入的 Vue 一般是 runtime 版本，没有 compiler，无法解析 template
		//因为这里是个js文件，如果是 vue 文件的话，vue-loader 还可以帮助我们解析 template
		render(h) { 
			return h('a', {attrs: {href: this.to}}, this.$slots.default);
		}
	});
	```
	
### router-view

* **router-view** 是帮助我们根据路由进行选择性展示 UI 的组件。


* 因为 Vue Router 是根据 routes 表来进行路由配置的，我们需要根据当前的 hash 来与路由表进行匹配。

	```js
	// VueRouter.js
	match(routes) {
		routes = routes || this.$options.routes;

		for (const route of routes) {
			if (route.path === "/" && this.current === "/") {
				this.matched.push(route);
				return;
			}

			if (route.path !== "/" && this.current.indexOf(route.path) !== -1) {
				this.matched.push(route);
				if (route.children) {
					this.match(route.children);
				}
				return;
			}
		}
	}
	```
	
	> 	之所以用数组来存储匹配的路由，是因为嵌套路由的存在

* 同样是嵌套路由的关系，我们需要一个 depth 变量，来帮助我们找到对应的子路由

	```js
	export default {
		render () {
			// 标记该组件是否为 Router View 组件
			this.$vode.data.routerView = true

			let depth = 0;
			
			const parent = this.$parent;
			while(parent) {
				const vnodeData = parent.$vnode && parent.$vnode.data;
				if (vnodeData && vnodeData.routerView) {
					depth++
				}
				parent = parent.$parent;
			}
	
			let component = null;
			const route = this.$router.matched[depth];
		
			if (route) {
				component = route.component;
			}

			return h(component);
		}		
	}
	```
	
* 如果 match 有了变化，我们就通知视图重新渲染，这就需要 match 是一个响应式数据

	```
	// VueRouter.js
	Vue.until.defineReactive(this, 'matched', []);
	```





