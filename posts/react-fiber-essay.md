---
title: React Fiber Essay
date: 2020-03-20 16:47:23
tags: [React,Essay]
published: true
hideInList: false
feature: /post-images/react-fiber-essay.jpg
---
### 概要

当我们了解 stck reconciler（react 16.0 之前的 diff 算法）后，我们知道 React 在大型项目中， setState 处理更新时，会因为虚拟DOM 进行 diff 而**一鼓作气**长时间占用主线程，去比较该组件及其子组件，导致一些高级别的任务，比如用户输入和动画执行出现延迟卡顿，而 Fiber 在 react 中的引入，就是为了解决这个问题。

**如果是100米短跑，或者1000米竞赛，当然越快越好。如果是马拉松，就需要考虑到保存体力了，需要注意休息了。性能是一个系统性的工程。**


既然我们要去和 fiber 做朋友了，那我们就需要去尝试了解一下它，它是怎么做到能够解决上面的问题的

* requestIdleCallback

* vdom 树形 转换到 Fiber 链式（Reconciliation Phase）

* 🚢新的 diff 算法（Reconciliation Phase）

* Fiber 转换为真实 DOM（Commit Phase）

### requestIdleCallback

**requestIdleCallback 是在浏览器空闲时会执行对应 callback 的 API**

页面一帧一帧绘制出来的，当每秒绘制的帧数（FPS）达到 60 时，页面是流畅的，小于这个值时，用户会感觉到卡顿。

1s 60帧，所以每一帧分到的时间是 1000/60 ≈ 16 ms。所以我们书写代码时力求不让一帧的工作量超过 16ms。

![](https://sfmonkey.github.io//post-images/1584699602216.jpeg)

这是浏览器一帧内的工作

上面六个步骤完成后没超过 16 ms，说明时间有富余，此时就会执行 requestIdleCallback 里注册的任务。

### Fiber

* 因为 createElement 返回的结果是 vnode ，我们需要将 vnode 树转换为 fiber 链表。因为链表的结构天然适合我们进行任务的暂停。

	```js
	//ReactDom.js

	// 下一个单元任务
	// let nextUnitOfWork = null;
	// work in progress 工作中的 fiber root
	let wrokInProgressRoot = null;
	// 现在的根节点
	let currentRoot = null;

	function render(vnode, container) {
		wrokInProgressRoot = {
			node: container,
			props: {children: [vnode]},
			base: currentRoot
		}

		nextUnitOfWork = wrokInProgressRoot;
	}

	function reconcilerChilren(workInProgressFiber, children) {
		let prevSibling = null;
		let oldFiber = workInProgressFiber.base && workInProgressFiber.base.child;
		for(let i = 0; i < children.length; i ++) {
			let child = children[i];
			let newFiber = null;

			newFiber = {
				type: child.type,
				props: child.props,
				node: null,
				base: null,
				parent: workInProgressFiber,
				effectTag: PLACEMENT
			}

			if (oldFiber) {
				oldFiber = oldFiber.sibling
			}

			if (i === 0) {
				workInProgressFiber.child = newFiber;
			} else {
				prevSibling.sibiling = newFiber;
			}

			prevSibling = newFiber;
		}
	}

	function updateClassComponent(fiber) {
		const {type, props} = fiber;
		const cmp = new type(props);
		const children = [cmp.render()];
		reconcilerChilren(fiber, children);
	}

	function updateFunctionComponent(fiber) {
		const {type, props} = fiber;
		const children = [type(props)];
		reconcilerChilren(fiber, children);
	}

	function updateHostComponent(fiber) {
		if (!fiber.node) {
			fiber.node = createNode(fiber);
		}

		const children = fiber.props;
		reconcilerChilren(fiber, children);
	}

	function preformUnitOfWork (fiber) {
		const {type} = fiber;
	
		if (typeof type === 'function') {
			type.isReactComponent ? 
				updateClassComponent(fiber) : 
				updateFunctionComponent(fiber)
		} else {
			updateHostComponent(fiber);
		}

		if (fiber.child) {
			return fiber.child;
		}

		let nextFiber = fiber;
		while(nextFiber) {
			if (nextFiber.sibling) {
				return nextFiber.sibling
			}
			nextFiber = nextFiber.parent;
		}
	}

	function workLoop(deadline) {
		while(nextUnitOfWork && deadline.timeRemaining() > 1) {
			nextUnitOfWork = preformUnitOfWork(nextUnitOfWork);
		}

		if (!nextUnitOfWork && wrokInProgressRoot) {
			commitRoot();
		}
	}

	requestIdleCallback(workLoop)
	```

* Fiber diff算法
	待更新
	
* 然后，我们需要在 Commit Phase 阶段，将 Fiber 转换为真实 DOM，在 Reconcilition Phase 阶段我们可以暂停去执行其他高优先级任务，但是在 Commit 阶段，这个更新是一气呵成的，因为这是用户要看到的 UI 阶段。

	```js
	function commitRoot() {
		commitWorker(wrokInProgressRoot.child);
		currentRoot = wrokInProgressRoot;
		wrokInProgressRoot = null;
	}

	function commitWorker(fiber) {
		if(!fiber) {
			return
		}

		let parentNodeFiber = fiber.parent;
		while (!parentNodeFiber.node) {
			parentNodeFiber = parentNodeFiber.parent;
		}

		const parentNode = parentNodeFiber.node;
		// 更新 删除 新增
		if (fiber.effectTag === PLACEMENT && fiber.node !== null) {
			parentNode.appendChild(fiber.node);
		}
		commitWorker(fiber.child);
		commitWorker(fiber.sibling);
	}
	```

### 参考资料

* [利用好浏览器的空闲时间 --- requestIdleCallback](https://www.cnblogs.com/Wayou/p/requestIdleCallback.html)
* [React Fiber](https://juejin.im/post/5ab7b3a2f265da2378403e57)
* [完全理解React Fiber](http://www.ayqy.net/blog/dive-into-react-fiber/)
* [React Fiber架构](https://zhuanlan.zhihu.com/p/37095662)





