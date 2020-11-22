## review

如何实时监听的？ 
	
> 在 vm 的 defineproperty 每个属性的 set函数， 调用 updateComponent, 生成新的 vnode 树， 然后执行 patch 函数




## 6章

## 6-6 ~ 6-8 理解mvvm

![img](https://github.com/shipskunkun/interview-tips2/blob/master/images/6.png)

![img](https://ss0.bdstatic.com/70cFuHSh_Q1YnxGkpoWK1HF6hhy/it/u=526963968,1756048168&fm=26&gp=0.jpg)


ViewModel 的理解，联系 View 和 Model  

	Model - 模型、数据  
	View - 视图、模板（视图和模型是分离的）	

三者之间的联系，以及如何对应到各段代码				
ViewModel 的理解，联系 View 和 Model
	
 	ViewModel - 连接 Model 和 View 	
		view 可以通过事件绑定 DOM Listener ，影响到 model
		method中修改 data数据，即model，比如说， this.list = [] 
		
		model 可以通过数据绑定 Data binding，影响到 view
		定义 data ,同步到视图中，比如 data 中list 多了一行，立马页面上 table 多了一行


## 6-9  vue 三要素

 + 响应式
 + 模板引擎
 + 渲染


		响应式：vue 如何监听到 data 的每个属性变化？  
		模板引擎：vue 的模板如何被解析，指令如何处理？  
		渲染：vue 的模板如何被渲染成 html ？以及渲染过程  


## 6-10 什么是响应式

问自己一个问题？

	Data binding 是如何通过，修改 model 影响 view 的？

分为两个划分，讲解。

	 修改 data 属性之后，vue 立刻监听到
	
	 data 属性被代理到 vm 上

## 6-11 defineProperty


+ 获取属性的时候，如何监听到？
+ 赋值属性的时候，如何监听到？

监听属性，访问和修改？

	var obj = {
		name: 'zhangsan',
		age: 25
	}
	console.log(obj.name)
	obj.age = 26

如何通过 defineProperty 监听到 属性 访问和 修改？
	
	var obj = {
		name: 'zhangsan'
	}
	// 简单的深拷贝
	var obj2 = JSON.parse(JSON.stringify(obj));
	
	 Object.defineProperty(obj, 'name', {
	    get: function () {
	        console.log('捕获监听') // 监听
	        //return obj['name'] //注意，不能直接访问 obj['name']，会导致递归，死循环
	        return obj2['name']
	    },
	    set: function (newVal) {
	        console.log('捕获修改') // 监听
	        obj2['name'] = newVal
	    }
	})

## 6-12 如何将data 与 vm 绑定

#### 思路：

	在上一节中，我们已经知道如何监听对象的属性，获取和修改
	那么如何监听，vm对象，每一个属性的，获取和修改？


```
var vm = {}
var data = {
    name: 'zhangsan',
    age: 20
}

var key, value
for (key in data) {
    (function (key) {
        Object.defineProperty(vm, key, {
            get: function () {
                console.log('get', data[key]) // 监听
                return data[key]
            },
            set: function (newVal) {
                console.log('set', newVal) // 监听
                data[key] = newVal
            }
        })
    })(key)
}
```

	view 的响应式是怎么回事？
关键点：

	关键是理解 Object.defineProperty
	
	将 data 的属性代理到 vm 上


![img](https://github.com/shipskunkun/interview-tips2/blob/master/images/7.png)


defineProperty 解析：

对象 某个属性的 配置，      
对象 o 拥有 name属性，可以获取值、修改值

```

Object.defineProperty(o, 'name', {
  value: 1,
  writable: true,      是否可以被重写
  configurable: true,  目标属性是否可以被删除或是否可以再次修改特性
  enumerable: true,  // 是否可以枚举 for in object.keys
  get : function(){
    return bValue;
  },
  set : function(newValue){
    bValue = newValue;
  },
});

```

## 6-14 模板引擎


1. 模板是什么  
2. render 函数  
3. render 函数与 vdom  


## 6-15  模板是什么？

> 问题

	本质?
	特点？
	与html区别？
	最后的呈现


+ 模板就是 .vue 文件写的 类似 html的代码， 本质：字符串  
+ 有逻辑，如 v-if v-for 等，能判断，能循环  
+ 与 html 格式很像，但有很大区别
+ 模板最终还要转换为 html 来显示  

#### 模板最终必须转换成 JS 代码，因为：  

+ 有逻辑（v-if v-for），必须用 JS 才能实现（图灵完备 )
+ 转换为 html 渲染页面，字符串不能，必须用 JS 才能实现
+ 因此，模板最重要转换成一个 render 函数



## 6-16 with 函数

 > with 函数的作用， 

 	通过 with 取代 对象的属性读取 . 操作


```
 var obj = {
    name: 'zhangsan',
    age: 20,
    getAddress: function () {
        alert('beijing')
    }
}

// 没有使用 with 时，需要通过 . 访问属性和方法
function fn() {
     alert(obj.name)
     alert(obj.age)
     obj.getAddress()
}
fn()

// 使用with 的时候，可以直接 访问，属性和方法，不用 对象.
function fn1() {
    with(obj) {
        alert(age)
        alert(name)
        getAddress()
    }
}
fn1() 
```

## 6-17  render 讲解1

> 思路

	模板，啥样
	转换成 模板函数 啥样？
	结合with

怎么感觉 render 函数和 h 函数差不多？知识返回的 是一个函数 _c()

this: vm 对象

```
// 模板
<div id="app">
    <p>{{price}}</p>
</div>
    
    
// 模板函数
function render() {
    with(this) {  // this 就是 vm
        return _c(
            'div',
            {
                attrs: {'id': 'app'}
            },
            [
                _c('p', [_v(_s(price))])
            ]
        )
    }
}

```


总结：
	
	 模板中所有信息都包含在了 render 函数中
	 
	 this 即 vm
	 
	 price 即 this.price 即 vm.price，即 data 中的 price
	 _c 即 this._c 即 vm._c





![img](https://github.com/shipskunkun/interview-tips2/blob/master/images/8.png)

## 6-18 render 讲解2


如何把 with(this) 改成 with(vm)?

	function render() {
	    with(this) {
	        return _c(
	        		...
	        )
	    }
	}
	
	function render1() {
	    return vm._c(
	    ...	
		)
	}

_c 函数，生成dom节点  
	
	function createElement()
	vm._c 其实就相当于 snabbdom 中的 h 函数
	render 函数执行之后，返回的是 vnode

_v ？？  

	function  createTextVnode(val)
	创建文本节点函数

_s ??  

	toString 函数
	function toString()




## 6-19 render 讲解3

> 问题？


+ 从哪里可以看到 render 函数？  
+ 复杂一点的例子，render 函数是什么样子的？  
+ v-if v-for v-on 都是怎么处理的？


> 为什么模板必须要转换成js？

	模板中有大量v-for,v-if 逻辑处理，模板必须转成html渲染，这些都必须转成js才能实现


```
this._c = function (a, b, c, d) {
  var vnode = createElement(contextVm, a, b, c, d, needNormalization);
  if (vnode) {
    vnode.fnScopeId = options._scopeId;
    vnode.fnContext = parent;
  }
  return vnode
};


this._v =  function createTextVNode (val) {
  return new VNode(undefined, undefined, undefined, String(val))
}

this._s = Object.prototype.toString;
```

## 6-20、21、22 render 讲解4、5、6



 vue2.0 开始，支持预编译，
 编译打包后，在生产环境下，就是js代码

根据 todo-list demo 的 render 函数：

v-model 是怎么实现的？

![img](https://github.com/shipskunkun/interview-tips2/blob/master/images/9.png)

	v-model 双向绑定，既能get , 也能set
	可以通过看到render函数，监听 input 函数

v-on:click 是怎么实现的？
	
	_c(
	'button',
		{
			on: {
				'click': add
			}
		}
	)

v-for 是怎么实现的？
	
	_c('div', [
		_c(
			'ul',
			_l((list), function(item){ 
				return _c('li', [_v(_s(item))])
			} )
		) 
	])



## 6-23 vm._c 返回了什么，render函数返回了什么

> 自己的总结
> 
> h 函数 = vm._c = createElement  
> patch 函数 = updateComponent



	vm._c 其实就相当于 snabbdom 中的 h 函数
	
	render 函数执行之后，返回的是 vnode 	

	updateComponent 中实现了 vdom 的 patch
	
	页面首次渲染执行 updateComponent
	
	data 中每次修改属性，执行 updateComponent

### 这个图还挺重要，加载不出来，要自己看

![img](https://github.com/shipskunkun/interview-tips2/blob/master/images/10.png)



如何实时监听的？ 
	
> 在 vm 的 defineproperty 每个属性的 set函数， 调用 updateComponent, 生成新的 vnode 树， 然后执行 patch 函数



## 6-25 整体流程，重要！可以再看一遍

第一步：解析模板成 render 函数

	打包到时候已经执行。模板编译成 render 函数。 

1. with 的用法
2. 模板中的所有信息都被 render 函数包含
3. 模板中用到的 data 中的属性，都变成了 JS 变量
4. 模板中的 v-model  v-for  v-on 都变成了 JS 逻辑
5. render 函数返回 vnode




```
	v-for、v-if、v-model
	
	with(this){  // this 就是 vm
            return _c(
                'div',
                {
                    attrs:{"id":"app"}
                },
                [
                    _c(
                        'div',
                        [
                            _c(
                                'input',
                                {
                                    directives:[
                                        {
                                            name:"model",
                                            rawName:"v-model",
                                            value:(title),
                                            expression:"title"
                                        }
                                    ],
                                    domProps:{
                                        "value":(title)
                                    },
                                    on:{
                                        "input":function($event){
                                            if($event.target.composing)return;
                                            title=$event.target.value
                                        }
                                    }
                                }
                            ),
                            _v(" "),
                            _c(
                                'button',
                                {
                                    on:{
                                        "click":add
                                    }
                                },
                                [_v("submit")]
                            )
                        ]
                    ),
                    _v(" "),
                    _c('div',
                        [
                            _c(
                                'ul',
                                _l((list),function(item){return _c('li',[_v(_s(item))])})
                            )
                        ]
                    )
                ]
            )
        }
```
第二步：响应式开始监听
	
	Object.defineProperty
	将 data 的属性代理到 vm 上

第三步：首次渲染，显示页面，且绑定依赖

主要理解，在render中访问vm的属性，如 with(this)  title 
这个时候回被get监听到

	1. 初次渲染，执行 updateComponent，执行 vm._render()
	2. 执行 render 函数，会访问到 vm.list vm.title
	3. 会被响应式的 get 方法监听到
	4. 执行 updateComponent ，会走到 vdom 的 patch 方法
	5. patch 将 vnode 渲染成 DOM ，初次渲染完成

绑定依赖：
	
	1.  data 中有很多属性，有些被用到，有些可能不被用到
	2.  被用到的会走到 get ，不被用到的不会走到 get
	3.  未走到 get 中的属性，set 的时候我们也无需关心
	4.  避免不必要的重复渲染

第四步：data 属性变化，触发 rerender
	
	修改属性，被响应式的 set 监听到
	set 中执行 updateComponent
	updateComponent 重新执行 vm._render()
	生成的 vnode 和 prevVnode ，通过 patch 进行对比
	渲染到 html 中



render 函数

![img](https://github.com/shipskunkun/interview-tips2/blob/master/images/11.png)


必须走get才能走set? 如何理解

	如果不被get监听，说明模板render函数中没有这个变量，模板中没有使用这个变量，所以不会在set中监听


## 6-28 总结

模板是什么：
	
	字符串，有逻辑，嵌入了js变量
为什么转成js代码
	
	有逻辑，渲染html，js变量



