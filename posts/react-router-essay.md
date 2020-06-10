---
title: React Router Essay
date: 2020-03-08 10:51:31
tags: [React,Essay]
published: true
hideInList: false
feature: /post-images/react-router-essay.jpg
---
### 概要

关于 [react-router](http://react-router.docschina.org/) 我们最常用的 API 有：

* Router
* Link
* Route
* Switch
* Redirect

这五个 API 基本就可以帮助我们构建一个基础的前端路由系统了

### Router

* 其中 Router 分为 BrowserRouter，HashRouter 以及 MemoryRouter 三种。

	**BrowserRouter** 是官方最为推荐的一种方式，它的优点在于 URL 地址按照路径划分，较为简洁与美观，使用 HTML5 history API 来保持页面的UI与URL同步。缺点在于 BrowserRouter 需要后端的支持，因为按照路径划分，就会导致用户可能直接访问某一路径，这就需要服务端有相应的路由支持。

	**HashRouter** 的优点在于简单方便，它利用浏览器 URL Hash 来区分页面 path，不需要后端的支持。缺点在于 Hash Router 不管在 URL 的简洁性，location.key，location.state的支持，动态路由以及其他问题（hash 是没法传给后端的，其他页面拼接参数也需要注意）上，都没有 BrowserRouter 好用。

	**MemoryRouter** 将 location 的改变放在内存中，我们在 URL 上是没有看到变化的，这在测试或非浏览器环境下非常适用，比如 RN。

* 这里只记录下 BrowserRouter 的实现

	我们需要明白 BrowserRouter 需要提供的功能是什么

	* 没得说，展示 children

		```js
		return this.props.chilren
		```

	* 提供一个context，将 history，location 传递下去

		```jsx
		<RouterContext value={{history: this.history, location: this.state.location}}></RouterContext>
		```

### Link

* Link API 分为 Link，NavLink 两种

	**Link** 帮助我们提供声明式的，可访问的导航。

	**NavLink** 是 Link 的一种特殊版本，在与 URL 相匹配时，给渲染元素提供一些样式。

* 这里我们想一下 Link 的实现

	* 本质上还是 a 标签

	```js
	<a onClick={handleClick}>{this.props.children}</a>
	```
	
	* 提供跳转功能

	当然我们不能使用原生的 href 属性了，因为会导致页面的刷新，我们需要使用 HTML history API 来帮助我们实现路由跳转

		```js
		handleClick(e, history) {
			e.preventDefault();
			const {to} = this.props;
			history.push(to);
		}
		```
		
### Route

* **Route** 组件是 React Router 中最重要的组件，它的作用是在 location 与 Route 的 path 匹配时呈现一些 UI。

* 我们来想一下实现，它都提供了那些功能
	
	* path 属性，我们在内部判断 path 与 location 是否 match ，来显示对应的 UI

	```js
	const match = matchPath(location.path, {...this.props, path});
	```
	
	* Route 的 UI 呈现，一共有三种方式：children > component > render
	
		* 如果 match 到了，那么就展示 children， component ，render 或者 null
	
		* 如果没有match到，那么就展示 children 或者 null
	
		```js
		match
				? children
					? typeof children === 'function'
						? children(props)
						: children
					: component
					 ? React.createElement(component, props)
						: render
							? render(props)
							: null
				: typeof children === 'function'
					? children(props)
					: null
		```

	* Route 可能会传入一个 location 属性，这时候就会首先使用传入的，而不是 URL 的。

### Switch
		
 * **Switch** 即独占路由，渲染与 location 匹配的第一个子节点
 
 * 实现的大体思路

	* 拿到所有的子节点进行遍历，需要一个变量来记录是否已经有匹配的节点

	```js
	React.children.forEach(children, (child)=> {
		if (match === null && React.isVaildElement) {
		match = path ? matchPath(location.pathname, child.props) : context.match
	})
	return match ? React.cloneElement(child, {location, computedMatch: match}) : null;
	```
	* 如果找到了 location 与 path 相匹配的路由，就将它渲染出来，否则返回 null
	 
### Redirect

* **Redirect**将URL导航到一个新的地址

* 实现的大体思路：

	* 接受 to 属性，利用 history API 将地址压入栈内

	```js
	render() {
		const {to} = this.props;
		return <RouterContext>{
		context => {
			const {history} = context;
			reutrn <LifeCycle isMounted={(history)=>{history.push(to)}}></LifeCycle>
			}
		}</RouterContext>
	}
	```
	
	* 这里需要 LifeCycle 组件的原因是我们需要一个组件 mounted 的时机，来帮助我们完成入栈操作，并且 redirect 还可以在 render 中顺利执行 return


	 
		
