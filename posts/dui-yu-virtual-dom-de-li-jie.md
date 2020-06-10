---
title: 对于 Virtual DOM 的理解
date: 2019-07-04 17:13:04
tags: []
published: true
hideInList: false
feature: 
---
## 定义

**虚拟 DOM 相当于是真实 DOM 和 js 之间的一层缓存，它可以帮助我们进行最小化的 DOM 操作，来保证视图更新的执行效率。**

虚拟 DOM 算法主要分三部分

* 用 JS 对象模拟 DOM树
* 比较两棵虚拟 DOM 树的差异
* 将差异应用到真正的 DOM 树上

之前我们在[《高性能 javascript》总结](https://sfmonkey.github.io/post/2018-12-18-js-highPreformance/) 中提到过，js 引擎与 DOM 分属于两处不同的地方，在这中间的交互操作是十分消耗性能的，所以我们必须最小化 DOM 访问次数。Virtual DOM 正是帮助我们做到了这一点，Vritual DOM 并不是加快了 DOM 操作的速度，而是让页面在不同状态之间变化时，使用次数尽量少的 DOM 操作来完成。

也由于包含这对比前后差异的过程，React 每次的 DOM 操作实际耗时是一定比我们执行原生 DOM 操作要长的。也就是说 React 是付出了额外的计算过程，来尽可能保持 DOM 操作的高效。

## 算法实现

以下的代码来自于 [livoras](https://github.com/livoras/blog/issues/13) 的实现。

### 用 JS 对象模拟 DOM 树

```javascript
	/**
 * 生成虚拟 DOM 函数
 * @param {String} tagName 
 * @param {Object} props 
 * @param {Array<Element|String>} children 
 */
function Element (tagName, props, children) {
    if (!(this instanceof Element)) {   //创建 Element 对象
        if (! (children instanceof Array) && children != null) {
            children = Array.prototype.slice.call(arguments, 2).filter(function(v){return !!v}); // 处理 children 不是数组的情况
        }
        return new Element(tagName, props, children);
    }


    if (props instanceof Array) {
        children = props;
        props = {};  
    }

    this.tagName = tagName;
    this.props = props || {};   //props 可能为空
    this.children = children || []; //children 可能为空

    this.key = props.key ? props.key : void 0; //获取元素 key , 用于之后 diff 比较时找到对应节点

    var count = 0;  //记录子元素个数

    children.forEach(function(child, i) {
        if (child instanceof Element) {
            count += child.count;
        } else {
            children[i] = '' + child;   //如果不是 Element 元素，保证 child 为字符串
        }

        count++;
    });

    this.count = count;
}

/**
 * 将虚拟 DOM 渲染为真实 DOM
 */
Element.prototype.render = function () {
    var el = document.createElement(this.tagName),  // 生成 DOM 节点
        props = this.props,
        children = this.children;

    for (var k in props) {
        if (props.hasOwnProperty(k)) {
            var v = props[k];
            el.setAttribute(k,v);   // 添加节点
        }
    }

    children.forEach(function(child, i) {
        var v = '';
        if (child instanceof Element) {
            v = child.render();
        } else {
            v = document.createTextNode(child);
        }
        el.appendChild(v);  // 添加子节点
    });

    return el;
}
```

上面的函数实现了虚拟DOM的生成，那么这个函数怎么用呢？举个例子

```javascript
var el = require('./element')

var ul = el('ul', {id: 'list'}, [
  el('li', {class: 'item'}, ['Item 1']),
  el('li', {class: 'item'}, ['Item 2']),
  el('li', {class: 'item'}, ['Item 3'])
])
```

我们可以将里面的参数看成 JSX， 这样是不是就有些眼熟了，那么如何将生成的虚拟 DOM 树渲染成真正的 DOM 呢

```javascript
var ulRoot = ul.render()
document.body.appendChild(ulRoot)
```

### 比较两棵虚拟 DOM 树的差异 （diff 算法）

两棵树完全的 diff 算法的时间复杂度为 O(n^3) ，显然无法满足前端快速的 UI 响应，所以这里 React 改变了 diff 算法，只对同一层的DOM元素进行比较，将时间复杂度降为了 O(n)

我们再来想想两个虚拟DOM树可能的差异类型：

1. 替换之前的节点
2. 移动，删除，新增子节点
3. 改变了节点的属性
4. 文本节点的内容发生了变化

以下是实现

```javascript
function diff (oldTree, newTree) {
    var index = 0;
    var patches = {};

    dfsWalk(oldTree, newTree, index, patches);

    return patches;
}

function dfsWalk (oldNode, newNode, index, patches) {
    var currentPatch = [];

    if (newNode === null)  {

    } else if (Object.prototype.toString.call(oldNode) && Object.prototype.toString.call(newNode)) {    // 比较文本节点的内容
        if (newNode !== oldNode) {
            currentPatch.push({type: patches.TEXT, content: newNode});
        }
    } else if (
        oldNode.tagName === newNode.tagName &&
        oldNode.key === newNode.key
    ) {
        //  比较属性的不同
        var propsPatches = diffProps(oldNode, newNode);
        if (propsPatches) {
            currentPatch.push({type: patches.PROPS, props: propsPatches});
        }

        diffChildren(
            oldNode.children,
            newNode.children,
            index,
            patches,
            currentPatch
        );

    } else {    // tagName 不同的情况
        currentPatch.push({type: patches.REPLACE, node: newNode});
    }

    if (currentPatch.length) {
        patches[index] = currentPatch;
    }
}

function diffProps (oldNode, newNode) {
    var oldProps = oldNode.props;
    var newProps = newNode.props;

    var key, value;
    var propsPatches = {};

    for (key in oldProps) {
        if (oldProps.hasOwnProperty(key)) {
            value = oldProps[key];
            if (newProps[key] !== value) {
                propsPatches[key] = newProps[key];
            }
        }
    }

    for (key in newProps) {
        value = newProps[key]
        if (!oldProps.hasOwnProperty(key)) {
            propsPatches[key] = newProps[key];
        }
    }

    if (JSON.stringify(propsPatches) === '{}') {
        return null;
    }
    
    return propsPatches;
}

function diffChildren (oldChildren, newChildren, index, patches, currentPatch) {
    var diffs = listDiff();
    newChildren = diffs.children;

    if (diffs.moves.length) {
        var reorderPatch = {type: patches.REORDER, moves: diffs.moves}
        currentPatch.push(reorderPatch);
    }

    var leftNode = null;
    var currentNodeIndex = index;

    oldChildren.forEach(function(child, i) {
        var newChild = newChildren[i]
        currentNodeIndex = (leftNode && leftNode.count)
          ? currentNodeIndex + leftNode.count + 1
          : currentNodeIndex + 1
        dfsWalk(child, newChild, currentNodeIndex, patches)
        leftNode = child
    });
}
```

这里我们可以看出，其实每个新老节点在对比的时候，都是对比tagname，props和children，而在对比 children 时， 刚开始使用了 listDiff 算法，具体算法如下：

```javascript
function listDiff (oldList, newList, key) {
    var oldMap = makeKeyIndexAndFree(oldList, key);
    var newMap = makeKeyIndexAndFree(newList, key);

    var newFree = newMap.free;

    var oldKeyIndex = oldMap.keyIndex;
    var newKeyIndex = newMap.keyIndex;

    var moves = [];

    var children = [];
    var i = 0;
    var item, itemKey;
    var freeIndex = 0;

    while (i < oldList.length) {
        item = oldList[i];
        itemKey = getItemKey(item, key); // 获取 key 的值, 它也是 newKeyIndex 的属性
        if (itemKey) { //如果包含key
            if (!newKeyIndex.hasOwnProperty(itemKey)) { // 新的列表里面没有老的列表元素,删除老的列表元素
                children.push(null)
            }  else {   // 新的列表里面有老的列表元素，又有key，移动和新增老的列表元素
                var newItemIndex = newKeyIndex[itemKey];
                children.push(newList[newItemIndex]);
            }
        } else {    //没有key的情况
            var freeItem = newFree[freeIndex++];
            children.push(freeItem || null);
        }
        i++;
    }

    var simulateList = children.slice(0);

    // 删除新列表中不存在的元素,数组中剩下的是 1.新列表需要移动和新增的元素 2.新列表没有key的元素？？
    i = 0;
    while (i < simulateList.length) {
        if (simulateList[i] === null) {
            remove(i);
            removeSimulate(i);
        } else {
            i ++;
        }
    }

    var j = i = 0;
    while (i < newList.length) {
        item = newList[i];
        itemKey = getItemKey(item, key);

        var simulateItem = simulateList[i];
        var simulateItemKey = getItemKey(simulateItem, key);

        if (simulateItem) {
            if (itemKey === simulateItemKey) {
                j++;
            } else {
                if (!oldKeyIndex.hasOwnProperty(itemKey)) {
                    // 新增
                    insert(i, item);
                } else {
                    // 移动
                    var nextItemKey = getItemKey(simulateList[j + 1], key);
                    if (nextItemKey === itemKey) {
                        remove(i);
                        removeSimulate(j);
                        j ++;
                    } else {
                        insert(i, item);
                    }
                }
            }
        } else {
            insert(i, item);
        }

        i++;
    }

    var k = simulateList.length - j;
    while (j++ < simulateList.length) {
        k --;
        remove(k + i);
    }

    function remove (index) {
        var move = {index: index, type: 0}
        moves.push(move);
    }
    
    function insert (index, item) {
        var move = {index: index, item: item, type: 1};
        moves.push(move);
    }

    function removeSimulate (index) {
        simulateList.splice(index, 1);
    }
}

function makeKeyIndexAndFree (list, key) {
    var keyIndex = {};
    var free = [];

    for (var i = 0, len = list.length; i < len; i++) {
        var item = list[i];
        var itemKey = getItemKey(item, key);
        if (itemKey) {
            keyIndex[itemKey] = i;
        } else {
            free.push(item);
        }
    }
}

/**
 * 获取数组元素key的值
 * @param {*} item 
 * @param {String} key 
 */
function getItemKey (item, key) {
    if (!item || !key) return void 0;
    return typeof key === 'string'
        ? item[key]
        : key(item)
}
```

其实，我们可以看出，listDiff 只是帮助我们记录了 children 列表元素应该移动，删除和新增的位置，具体移动后的元素，tagName ， props 如何变化，我们是在深度优先遍历的递归中做的，也就是到了下一个节点。

### 把差异应用到真正的DOM树上

现在我们已经得到了虚拟DOM树，以及新旧两个虚拟 DOM 树的差异，那边接下来我们就将 patches 应用到 DOM 树上。

```
var REPLACE = 0;
var REORDER = 1;
var PROPS = 2;
var TEXT = 3;

function patch (node, patches) {
    var walker = {index: 0};
    dfsWalk(node, walker, patches);
}

function dfsWalk (node, walker, patches) {
    var currentPatches = patches[walker.index];

    var len = node.childNodes ? node.childNodes.length : 0;

    for (var i = 0; i < len; i++) {
        var child = node.childNodes[i];
        walker.index ++;
        dfsWalk(child, walker, patches);
    }

    if (currentPatches) {
        applyPatches(node, currentPatches)
    }
}

function applyPatches (node, currentPatches) {
    currentPatches.forEach(currentPatch => {
        switch (currentPatch.type) {
            case REPLACE:
              var newNode = (typeof currentPatch.node === 'string')
                ? document.createTextNode(currentPatch.node)
                : currentPatch.node.render()
              node.parentNode.replaceChild(newNode, node)
              break
            case REORDER:
              reorderChildren(node, currentPatch.moves)
              break
            case PROPS:
              setProps(node, currentPatch.props)
              break
            case TEXT:
              if (node.textContent) {
                node.textContent = currentPatch.content
              } else {
                // fuck ie
                node.nodeValue = currentPatch.content
              }
              break
            default:
              throw new Error('Unknown patch type ' + currentPatch.type)
          }
    });
}

function setProps (node, props) {
    for (var key in props) {
      if (props[key] === void 666) {
        node.removeAttribute(key)
      } else {
        var value = props[key]
        _.setAttr(node, key, value)
      }
    }
}

function reorderChildren (node, moves) {
    var staticNodeList = _.toArray(node.childNodes)
    var maps = {}
    
    staticNodeList.forEach(node => {
        if (node.nodeType === 1) {
            var key = node.getAttribute('key')
            if (key) {
              maps[key] = node
            }
          }
    });
    
    moves.forEach(function(move) {
        var index = move.index
        if (move.type === 0) { // remove item
          if (staticNodeList[index] === node.childNodes[index]) { // maybe have been removed for inserting
            node.removeChild(node.childNodes[index])
          }
          staticNodeList.splice(index, 1)
        } else if (move.type === 1) { // insert item
          var insertNode = maps[move.item.key]
            ? maps[move.item.key].cloneNode(true) // reuse old item
            : (typeof move.item === 'object')
                ? move.item.render()
                : document.createTextNode(move.item)
          staticNodeList.splice(index, 0, insertNode)
          node.insertBefore(insertNode, node.childNodes[index] || null)
        }
    });
}
```

当然，以上算法开始于我们已经有了一个 tagname， props， children 的入参， 我们避开了 JSX，这也不是虚拟 DOM 算法应该关注的，还有什么时候去触发 update 函数，来进行 虚拟DOM 算法，这些无关主题的，也就都省略了。

**猛兽总是独行**

参考资料：
[深度剖析：如何实现一个 Virtual DOM 算法](https://github.com/livoras/blog/issues/13)
[React的diffing算法学习](https://imweb.io/topic/5b67e47cf3fbd8d9125fe800)