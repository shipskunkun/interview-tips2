## 5-1 virtual dom


vdom  80%依赖diff算法

<b>vdom是什么?</b>

	在前端中，对dom直接修改操作是昂贵的，效率是低下的，
	我们用js对象模拟dom结构，把dom变化的对比,放在js层来做，提高重绘性能
	
	尝试自己写一个vdom的例子

dom:  

![img](https://github.com/shipskunkun/interview-tips2/blob/master/images/2.png)



vdom:


![img](https://github.com/shipskunkun/interview-tips2/blob/master/images/3.png)


## 5-4 jQuery是如何实现 render的

例如表格，每次重新添加 html, HTML 和 data 混合在一起，效率低下

> steps:

1. 清空表格所在的位置  
2. 给表格添加头 
3. 遍历数据，添加每一行每一列
4. container 添加表格

![img](https://github.com/shipskunkun/interview-tips2/blob/master/images/4.png)






## 5-7~ 5-11 snabbdom

随着 React Vue 等框架的流行，Virtual DOM 也越来越火，snabbdom 是其中一种实现，而且 Vue 2.x 版本的 Virtual DOM 部分也是基于 snabbdom 进行修改的。snabbdom 这个库核心代码只有 200 多行，非常适合想要深入了解 Virtual DOM 实现的读者阅读。如果您没听说过 snabbdom，可以先看看官方文档。



diff算法步骤

	1. 用js对象来描述dom树结构，然后用这个js对象来创建一棵真正的dom树，插入到文档中
	2. 当状态更新时,将新的js对象和旧的js对象进行比较，得到两个对象之间的差异
	3. 将差异应用到真正的 dom 上

	
snabbdom 主要的两个函数：

1. h(type, data, children)，返回 vnode 节点，模拟的 dom 节点

		h(‘<标签名>’, {…属性…}, […子元素…])
		h(‘<标签名>’, {…属性…}, ‘….’)  
2.  patch(oldVnode, newVnode)，比较新旧 Virtual DOM 树并更新。

		patch(container, vnode) 
		patch(vnode, newVnode) 

   

![img](https://github.com/shipskunkun/interview-tips2/blob/master/images/5.png)


## 5-8 snabbdom

snabbdom 是如何实现， vnode 渲染的？
初次渲染和  更新渲染 ？

####自己的思路：
	
	1，使用插件的 h 函数和 patch 函数
	2， 找到 插入node的节点，patch
	3, 监听改变dom 动作，生成新的 vnode ， patch

```
 <script src="https://cdn.bootcss.com/snabbdom/0.7.0/h.js"></script>
 <script type="text/javascript">
 
        var snabbdom = window.snabbdom
        
        // 定义关键函数 patch
        var patch = snabbdom.init([
            snabbdom_class,
            snabbdom_props,
            snabbdom_style,
            snabbdom_eventlisteners
        ])

        // 定义关键函数 h
        var h = snabbdom.h


        var container = document.getElementById('container')

        // 渲染函数
        var vnode
        function render(data) {
	        var newVnode = h(data);  //根据data 生成 vnode
	       
	        if (vnode) {  //如果有旧节点，更新
	            // re-render
	            patch(vnode, newVnode)
	        } else {
	            // 初次渲染
	            patch(container, newVnode)
	        }
	
	        // 存储当前的 vnode 结果
	        vnode = newVnode
	    }

        // 初次渲染
        render(data)


        var btnChange = document.getElementById('btn-change')
        btnChange.addEventListener('click', function () {
            data[1].age = 30
            data[2].address = '深圳'
            // re-render
            render(data)
        })

```

## 5-12 ~ 5-14 废话，介绍diff 算法, 略

<hr>

## 自己的思路

> 自己整理的思路？  

	1. 核心就两个，根据js对象模拟的 vdom 生成 vnode  
		如果节点是空，插入		
		否则，更新之前的 vnode	
	
	2. 更新vnode, 如何以代价最小方式更新？如何判断	

1. 首先，我们介绍	
patch(container, vnode)		 
如何把 json 描述的 虚拟对象，生成真实的  DOM 呢？
	
		通过createElement(vnode) 函数
		根据tag 类型，创建节点，
		遍历节点的属性 attrs ，给节点添加属性
		遍历节点的孩子节点，通过appendChild, 递归调用createElement，生成dom， 添加到节点上


2. 如何更新节点？
	
		遍历vnode 的孩子节点，遍历。  
		如果  oldVnode 孩子i.tag = newNode 孩子i.tag, 递归比较更新  他们各自的孩子，下一层的事情
		如果 不相等，直接 用新节点 newVode[i]，替换 oldVnode[i]


<hr>

## 5-15 ~ 5-21  diff算法 实现

> 为何要使用diff算法？

	1.  DOM 操作是“昂贵”的，因此尽量减少 DOM 操作	2.  找出本次 DOM 必须更新的节点来更新，其他的不更新	3.  这个“找出”的过程，就需要 diff 算法
	


> diff 实现过程	1.  patch(container, vnode)
	2.  patch(vnode, newVnode)
	3.  createElment
	4.  updateChildren
> h 函数的具体实现

```
function createElement(vnode) {
    var tag = vnode.tag  // 'ul'
    var attrs = vnode.attrs || {}
    var children = vnode.children || []
    if (!tag) {
        return null
    }

    // 创建真实的 DOM 元素
    var elem = document.createElement(tag)
    // 属性
    var attrName
    for (attrName in attrs) {
        if (attrs.hasOwnProperty(attrName)) {
            // 给 elem 添加属性
            elem.setAttribute(attrName, attrs[attrName])
        }
    }
    // 子元素
    children.forEach(function (childVnode) {
        // 给 elem 添加子元素
        elem.appendChild(createElement(childVnode))  // 递归
    })

    // 返回真实的 DOM 元素
    return elem
}


```


patch 函数的具体实现


```
function updateChildren(vnode, newVnode) {
    var children = vnode.children || []
    var newChildren = newVnode.children || []

    children.forEach(function (childVnode, index) {
        var newChildVnode = newChildren[index]
        if (childVnode.tag === newChildVnode.tag) {
            // 深层次对比，递归
            updateChildren(childVnode, newChildVnode)
        } else {
            // 替换
            replaceNode(childVnode, newChildVnode)
        }
    })
}

function replaceNode(vnode, newVnode) {
    var elem = vnode.elem  // 真实的 DOM 节点
    var newElem = createElement(newVnode)

    // 替换
}

```


```

// 渲染函数
var vnode
function render(data) {
    var newVnode = h('table', {}, data.map(function (item) {
        var tds = []
        var i
        for (i in item) {
            if (item.hasOwnProperty(i)) {
                tds.push(h('td', {}, item[i] + ''))
            }
        }
        return h('tr', {}, tds)
    }))

    if (vnode) {
        // re-render
        patch(vnode, newVnode)
    } else {
        // 初次渲染
        patch(container, newVnode)
    }

    // 存储当前的 vnode 结果
    vnode = newVnode
}
```





## diff 总结

vdom 中应用 diff 算法是为了找出需要更新的节点  

实现，patch(container, vnode) 和 patch(vnode, newVnode)

核心逻辑，createElement 和 updateChildren


## 参考文章

https://juejin.im/post/5b9200865188255c672e8cfd

## review  DOM 

```
// 设置属性
element.setAttribute(attributename,attributevalue)

getElementById
getElementsByTagName
getElementsByClassName

appendChild
removeChild
replaceChild

insertBefore()
	element.insertBefore(被插dom, 想要在哪个dom前插入)
createElement
createTextNode()

getAttribute()
setAttribute()

```










