---
title: Vuex Essay
date: 2020-03-11 14:24:36
tags: [Vue,Essay]
published: true
hideInList: false
feature: /post-images/vuex-essay.jpg
---
### 概要

* vuex **集中式**存储管理应用的所有组件的状态，并以相应的规则保证状态以**可预测**的方式发生变化。

	> 	vuex 与 vue-router 因为都采用 vue 的响应式机制来实现了对应 view 层的更新，所以两个都依赖 vue 才能使用

* 核心概念
	* state 状态数据
	* mutations 更改状态的函数，commit 触发
	* actions 异步操作，dispatch 触发
	* store 包含以上概念的容器

### 插件机制

* 这个之前 vue-router 写过，直接上代码

	```js
	const install = (_vue) => {
		let vue = _vue;

		vue.mixin({
			beforeCreate() {
				if (this.$options.store) {
					vue.prototype.$store = this.$options.store;
				}
			}
		});
	}

	if (type window === 'object' && window.vue) {
		install(window.vue);
	}
	```

### state

* 使用

	```js
	// store.js
	export default new Vuex.store({
		state: {counter: 0}
	});
	
	// page.vue
	console.log(this.$store.state.counter)
	```
	
* 实现

	```js
	class Store {
		constructor(options = {}){
			this._vm = new Vue({
				data: {
					$$state: options.state
				}
			});
		}

		get state() {
			return this._vm._data.$$state
		}

		set state(v) {
			console.error('you can not set state immediately');
		}
	}
	```

### mutations

* 使用

	```js
	export default new vuex.store({
		mutations: {
			add(state,payload) {
				state.counter ++
			}
		}
	});
	```

* 实现

	```js
	class Store {
		constructor(options = {}) {
			this._mutations = options.mutations || {};
		}

		commit(type, payload) {
			const entry = this._mutations[type];

			if (!entry) {
				console.error(`unknown mutation type: ${type}`);
				return ;
			}

			entry(this.state, payload);
		}
	}
	```

### actions

* 使用

	```js
	export default new vuex.Store({
		actions: {
			add({commit}, payload) {
				setTimeout(()=> {
					commit('add');
				}, 1000);
			}
		}
	});
	```

* 实现

	```js
	class Store {
		constructor(options = {}) {
			this._actions = options.actions || {}

			this.commit = this.commit.bind(this);
			this.action = this.action.bind(this);
		}

		dispatch(type, payload) {
			const entry = this._actions[type];

			if (!entry) {
				console.error(`unknown action type: ${type}`);
				return ;
			}

			return entry(this, payload);
		}
	}
	```
	
### getters

* 使用

	```js
	export default new vuex.Store({
		getters: {
			doubleCounter(state) {
				return state.counter * 2;
			}
		}
	});
	```

* 实现

	```js
	class Store {
		constructor(options = {}) {
			const _getters = options.getters || {}
			this._vm = new vue({
				computed: _getters
			});

			for(key in _getters) {
				Object.defineProperty(this.getters, key, {
					get: () => this._vm[key],
					enumerable: true // for local getters
				})
			}
		}
	}
	```



