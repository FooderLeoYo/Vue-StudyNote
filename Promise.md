# Promise

## 目录

[什么是Promise](#jump1)

[resolve方法](#jump2)
                      
[reject方法](#jump3)

[then方法](#jump4)

[catch方法](#jump5)

[then带1个参数和2个参数的示例](#jump6)

[链式调用](#jump7)

[all方法](#jump8)

---

<span id="jump1"></span>
## 什么是Promise

- Promise是异步编程的一种解决方案

- Promise构造函数包含一个参数，它是一个带有resolve方法和reject方法两个参数的回调函数

- 自己身上有resolve、reject、all、race这几个方法，原型上有then、catch等方法

- 大致的使用方法是将需异步处理的内容放入Promise中，再根据处理是成功或失败分别调用then或catch

- Promise一旦被new出来，其中的代码便会自己执行，而不是像函数内的代码需要调用该函数才执行
	
	- 因为Promise本质是一个存放异步处理内容的容器

	- 可以理解为只是代码段外面被多包了一层，但不影响其执行

---

<span id="jump2"></span>
## resolve方法

promise成功时会回调resolve，这时做了三件事情

	1. 调用then方法

	2. 将参数传递给then方法，这个参数可能是：

	3. 把promise的状态修改为fullfiled

---
	
<span id="jump3"></span>
## reject方法

promise失败的时会回调reject，做了三件事：

	1. 调用catch（then接收1个参数时）方法或then方法（then接收2个参数时）

	2. 传递错误对象给catch（then接收1个参数时）或then（then接收2个参数时）

	3. 把promise的状态修改为rejected

---

<span id="jump4"></span>
## then方法

- Promise.prototype.then方法返回的是一个新的Promise对象

	- 可以理解为.then()把括号里的东西包在一个新的Promise里

	- 该Promise对象的状态为resolved或rejected

- then方法可以分为接收一个参数或两个参数的两种写法：

	- 1个参数

		then只处理promise成功时传递过来的参数，即promise成功时回调resolve后传递给then一个成功后的参数

	- 2个参数

		- then同时处理promise成功或失败时传递过来的参数

		- 这两个参数分别是两个函数，第一个对应resolve的回调，第二个对应reject的回调
		
		- 也就是说then方法中接受两个回调，一个成功的回调函数，一个失败的回调函数，并且能在回调函数中拿到成功的数据和失败的原因
	
---

<span id="jump5"></span>
## catch方法

- Promise.prototype.catch 方法是 Promise.prototype.then(null, rejection) 的别名，用于指定发生错误时的回调函数

- Promise 对象的错误具有"冒泡"性质，会一直向后传递，直到被捕获为止
	
	- 即把catch放在最后，前面若干个回调的错误也能被处理

	- 可以理解为if判断，有错时就不走then而是一路向下直到最后的catch

	- 链式调用的最后一定要有一个catch（中间有几个catch要看具体需求），不然catch后的then发生错误时无法捕获

```javascript
getJSON("/post/1.json").then(function (post) {
  return getJSON(post.commentURL);
}).then(function (comments) {
  // some code
}).catch(function (error) {
  // 处理前两个回调函数的错误
});

```

---
	
<span id="jump6"></span>
## then带1个参数和2个参数的示例

- then只处理promise成功时传递过来的参数

```javascript
new Promise((resolve, reject) => {
  setTimeout(() => {
	if (成功条件) {
	  resolve('成功')
	}
	else {
	  reject('失败')
	}
  }, 1000)
}).then(res => { 
  console.log(res);
}).catch(err => {
  console.log(err);
})
```

- then同时处理promise成功或失败时传递过来的参数

省略catch，then中带两个函数

```
new Promise((resolve, reject) => {
  setTimeout(() => {
  	if (成功条件) {
	  resolve('Hello Vuejs')
	}
	else {
	  reject('error message')
	}
  }, 1000)
}).then(res => { 
  console.log(res);
}, err => { 
  console.log(err);
})
```

---

<span id="jump7"></span>
## 链式调用

then 方法返回的是一个新的 Promise 对象，因此可以采用链式写法

```javascript
const p = new Promise((resolve, reject) => {
  resolve(1);
}).then((value) => {
  console.log(value); // 打印结果为1
  return value * 2;
}).then((value) => {
  console.log(value); // 打印结果为2
}).then((value) => {
  console.log(value); // 打印结果为undefined
  return Promise.resolve('resolve');
}).then((value) => {
  console.log(value); // 打印结果为resolve
  return Promise.reject('reject');
}).then((value) => {
  console.log('resolve:' + value); // 不会执行打印
}, (err) => {
  console.log('reject:' + err); // 打印结果为reject:reject
});
```

注意点：

	1. resolve只用于第一次，then中不能用resolve；第一次必须有哪怕是不传参：resolve()，否则无法调用then

	2. 除了最后一个then，其他then最后必须return，否则下一个then拿不到数据，res为undefined

---

<span id="jump8"></span>
## all方法

应用场景：当有两个以上异步请求，且要求这几个请求都完成才执行下一步

```javascript
Promise.all([ // Promise的all方法
  new Promise((resolve, reject) => { // 异步请求1
    setTimeout(() => {
      resolve({ name: 'why', age: 18 })
    }, 2000)
  }),
  new Promise((resolve, reject) => { // 异步请求2
    setTimeout(() => {
      resolve({ name: 'kobe', age: 19 })
    }, 1000)
  })
]).then(results => { // 两个异步请求都完成的话才会执行then
  console.log(results);
})
```

---
	
<span id="jump9"></span>
## race方法

- all是等所有的异步操作都执行完了再执行then方法，那么race方法就是相反的

- 谁先执行完成就先执行回调。先执行完的不管是进行了race的成功回调还是失败回调，其余的将不会再进入race的任何回调
