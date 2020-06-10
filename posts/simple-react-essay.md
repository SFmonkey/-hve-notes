---
title: Simple React Essay
date: 2020-03-16 11:20:13
tags: [React,Essay]
published: true
hideInList: false
feature: /post-images/simple-react-essay.jpg
---
### 概要

和 Vue 一样，在实现 React 之前，我们需要弄清楚 React 到底是什么，这样才方便我们去实现它。

* React 是用于构建用户界面的 JavaScript 库

* React 不是一个完整的 MVC 框架，最多可以认为是 MVC 中的 V（view）

所以我们实现 React 的时候，只需要关注 React 如何处理 UI 就可以了，也就是这三个 api

* createElement

* render

* component

### createElement

* createElement 函数帮助我们生成 vnode（虚拟函数）

	```js
	// react.js
	function createElement(type, props, ...children) {
		if (props) {
			detele props.__source,
			delete props.__self
		}
		
		return {
			type: type,
			props: {
				...props,
				children: children.map(child => {
					typeof child === 'object' ? child : createTextNode(child);
 				});
			}
		}
	}
	
	function createTextNode(text) {
		return {
			type: "TEXT",
			props: {
				children: [],
				nodeValue: text
			}
		}
	}
	
	export default {
		createElement
	}
	```
	
### render
	
* 我们再使用 render 函数将 vnode 转换为 node

	```js
	// react-dom.js
	function render (vnode, container) {
		const node = createNode(vnode);

		container.appendChild(node);
	}

	function createNode(vnode) {
		const {type, props} = vnode;
		let node;

		if (typeof type === 'function') {
			node = type.isReactComponent ? updateClassComponent(vnode) : updateFunctionComponent(vnode)
		} else if (type === 'TEXT') {
			node = document.createTextNode("");
		} else if (type) {
			node = document.createElement(type);
		} else {
			node = document.createDocumentFragment();
		}
		updateNode(node, props);
		reconcilerChildren(props.children, node);

		return node;
	}

	function updateNode(node, props) {
		Object.keys(props)
			.filter(k => k !== 'children')
			.forEach(k => {
				if (k.slice(0,2) === 'on') {
					let eventName = k.slice(2).toLocaleLowserCase();
					node.addEventListener(eventName, props[K]);
				} else {
					node[k] = props[k];
				}
			});
	}

	function reconcilerChildren(children, node) {
		for (let i =0; i < children.length, i++) {
			let child = children[i];

			if (Array.isArray(child)) {
				for (let i =0; i < child.length, i++){
					render(child[i], node);
				}
			} else {
				render(child, node);
			}
		}
	}

	export default {
		render
	}
	```

* 需要对 type 类型进行判断，根据不同的 type 创建不同的元素，比较特殊的是函数组件和 Class 组件

	```js
	function updateClassComponent(vnode) {
		const {type, props} = vnode;
		const cmp = new type(props);
		const vvnode = cmp.render();
		const node = createNode(vvnode);

		return node;
	}

	function updateFunctionComponent(vnode) {
		const {type, props} = vnode;
		const vvnode = type(props);
		const node = createNode(vvnode);

		return node;
	}
	```
	
### component

* 上面用到的 isReactComponent 属性就是在 Component 中定义的

	```js
	export default class Component {
		static isReactComponent  = {};
		constructor(props) {
			this.props = props;
		}
	}
	```

* 其实，如果用上面的方式实现一个时钟，我们只能每次都删除原来的所有节点并重新创建

	```js
	const rootDom = document.getElementById("root");

	function tick() {
		const time = new Date().toLocaleTimeString();
		const clockElement = <h1>{time}</h1>;
		render(clockElement, rootDom);
	}

	tick();
	setInterval(tick, 1000);
	```

	很明显的，这个在性能上是不可接受的，所以我们需要 diff 算法，这个不在本文讨论。


### 总结

1. babel 将 jsx 转换为 cleateElement
2. cleateElement 函数得到 vnode
3. render 函数将 vnode 转换为 node





