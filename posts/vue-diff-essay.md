---
title: Vue diff Essay
date: 2020-03-18 11:00:14
tags: [Vue,Essay]
published: true
hideInList: false
feature: 
---
### 概要

在之前实现的 Simple Vue 里，我们在模版的 compile 依赖收集时将每个依赖都创建了一个watcher（Vue 1.0），这样是没法撑住大应用的，所以在 2.0 里面每一个组件创建一个 watcher，但问题是这种粒度下我们无法判断组件中那些元素进行了更改，**这时候，我们就需要虚拟DOM了**。

那么有了虚拟DOM后，我们如何进行高效的 DOM 更新呢，**这时候 diff 算法就出现了**。

为了方便我们随时清楚自己在diff算法的哪个流程，我们附上下图

![](https://sfmonkey.github.io//post-images/1584501228011.jpeg)

我们来根据这张图，逐步理解 diff 算法

### patch

* patch 主要接收两个参数， oldVnode 和 Vnode。**因为有较多情况存在直接创建新树的情况，一般也可以理解为 patch 进行的是树级别的比较。**

	```js
	function patch() {
		if (isUndef(oldVnode)) {
				// empty mount (likely as component), create new root element
				isInitialPatch = true
				createElm(vnode, insertedVnodeQueue)
		} else {
			if (sameVnode(oldVnode, vnode)) {
				patchVnode(oldVnode, vnode)
			} else {
				...
				// create new node
				createElm(
					vnode,
					nodeOps.nextSibling(oldElm)
				)
			}
		}
		
		...
		removeVnodes([oldVnode], 0, 0)
	}
	```

	1. 如果 oldVnode 不存在，则直接根据 vnode 创建 DOM
	2. 如果 oldVnode 与 vnode 不属于 sameVnode，这直接替换,属于 replace 操作
	3. 如果属于 sameVnode，那么开始进行节点比较 patchVnode

### sameVnode

* sameVnode 用来判断是否属于相同的节点，其实如果 key，tag，isComment，isDef(data) ，sameInputType相同，就是相同的节点。可以进行 patchVnode 了。

	```js
	functon sameVnode(a，b) {
			return (
			a.key === b.key && (
				(
					a.tag === b.tag &&
					a.isComment === b.isComment &&
					isDef(a.data) === isDef(b.data) &&
					sameInputType(a, b)
				)
				...
		)
	}
	```

	1. 	key：key值
	2. 	tag：标签名
	3. 	isComment 是否为注释节点
	4. 	isDef(data)：是否定义了 data
	5. 	sameInputType：当标签是 input 的时候，type 必须相同

### patchVnode

* patchVnode 用于进行两个相同节点的比较,然后进行相关DOM操作，操作的类型有三种

	* 属性更新
		* 当 data 有定义时，需要进行属性的更新
	* 文本更新
		* 当新老节点都无子节点的时候，只是文本的替换
	* 子节点更新
		* 如果老节点没有子节点而新节点有子节点，先清空DOM的文本内容，然后为 DOM 新增子节点
		* 如果新节点没有子节点而老节点有子节点，则移除DOM的所有子节点（在外层已经判断新节点没有文本了）
		* 如果新老节点都存在 children 子节点，则调用 updateChildren，对子节点进行 diff 操作
	
	```js
	function patchVnode(odlVnode， vnode) {
		...
	
	const oldCh = oldVnode.children
	const ch = vnode.children
	
	if (isDef(data) && isPatchable(vnode)) {
		for (i = 0; i < cbs.update.length; ++i) cbs.update[i](oldVnode, vnode)
		if (isDef(i = data.hook) && isDef(i = i.update)) i(oldVnode, vnode)
	}
	
	if (isUndef(vnode.text)) {
		if (isDef(oldCh) && isDef(ch)) {
			if (oldCh !== ch) updateChildren(elm, oldCh, ch, insertedVnodeQueue, removeOnly)
		} else if (isDef(ch)) {
			if (isDef(oldVnode.text)) nodeOps.setTextContent(elm, '')
			addVnodes(elm, null, ch, 0, ch.length - 1, insertedVnodeQueue)
		} else if (isDef(oldCh)) {
			removeVnodes(oldCh, 0, oldCh.length - 1)
		} else if (isDef(oldVnode.text)) {
			nodeOps.setTextContent(elm, '')
		}
	} else if (oldVnode.text !== vnode.text) {
		nodeOps.setTextContent(elm, vnode.text)
	}
		...
	}
	```

### updateChildren

* updateChildren 用于判断新老节点的子节点

	```js
	while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
		if (isUndef(oldStartVnode)) {
			oldStartVnode = oldCh[++oldStartIdx] // Vnode has been moved left
		} else if (isUndef(oldEndVnode)) {
			oldEndVnode = oldCh[--oldEndIdx]
		} else if (sameVnode(oldStartVnode, newStartVnode)) {
			patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
			oldStartVnode = oldCh[++oldStartIdx]
			newStartVnode = newCh[++newStartIdx]
		} else if (sameVnode(oldEndVnode, newEndVnode)) {
			patchVnode(oldEndVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx)
			oldEndVnode = oldCh[--oldEndIdx]
			newEndVnode = newCh[--newEndIdx]
		} else if (sameVnode(oldStartVnode, newEndVnode)) { // Vnode moved right
			patchVnode(oldStartVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx)
			canMove && nodeOps.insertBefore(parentElm, oldStartVnode.elm, nodeOps.nextSibling(oldEndVnode.elm))
			oldStartVnode = oldCh[++oldStartIdx]
			newEndVnode = newCh[--newEndIdx]
		} else if (sameVnode(oldEndVnode, newStartVnode)) { // Vnode moved left
			patchVnode(oldEndVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
			canMove && nodeOps.insertBefore(parentElm, oldEndVnode.elm, oldStartVnode.elm)
			oldEndVnode = oldCh[--oldEndIdx]
			newStartVnode = newCh[++newStartIdx]
		} else {
			if (isUndef(oldKeyToIdx)) oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx)
			idxInOld = isDef(newStartVnode.key)
				? oldKeyToIdx[newStartVnode.key]
				: findIdxInOld(newStartVnode, oldCh, oldStartIdx, oldEndIdx)
			if (isUndef(idxInOld)) { // New element
				createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx)
			} else {
				vnodeToMove = oldCh[idxInOld]
				if (sameVnode(vnodeToMove, newStartVnode)) {
					patchVnode(vnodeToMove, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
					oldCh[idxInOld] = undefined
					canMove && nodeOps.insertBefore(parentElm, vnodeToMove.elm, oldStartVnode.elm)
				} else {
					// same key but different element. treat as new element
					createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx)
				}
			}
			newStartVnode = newCh[++newStartIdx]
		}
	}
	```

	这里的逻辑稍微有点复杂，在 oldCh 和 ch 中，提供四个指针

	![](https://sfmonkey.github.io//post-images/1584511735002.jpeg)

* oldCh 和 ch 指针两两进行 sameVnode 比较，共分四种情况
	* oldS 和 S 匹配，那么两者进行 patchVnode，匹配上的两个指针向中间移动
	* oldE 和 E 匹配，那么两者进行 patchVnode，匹配上的两个指针向中间移动
	* oldS 和 E 匹配，那么两者进行 patchVnode，匹配上的两个指针向中间移动，真实 DOM 中的第一个节点会移动到最后
	* oldE 和 S 匹配，那么两者进行 patchVnode，匹配上的两个指针向中间移动，真实 DOM 中的最后一个节点会移动到最前

* 如果四种匹配都没有成功，则又分为两种情况

	* 如果新旧节点都存在key，那么会根据 oldCh 的key 生成一张 hash表，用S的key与 hash匹配，如果匹配成功就判断 S 和匹配节点是否是 sameNode，如果是，就进行 patchVnode，将 DOM 放在 oldS 前面，然后移动 S，否则剩下情况就创建新的 DOM，插入 oldS 的位置，然后 S 指针向中间移动。
	* 如果没有key，则直接创建新DOM 插入 oldS 位置

* 当然，在最后，结束条件即 oldS > oldE 或者 S > E 后

	```js
	if (oldStartIdx > oldEndIdx) {
		refElm = isUndef(newCh[newEndIdx + 1]) ? null : newCh[newEndIdx + 1].elm
		addVnodes(parentElm, refElm, newCh, newStartIdx, newEndIdx, insertedVnodeQueue)
	} else if (newStartIdx > newEndIdx) {
		removeVnodes(oldCh, oldStartIdx, oldEndIdx)
	}
	```
	
	* 如果 oldStartIdx > oldEndIdx ，那么将 S 与 E 及两者之间的进行 DOM 新增
	* 如果 newStartIdx > newEndIdx ，那么将 oldS 与 oldE 及两者之间的进行 DOM 删除

### 参考资料
[详解vue的diff算法](https://juejin.im/post/5affd01551882542c83301da#heading-6)