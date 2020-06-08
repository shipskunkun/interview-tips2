
## 4-2 单线程

单线程 -  单线程就是同时只做一件事，两段 JS 不能同时执行  DOM渲染和JS执行不能同时进行。	
原因 - 避免 DOM 渲染的冲突  解决方案 - 异步 
浏览器需要渲染 DOM  JS 可以修改 DOM 结构  JS 执行的时候，浏览器 DOM 渲染会暂停  两段 JS 也不能同时执行（都修改 DOM 就冲突了）   
webworker 支持多线程，但是不能访问 DOM  ##4-7 异步

同步代码，直接执行  
异步函数先放在 异步队列中  待同步函数执行完毕，轮询执行异步队列的函数  ## 4-8 jQuery
js解决异步的方案：
	
	jQuery deferred
	promise
	async\await
	generator
异步队列
	
	什么是异步队列，何时被放入异步队列


## 4-10 ~ 4-16 介绍ajax defered


Query 1.5 的变化：   
jQuery 1.5 之后，支持 deferred   
慢慢演化成一个标准，这个标准叫promise

![img](https://github.com/shipskunkun/interview-tips2/blob/master/images/1.png)


开放封闭原则：

	对扩展开放，对修改封闭
	
	
## 4-17 promise 链式操作


	function loadImg(src) {
	    var promise = new Promise(function (resolve, reject) {
	        var img = document.createElement('img')
	        img.onload = function () {
	            resolve(img)
	        }
	        img.onerror = function () {
	            reject('图片加载失败')
	        }
	        img.src = src
	    })
	    return promise
	}
	
	// 只要是一个 promise 对象，就可以链式调用
	
	var result = loadImg(src);
	result.then(function(){
		
	}, function(){
		
	}).then(function(){
		
	})






## 4-18 捕获异常


```
// 统一捕获异常
// then 只接受一个参数，最后统一用 catch 捕获异常

result.then(function(){
	
}).then(function(){

}).catch(function(ex) {
	// 最后统一 catch
	console.log(ex)
})
```

## 4-20 多个串联

>  问题，我希望，串联，第一个返回的 callback 中，再发第二个请求，有先后顺序, 请求间有相互顺序，如何操作？


串联：

A ——> B 

B 写在A 的then 中

###注意，链式操作 和 串联操作不一样！上次面试就gg


```
var src1 = 'https://www.imooc.com/static/img/index/logo_new.png'
var result1 = loadImg(src1)

var src2 = 'https://img1.mukewang.com/545862fe00017c2602200220-100-100.jpg'
var result2 = loadImg(src2)

result1.then(function (img1) {
    console.log('第一个图片加载完成', img1.width)
    
    return result2  // 重要！！！
    
}).then(function (img2) {
    console.log('第二个图片加载完成', img2.width)
}).catch(function (ex) {
    console.log(ex)
})
```


## 4-21 promise all race


Promise.al 接受一个promise对象的数组
待全部完成之后，统一执行success


注意：

+ 接受的参数，都是 promise 数组
+ promise.all 返回的是datas 是数组，依次包含了多个promise 返回的内容
+ promise.race 最先执行完成的 promise 返回值


```
var src1 = 'https://www.imooc.com/static/img/index/logo_new.png'
var result1 = loadImg(src1)

var src2 = 'https://img1.mukewang.com/545862fe00017c2602200220-100-100.jpg'
var result2 = loadImg(src2)

Promise.all([result1, result2]).then(function (datas) {

    console.log('all', datas[0])
    console.log('all', datas[1])
})

	// 接收一个包含多个promise 对象的数组
	// 只要有一个完成，就执行success
Promise.race([result1, result2]).then(function (data) {

    // data 为最先执行完成的promise 的返回值
    console.log('race', data)
})
```

## 4-22 总结

> 状态变化

 + 三种状态：pending fulfilled rejected
  + 初始状态是 pending
  
 + pending 变为 fulfilled ，或者 pending 变为 rejected
  
 +  状态变化不可逆

> then

	 Promise 实例必须实现 then 这个方法	 then() 必须可以接收两个函数作为参数	 then() 返回的必须是一个 Promise 实例


如果 then 中没有返回 一个新的promise, 比如 return promise2 , 第二个 then 还是 result, 是本身的实例。

	var result = loadImg(src);
	result.then(function(){
		
	}, function(){
		
	}).then(function(){
		
	})



## 4-24 async-await
	
	
	async/await 是最直接的同步写法	
作用： 同步执行代码
await 后面是一个 promise 对象

	
	const load = async function() {
		const result1 = await loadImg(src1)
		const result2 = await loadImg(src2)
	}

作用：异步，改写成同步的写法

> 结果：

和 promise 结合， 但是没有和promise 冲突   
使用了promise, 但是没有用 同步回调函数，完全是同步的写法						
改变不了js单线程，异步的本质


> 语法：

 + 使用 await ，函数必须用 async 标识
 + await 后面跟的是一个 Promise 实例
 + 需要 babel-polyfill



