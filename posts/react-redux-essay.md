---
title: React-redux Essay
date: 2020-03-10 11:58:34
tags: [React,Essay]
published: true
hideInList: false
feature: /post-images/react-redux-essay.jpg
---
### 概要

React-redux 是 React 的状态应用，可以保证数据的一致性且易于追溯和测试。关于 React-redux，前置知识有：

* context
* redux

接着才是主菜

* react-redux

### Context

这里的 Context 指的是 React 的上下文，属于 React 的高级部分，虽然官方认为一个稳定的 React 应用不应该使用 Context，但是许多的优秀的 React 库都使用了 Context，比如 React-router 以及 React-redux。

Context 的 API 有三个 (hooks 相关的这里不做讨论)

* CreateContext （可以理解为种菜的）

	```js
	// 参数可以接收初始值
	const UserContext = React.CreateContext({name: "potter"});
	```

* Provider （可以理解为收菜的供货商）

	```js
	const UserProvider = UserContext.Provider;
	```

	使用方式：

	```jsx
	render() {
		return <UserProvider></UserProvider>
	}
	```

* Cosumer （可以理解为消费者）

	```js
	const UserCosumer = UserContext.Cosumer;
	```

	使用方式：

	```jsx
	render() {
		return <UserCosumer>{
			context => {}
		}</UserCosumer>
	}
	```

	这里接收 Context 还有另外一种方式，即在 Class 组件中

	```js
	// 这种接收方式只能获取单一 Context，通过 this.context 获取传递内容
	Class User extend Component {
		static contextType = UserContext
	}
	```
	
### Redux

Redux 是 Js 应用的状态容器。
	
* 使用方式

	redux 在使用时，大概分为以下几步：

	* reducer 纯函数制定修改规则
	* createStore 创建 store
	* store.getState 得到数据
	* store.dispatch 更改数据
	* store.subscribe 订阅数据改动

* 实现

	* redux 的实现主要考虑的是 createStore ， 返回的 store 中包含剩下的 getState，dispath 与 subscribe。

	```js
	const createStore = (reducer, enhancer) => {
		
		if(enhancer) { // 这里是判断是否有 redux 中间件，如果有，将 createStore 与 reducer 传入
			return enhancer(createStore)(reducer)
		}
		
		let currentState;
		let currentListener = [];
		
		function getStore() {
			return currentState;
		}
		
		function  dispatch(action) {
			const currentState = reducer(currentState, action);
			currentListener.forEach( v=> v());
		}
		
		function subscribe(v) {
			currentListener.push(v);
		}
		
		// 处理初始值的情况
		dispatch({type: '@@INIT'});
		
		return {
			getState,
			dispatch,
			subscribe
		}
	}
	```
	
	* redux 只是一个纯粹的状态管理器，默认只支持同步，我们需要通过中间件来支持异步。所有的中间件都是一个函数，通过对 dispatch 的改造，在 action 与 reducer 之间，添加了其他的功能。

		这里我们需要做两件事：
		
		* createStore 增加 enhancer 参数，增加 applyMiddleware 函数

		```js
		const applyMiddleware = (...middlewares) => {
			return createStore => （...args）=> {
				const store = createStore(...args);
				const middleApi = {
					getState: store.getState,
					dispatch: store.dispatch
				};
				const middlewaresChain = middlewares.map(middleware => middleware(middleApi))
				
				let dispatch = compose(middlewaresChain)(store.dispatch);
				
				return {
					...store,
					dispatch
				}
			}
		}
		```
		
		这里面还涉及到一个 compose 函数
		
		```js
		const compose = (...funs) => {
			if (funs.length === 0) {
				return arg => arg;
			}
			if (funs.length === 1) {
				return funs[0];
			}
			
			return funs.reduce((a,b) => (...args) => a(b(...args)));
		}
		```

		* 外部其他库实现 redux-thunk ， redux-logger 等中间件

		```js
		// redux-thunk
		function thunk({dispatch, getState}) {
			return dispatch  => action => {
				if (typeof action === "function") {
					return action(dispatch, getState);
				} else {
					return dispatch(action)
				}
			}
		}
		
		// redux-logger
		function logger() {
			return dispath => action => {
				console.log(action.type + '执行了');
				return dispatch(action);
			}
		}
		```
		
