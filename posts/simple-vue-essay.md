---
title: Simple Vue Essay
date: 2020-03-13 10:05:16
tags: [Vue,Essay]
published: true
hideInList: false
feature: /post-images/simple-vue-essay.jpeg
---
### 概要

在实现 Vue 之前，我们先要清楚 Vue 的概念，真正的知道它是什么，然后才能知道怎么去实现。

* Vue 是一套用于构建用户界面（UI）的渐进式框架，它的核心库只关注视图层。

* Vue 是一个 MVVM 框架。

	实现 MVVM 主要有三个核心点：

	* 数据响应式：监听数据变化并在视图中更新

		* Object.defineProperty()
		
		* Proxy
	
	* 模版引擎：提供描述视图的模版语法

		* 插值：{{}}

		* 指令：v-bind，v-on，v-model，v-for，v-if

	* 模版渲染：如何将模版转换为 html

		* 模版 -> vdom -> dom

	所以，如果我们去实现，可以从 MVVM 结构，三个核心点去着力实现。
	
### 数据响应式

这里我们只讲 defineProperty 实现的数据响应式。

* 首先我们需要一个函数，去帮助我们给数据赋予响应式的能力

	```js
	function defineReactive(obj,key,val) {
		Object.defineProperty(obj, key, {
			get() {
				console.log(`get ${key}:${val}`);
				return val;
			},
			set(newVal) {
				if (newVal !== val) {
					console.log(`set ${key}:${newVal}`);
					val = newVal;
				}
			}
		});
	}
	```
	
* 当我们结合视图时，我们就需要一个更新函数了

	```html
	<html>
		<head></head>
		<body>
			<div id="app"></div>
			<script>
					function defineReactive(obj,key,val) {
						Object.defineProperty(obj, key, {
							get() {
								console.log(`get ${key}:${val}`);
								return val;
							},
							set(newVal) {
								if (newVal !== val) {
									console.log(`set ${key}:${newVal}`);
									val = newVal;
									update();
								}
							}
						});
					}
					
					const obj = {};
					defineReactive(obj, 'foo', '');
					obj.foo = new Date().toLocaleTimeString();
				
					//! 更新函数
					function update(val) {
						app.innerText = obj.foo;
					}
				
					setInterval(() => {
						obj.foo = new Date().toLocaleTimeString();
					}, 1000);
			</script>
		</body>
	</html>
	```

* 当然，用户不可能只传入一个属性，我们需要对传入的数据进行遍历

	```js
	function observe(obj) {
		// 当传入的数据不是对象或者为空时，返回
		if (typeof obj !== 'object' || obj == null) {
			return;
		}
		
		Object.keys().forEach(key= > defineReactive(obj, key, obj[key]));
	}

	function defineReactive(obj,key,val) {
		observe(val); //! val 是对象的情况
		Object.defineProperty(obj, key, {
			get() {
				console.log(`get ${key}:${val}`);
				return val;
			},
			set(newVal) {
				if (newVal !== val) {
					console.log(`set ${key}:${newVal}`);
					observe(newVal)//! 赋值是对象的情况
					val = newVal;
					update();
				}
			}
		});
	}
	```
	
* 还有一种情况，就是我们给对象添加新的属性,这时候，我们需要手动为其赋予响应式能力

	```js
	function set(obj, key, val) {
		defineReactive(obj, key, val)
	}
	```

### 模版引擎以及渲染

* 这里我们不去考虑如何实现模版引擎，以及 模版-> vnode -> dom 的过程中 vnode 的部分，只考虑 模版 -> dom。

	```js
	class Compile {
		constructor(el, vm) {
			this.$vm = vm;
			this.$el = document.querySelector(el);

			if (this.$el) {
				this.compile(this.$el);
			}
		}

		compile(el) {
			const childNodes = el.childNodes;

			Array.from(childNodes).forEach(node => {
				if (this.isElement(node)) {
					this.compileElement(node);
				} else if (this.isInterpolation(node)) {
					this.compileText(node);
				}

				if(node.childNodes && node.childNodes.length > 0) {
					this.compile(node);
				}
			});
		}
	}
	```
	
* 关于 isElement，isInterpolation, compileElement, compileText 函数

	```js
	function isElement(node) {
		return node.nodeType === 1;
	}

	function isInterpolation(node) {
		return node.nodeType === 3 && /\{\{(.*)\}\}/.test(node.textContent);
	}

	function compileElement(node) {
		let nodeAttrs = node.attributes;
		Array.from(nodeAttrs).forEach(attr => {
			let name = attr.name;
			let value = attr.value;
			if (this.isDirective(name)) {
				let dir = name.substring(2);
				this[dir] && this[dir](node, value)
			}
		});
	}

	function compileText(node) {
		node.textContent = this.$vm[RegExp.$1]
	}

	function isDirective(attr) {
		return attr.indexOf("k-") === 0
	}
	```
	
* 当然，指令就很多，我们简单写两个

	```js
	function text(node, value) {
		node.textContent = this.$vm[value];
	}
	function html(node, value) {
		node.innerHTML = this.$vm[value];
	}
	```

* 在 compile 的过程中，我们需要进行依赖收集，这里我们实现的是 Vue1.0 时候的依赖收集，一个依赖一个watcher，一个数据响应式的 Key 一个 Dep
	
	*  watcher 类
	
		```js
		// watcher
		class watcher {
			constructor(vm, key, updateFn) {
				this.vm = vm;
				this.key = key;
				this.updateFn = updateFn;

				// 订阅到 dep
				Dep.target = this;
				this.vm[this.key];
				Dep.target = null;
			}

			update() {
				this.updateFn.call(this.vm, this.vm[this.key]);
			}
		}
		```
	
	* 在生成 dom 时订阅 watcher
	
		```js
		compileText(node) {
			this.update(node, RegExp.$1, 'text');
		}

		update(node, exp, 'text') {
			const fn = this[dir + 'Updater']
			fn && fn(node, this.vm[exp])

			new watcher(this.vm, exp, function(val) {
				fn && fn(node, val)
			})
		}
		```
	
	* Dep 类
	
		```js
		//Dep
		class Dep {
			constructor() {
				this.deps = []
			}

			addDep(watcher) {
				this.deps.push(watcher);
			}

			notify() {
				this.deps.forEach(watcher => watcher.update());
			}
		}
		```

	* 在数据响应式时，实例化 Dep

	```js
	defineReactive() {
		...
		
		const dep = new Dep();
		
		Object.defineProperty(obj, key, {
			get() {
				Dep.target && dep.addDep(Dep.target);
				return val;
			},
			set(newVal) {
				if (newVal !== val) {
					observe(newVal)//! 赋值是对象的情况
					val = newVal;
					dep.notify();
				}
			}
	}
	```
	
### 总结

1. 	defineReactive 为每一个 key 创建一个 dep
2. 	初始化视图的时候读取某个数据，创建一个watcher
3. 	由于在watcher中读取了key，便将 watcher 订阅到了 dep 中
4. 	当某个key更新时，触发setter，相关dep 通知所有watcher更新

	![](https://sfmonkey.github.io//post-images/1584324343056.jpg)


	

