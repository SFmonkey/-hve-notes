---
title: React diff Essay
date: 2020-03-19 14:19:05
tags: [React,Essay]
published: true
hideInList: false
feature: 
---
### 概要
我们都知道，react 中是通过 setState 来触发组件 render 函数，得到虚拟 DOM，然后 oldVnode 与 vnode 进行 diff ，最后得到高效的 DOM 更新操作。

在 React diff 中，更新策略有三点

1. Web UI 中 DOM 节点跨层级的移动操作特别少，可以忽略不计
2. 拥有相同类的两个组件将会生成相似的树形结构，拥有不同类的两个组件将会生成不同的树形结构
3. 对于同一层级的一组子节点，他们可以通过唯一 id 进行区分

基于这三个更新策略，我们可以得出 React diff 中三种不同的 diff 类型

1. tree diff
2. component diff
3. element diff

### tree diff

React 对🌲的算法进行了简洁明了的优化，即对树进行分层比较，两棵树只会对同一层次的节点进行比较。

* 首先我们来控制同层比较

	```js
	_updateChildren: function(      
		nextNestedChildrenElements,
		transaction,
		context,
	) {
		...
		var nextChildren = this._reconcilerUpdateChildren(
			prevChildren,
			nextNestedChildrenElements,
			mountImages,
			removedNodes,
			transaction,
			context,
		);
	}
	```

	prevChildren 和 nextChildren 就是对应的虚拟 DOM，nextNestedChildrenElements 这玩意儿是个指针，来标明当前树的层次，得到当前 prevChildren 的层次。

* 然后 ReactChildReconciler.updateChildren 会将 prevChildren、nextChildren封装成ReactDOMComponent类型，并进行后续比较和操作。

所以当出现节点跨层级移动时，并不会出现想象中的移动操作，而是以 A 为根节点的树被整个重新创建，这是一种影响 React 性能的操作，因此 **React 官方建议不压迫进行 DOM 节点跨层级的操作**