### react-redux

* 出现原因

	Redux 和 React 一起用的时候有两个不太完美的地方

	* store 每个组件都需要引入
	* 每次都需要调用订阅函数，在里面触发 render

  React-redux 帮助我们解决了这两个痛点，提供了两个 API

	* Provider 为后代组件提供 store
	* connect 为组件提供数据和变更方法

* 使用

	* Provider

	```js
	//一般放在根组件外层，方便后代组件调用
	ReactDOM.render(
	<Provider store={store}>
		<App/>
	</Provider>,
	document.getElementById('root')
	);
	```
	
	* connect

	```js
	export default connect(
		// mapStateToProps Function (state, ownProps)
		// ! ownProps 需要谨慎使用，如果它发生变化，mapStateToProps 就会执行，里面的 state 会被重新计算
		// ! 容易影响性能
		state => ({num: state.num}),
		
		// mapDispatchToProps Object/Function/undefined 
		// ! 如果不定义，默认把 dispatch 注入组件
		// ! 如果是对象的话，原版的 dispatch 就没有被注入了
		{
			add: () => ({type: 'ADD'})
		},
		// ! 如果是Function (dispatch, ownProps) 的话，同样谨慎使用 ownProps
		(dispatch) => {
			let res = {add: ()=> ({type: 'ADD'}), minus: ()=> ({type: 'MINUS'})};
			
			// bindActionCreators 可以帮助我们用 dispatch 对 action creator 进行包装
			res = bindActionCreators(res, dispatch);
			
			return {dispatch, ...res}
		}
		
		// mergeProps Function (mapStateToProps, mapDispatchToProps, ownProps)
		// ! mapStateToProps, mapDispatchToProps 的执行结果，以及组件自身的 prop 都将作为参数传入这个函数
		(mapStateToProps,mapDispatchToProps,ownProps) => {
			return {omg:"omg",...mapStateToProps,...mapDispatchToProps,...ownProps}
		}
		
	)(<SomePage></SomePage>);
	```

*  实现

	* Provider

	```js
	class Provider extends Component {
		render () {
			return <ValueContext.Provider value={this.props.store}>{this.props.children}</ValueContext.Provider>
		}
	}
	```
	
	* connect

	```js
	const connect = (mapStateToProps = state =>state , mapDispatchToProps) => WrappedComponent => {
		return class extends Component {
			static contextType = ValueContext;
			constructor(props) {
				super(props);
				this.state = {
					props: {}
				}
			}
			
			componentDidMount() {
				this.update();
				const {subscribe} = this.context;
				subscribe(() => {
					this.update();
				});
			}
			
			update() {
				const {getState, dispatch} = this.context;
				
				const stateProps = mapStateToProps(getState());
				let dispatchProps;
				if (typeof mapDispatchToProps === 'object') {
					dispatchProps = bindActionCreators(mapDispatchToProps, dispatch);
				}else if (typeof mapDispatchToProps === 'function') {
					dispatchProps = mapDispatchToProps(dispatch, this.props);
				}else {
					dispatchProps = {dispatch};
				}
				
				this.setState({
					props: {
						...stateProps,
						...dispatchProps
					}
				});
			}
	
			return <WrappedComponent {...this.props} {...this.state.props}></WrappedComponent>
		}
	}
	```
	
	* bindActionCreators

	```js
	const bindActionCreator = (creator, dispatch) => {
		return (...args) => dispatch(creator(...args));
	}

	const bindActionCreators = (creators, dispatch) => {
		const obj = {};
		for(const key in creators){
			obj[key] = bindActionCreator(creators[key], dispatch)
		}
		return obj;
	}
	```



	
	
	
	
	