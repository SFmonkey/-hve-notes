---
title: 手写一个 promise
date: 2019-01-25 09:56:02
tags: [blog]
published: true
hideInList: false
feature: 
---
```javascript

    function Promise (executor) {

        let _this = this,
            _this.status = 'pending',
            _this.value = undefined,
            _this.reason = undefined,
            _this.onResolvedCallbacks = [],
            _this.onRejectedCallbacks = [];

        function resolve (value) {
            if (_this.status === 'pending') {
                _this.status = 'resolved';
                _this.value = value;

                _this.onResolvedCallbacks.forEach(function(fn){
                    fn();
                });
            }
        };

        function reject (reason) {
            if (_this.status === 'pending') {
                _this.status = 'reject';
                _this.value = reason;

                _this.onRejectedCallbacks.forEach(function(fn){
                    fn();
                });
            }
        };

        try {
            executor(resolve,reject);
        } catch (e) {
            reject(e);
        }
    }
    
    Promise.prototype.then = function(onFulfilled, onRejected) {

        type onFulfilled === 'function' ? onFulfilled : function(value) {
            return value;
        }

        type onRejected === 'function' ? onRejected : function(err) {
            throw err;
        }

        let _this = this;
        let promise2;

        if (_this.status = 'pending') {
            promise2 = new promise (function(resolve, reject) {
                _this.onResolvedCallbacks.push(function(){
                           
                    try {
                        let x = onFulfilled(_this.value);
                        resolve(x);
                    } catch (e) {
                        reject(e);
                    }
                });
                _this.onRejectedCallbacks.push(function(){

                    try {
                        let x = onRejected(_this.reason);
                        resolve(x);
                    } catch (e) {
                        reject(e);
                    }
                });
            }
        }

        if (_this.status = 'resolved') {

            let x = onFulfilled(_this.value);
            promise2 = new promise (function(resolve, reject) {
                try {
                    resolve(x);
                } catch (e) {
                    reject(e);
                }
            })
        }

        if (_this.status = 'rejected') {
            
            let x = onRejected(_this.reason);
            promise2 = new promise (function(resolve, reject) {
                try {
                    resolve(x);
                } catch (e) {
                    reject(e);
                }
            })
        }

        return promise2;
    }

    Promise.prototype.resolve = function (value) {
        return new Promise(function(resolve,reject){
            resolve(value);
        })
    }

    Promise.prototype.reject = function (reason) {
        return new Promise(function(resolve,reject){
            resolve(reason);
        })
    }

    Promise.prototype.race = function (promises) {
        return new Promise(function(resolve,reject){
            for ( let i = 0; i < promises.length; i ++) {
                promises[i].then(resolve,reject);
            }
        })
    }

    Promise.prototype.all = function (promises) {
        return new Promise(function(resolve,reject){

            let arr = [];
            let i = 0; 

            function processData(index, y) {
                arr[index] = y;
                if (++index === promises.length) {
                    resolve(arr);
                }
            }

            for ( let i = 0; i < promises.length; i ++) {
                promises[i].then(function(y){
                    processData(i, y)
                },reject);
            }
        })
    }

    module.exports = Promise
```

### 参考资料

[可能是目前最易理解的手写promise](https://juejin.im/post/5dc383bdf265da4d2d1f6b23)