![](https://sfmonkey.github.io//post-images/1584599830004.jpeg)

### component diff

我们需要清楚的是，createElement 生成的虚拟 DOM，如果节点本身是组件，那么虚拟 DOM 中的 tag 其实就是对应的 class 或者 function，这个在之前我们实现 react 的时候就已经有体现了。当然这是 react 虚拟 DOM，在 vue 中 tag 不太一样，这是后话。

* 如果是同一类型的组件，按照原策略继续比较虚拟 DOM 树

* 如果不是，则将该组件判断为 dirty component，从而替换整个组件下的所有子节点

* 对于同一类型的组件，有可能其 Virtual DOM 没有任何变化，这是我们可以通过 shouldComponentUpdate 钩子来告诉该组件是否进行 diff，从而提高大量的性能。

	```
	// 超简单代码实现
	const compareTwoVnodes(oldVnode, newVnode, dom) {
			let newNode = dom

			// 如果新的虚拟DOM是null，那么就将前一次的真实DOM移除掉
			if (newVnode == null) {
					destroyVnode(oldVnode, dom)
					dom.parentNode.removeChild(dom)

			} else if (oldVnode.type !== newVnode.type || oldVnode.key !== newVnode.key) {
					// replace
					destroyVnode(oldVnode, dom)
					newNode = initVnode(newVnode, parentContext, dom.namespaceURI)
					dom.parentNode.replaceChild(newNode, dom)

			} else if (oldVnode !== newVnode || parentContext) {
					// same type and same key -> update
					newNode = updateVNode(oldVnode, newVnode, dom, parentContext)
			}
	}
	/** 
	* 更新虚拟DOM
	* 这里的type需要注意一下，如果vnode是个html元素，例如h1，那么type就是'h1'
	** 如果vnode是一个函数组件，例如const Header = () => <h1>header</h1>，那么type就是函数Header
	** 如果vnode是一个class组件，那么type就是那个class
	*/
	const updateVNode = (vnode, node) => {
			const { type } = vnode; // type是指虚拟DOM的类型
			// 如果是class组件
			if (type === VCOMPONENT) {
					return updateComponent(vnode, node)
			} else (type === VSTATELESS){
					return updateStateLess(vnode, node)
			}
			updateVChildren(vnode, node)
	}
	// 更新class组件（调用render方法拿到新的虚拟DOM）
	const updateComponent = (vnode, node) => {
			const { type: Component } = vnode; // type是指虚拟DOM的类型
			const newVNode = new Component().render();
			compareTwoVnodes(newVNode, vnode, node);
	}
	// 更新无状态组件（直接执行函数拿到新的虚拟DOM）
	const updateStateLess = (vnode, node) => {
			const { type: Component } = vnode; // type是指虚拟DOM的类型
			const newVNode = Component();
			compareTwoVnodes(newVNode, vnode, node);
	}
	const updateVChildren = (vnode, node) => {
			for (let i = 0; i < node.children.length; i++) {
					updateVNode(vnode.children[i], node.children[i])
			}
	}
	```

### element diff 

当节点处于同一层级时，React diff 提供了三种节点操作

```js
// oldDoms是真实DOM，newDoms是最新的虚拟DOM
const oldDoms = [A, B, C, D],
	newDoms = [B, A, E, D],
	updates = [],
	removes = [],
	creates = [];
// 进行两层遍历，获取到哪些节点需要更新，哪些节点需要移除。
for (let i = 0; i < oldDoms.length; i++) {
	const oldDom = oldDoms[i]
	let shouldRemove = true
	for (let j = 0; j < newDoms.length; j++) {
		const newDom = newDoms[j];
		if (
				oldDom.key === newDom.key &&
				oldDom.type === newDom.type
		) {
				updates[j] = {
						index: j,
						node: oldDom,
						parentNode: parentNode // 这里真实DOM的父节点
				}
				shouldRemove = false
		}
	}
	if (shouldRemove) {
			removes.push({
					node: oldDom
			})
	}
}
// 从虚拟DOM节点来取出不要更新的节点，这就是需要新创建的节点。
for (let j = 0; j < newDoms.length; j++) {
	if (!updates[j]) {
			creates.push({
					index: j,
					vnode: newDoms[j],
					parentNode: parentNode // 这里真实DOM的父节点
			})
	}
}
```

* 插入

	```js
	const create = (creates) => {
			creates.forEach(create => {
					const index = create.index,
							parentNode = create.parentNode,
							vnode = create.vnode,
							curNode = parentNode.children[index],
							node = createNode(vnode); // 创建DOM节点
					parentNode.insertBefore(node, curNode)
			})
	}
	```

* 移动

	```js
	const update = (updates) => {
			updates.forEach(update => {
					const index = update.index,
							parentNode = update.parentNode,
							node = update.node,
							curNode = parentNode.children[index];
					if (curNode !== node) {
							parentNode.insertBefore(node, curNode)
					}
			})
	}
	```

* 删除

	```js
	const remove = (removes) => {
			removes.forEach(remove => {
					const node = remove.node
					node.parentNode.removeChild(node)
			})
	}
	```

### 后续

* 看到这里，我们首先能引出一个思考🤔，那就是 vue 与 React diff 算法的不同
	
	* react 有 compnent diff，而 vue 没有
	
		> 	在 React 应用中，当某个组件的状态发生变化时，它会以该组件为根，重新渲染整个组件子树。
		> 
		> 	如要避免不必要的子组件的重渲染，你需要在所有可能的地方使用 PureComponent，或是手动实现 shouldComponentUpdate 方法。同时你可能会需要使用不可变的数据结构来使得你的组件更容易被优化。
		> 
		> 	然而，使用 PureComponent 和 shouldComponentUpdate 时，需要保证该组件的整个子树的渲染输出都是由该组件的 props 所决定的。如果不符合这个情况，那么此类优化就会导致难以察觉的渲染结果不一致。这使得 React 中的组件优化伴随着相当的心智负担。
		> 
		> 	在 Vue 应用中，组件的依赖是在渲染过程中自动追踪的，所以系统能精确知晓哪个组件确实需要被重渲染。你可以理解为每一个组件都已经自动获得了 shouldComponentUpdate，并且没有上述的子树问题限制。
		> 
		> 	Vue 的这个特点使得开发者不再需要考虑此类优化，从而能够更好地专注于应用本身。

		所以说，React 之所以会有 component 的比较，而 vue 在虚拟DOM 为 component 时会跳过 diff ，就是这个原因
	
	* 剩下的 tree diff 与 element diff 两者都有，虽然具体实现不同，但都是有的

* 其次，我们想这么一种场景🎬，因为 React setState 会更新其组件以及它的所有子组件，这些组件会全部进行 diff 算法对比，那么在存在大量子组件的情况下，这种算法无疑是十分耗费性能的，所以，这也就是 React 团队会耗时两年的时间，将 fiber 引入 React 的原因。

### 参考资料

[react diff简单实现](https://zhuanlan.zhihu.com/p/63964441)

[[译] React性能优化：Virtual Dom原理浅析](https://zhuanlan.zhihu.com/p/36798520)

[React 源码剖析系列 － 不可思议的 react diff](https://zhuanlan.zhihu.com/p/20346379)

[协调](https://zh-hans.reactjs.org/docs/reconciliation.html)

[React源码之Diff算法](https://segmentfault.com/a/1190000010686582)