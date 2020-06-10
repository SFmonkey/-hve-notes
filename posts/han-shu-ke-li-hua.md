---
title: 函数柯里化
date: 2019-06-17 18:21:43
tags: []
published: true
hideInList: false
feature: /post-images/han-shu-ke-li-hua.jpg
---
记得之前找实习的时候，面试官总喜欢问我： **遇到过的闭包的使用场景** ，那时候年轻啊，现在想想面试官希望我回答的，可能就是**偏函数**和**柯里化**吧。

### 闭包
> 闭包（closure）是函数和声明该函数的词法环境的组合。

### 偏函数
> 固定了你函数的某一个或几个参数，返回一个新的函数，接收剩下的参数, 参数个数可能是1个，也可能是2个，甚至更多,这就是偏函数。**偏函数相当于执行一次的柯里化**。

```javascript
add = (x, y, z) => x + y + z
addBySeven = Partial(Add, 7);
addBySeven(5, 10); // returns 22;
```

### 柯里化
> 只使用传递给函数的一部分参数来调用它，让它返回一个函数去处理剩下的参数。

```javascript
add = (x, y, z) => x + y + z
curryAdd = Curry(add);    //Curry = x => y => z => x + y + z
addBySeven = curryAdd(7);
addBySeven(5)(10); // returns 22
```

### 当然我们今天的主菜是柯里化

* 实际应用
	* 延迟计算
	```javascript
	const add = (...args) => args.reduce((a, b) => a + b)
	
	const currying = (func) => {
		const args = [];
		return function result (...rests) {
			if (rests.length === 0) {
				return func(...args);
			} else {
				args.push(...rests);
				return result;
			}
		}
	}
	
	const sum = currying(add);

	sum(1,2)(3); // 未真正求值
	sum(4); 		 // 未真正求值
	sum(); 			 // 输出 10
	```
	
	```javascript
	// 实现 bind 函数
	Function.prototype.bind = function (context) {
		var _ = this,
					args = Array.prototype.slice.call(arguments, 1);
		
		return () => {
				var bindArgs = Array.prototype.slice.call(arguments);
				
				return _.apply(context, args.concat(bindArgs));
		}
	}
	```
	
	* 动态创建函数
	```javascript
	const addEvent =(function () {
		if (window.addEventListener) {
			return function (type, el, fn, capture = false) {
				el.addEventListener(type, fn, capture);
			}
		} else if (window.attachEvent) {
			return function (type, el, fn) {
				el.attachEvent('on' + type, fn);
			}
		}
	})()
	```
	
	* 参数复用
	```javascfript
	const toStr = Function.prototype.call.bind(Object.prototype.toString);
	
	// 改造后
	toStr([1, 2, 3]); 	// "[object Array]"
	toStr('123'); 		// "[object String]"
	toStr(123); 		// "[object Number]"
	toStr(Object(123)); // "[object Number]"
	```

* 实现 currying 函数
```javascript
const currying = (fn, length) => {
	const len = length || fn.length;
	
	return function result(...args) {
		if (args.length >= len) {
			return fn.apply(this, ...args);
		} else {
			return currying(fn.bind(this, ...args), len - args.length)
		}
	}
}
const fn = currying(function(a, b, c) {
		console.log([a, b, c]);
});
fn("a", "b", "c") // ["a", "b", "c"]
fn("a", "b")("c") // ["a", "b", "c"]
fn("a")("b")("c") // ["a", "b", "c"]
fn("a")("b", "c") // ["a", "b", "c"]
```

#### 总结
当然，从上面的三种应用场景可以看出，函数柯里化其实是一种工具方法，它将函数改造为更适合我们的模式，按照Stoyan Stefanov --《JavaScript Pattern》作者 的说法，所谓“柯里化”,  `就是使函数理解并处理部分应用` 。从闭包引入柯里化函数的概念，是因为根据概念 ： `只使用传递给函数的一部分参数来调用它，让它返回一个函数去处理剩下的参数`，所以柯里化函数是必然包含闭包的。

#### 以上

#### 献出心脏！！

参考资料：
[柯里化和偏函数有什么区别?](https://segmentfault.com/q/1010000008626058)
[深入高阶函数应用之柯里化](https://github.com/yygmind/blog/issues/37)
[JavaScript 函数式编程技巧 - 柯里化](https://mp.weixin.qq.com/s/bnhMvxz3mVsuOx0QRrnL-w)






