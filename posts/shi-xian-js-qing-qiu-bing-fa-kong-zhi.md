---
title: 实现 js 请求并发控制
date: 2019-05-13 11:34:23
tags: []
published: true
hideInList: false
feature: 
---
```javascript
const requestPool = (urls, num, callback) => {
    const urlsPool = [...urls];
    let pendPool = [];

    const refresh = () => {
        if (urlsPool.length) {
            const url = urlsPool.pop();
            requestHandle(url);
        } else {
            callback();
        }
    }

    const requestHandle = async (url) => {
        const key = Symbol(url);

        pendPool.push(key);
        const data = await fetch(url);
        pendPool.filter((v) => {
            return v !== key
        });
        refresh();

        return data;
    }

    const limit = Math.min(num, urlsPool.length);

    for (let i = 0; i < limit; i++) {
        refresh();
    }
}
```