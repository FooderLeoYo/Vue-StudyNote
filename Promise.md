# Promise

## 目录

[什么是Promise](#jump1)

[resolve方法](#jump2)
                      
[reject方法](#jump3)

[then方法](#jump4)

[catch方法](#jump5)

[链式调用](#jump7)

[all方法](#jump8)

[race方法](#jump9)

---

<span id="jump1"></span>
## 什么是Promise

- Promise是异步编程的一种解决方案

- Promise构造函数包含一个参数（必须），它是一个带有resolve方法和reject方法两个参数（必须）的回调函数

- 自己身上有resolve、reject、all、race这几个方法，原型上有then、catch等方法

- 大致的使用方法是将需异步处理的内容放入Promise中，再根据处理是成功或失败分别调用then或catch

- Promise一旦被new出来，其中的代码便会自己执行，而不是像函数内的代码需要调用该函数才执行
	
	- 因为Promise本质是一个存放异步处理内容的容器

	- 可以理解为只是代码段外面被多包了一层，但不影响其执行

---

<span id="jump2"></span>
## resolve方法

- Promise.resolve(value)方法返回一个以给定值解析后的Promise 对象

- promise成功时会回调resolve(value)，这时做了三件事情

	1. 调用then方法

	2. 将value传递给then方法，这个value可能是：
		
		- value是一个普通值，then括号中的函数的参数就是这个值

		```javascript
		let p1 = new Promise(...)
		let p2 = new Promise(...)

		p1.then(res => {
		  // then括号里的函数就是then生成的新Promise的resolve
		  return ('p2')
		}).then(res => { 
		  // 这时res是return的值
		  console.log(res) // 打印结果为p2（字符串）
		}).catch(() => console.log('发生错误'))
		```

		- value是一个 promise ，那么这个promise将作为下一个then的调用者

		```javascript
		let p1 = new Promise(...)
		let p2 = new Promise(...)

		p1.then(res => {
		  return p2
		}).then(res => {
		  // 这时res不是p2，而是p2的resolve的value
		  console.log(res) // 打印结果为p2的resolve的value
		}).catch(() => console.log('发生错误'))
		```

		- 空：resolve()，仅起到调用then的作用，无法将任何值传递给then（用的很少）

		- value是thenable的，即带有"then" 方法（用得很少）

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

	- then带2个参数（非必须），均为函数。第一个函数就是新Promise的resolve，第二个函数就是reject

	- 该Promise对象的状态为resolved或rejected

- then方法可以分为接收一个参数或两个参数的两种写法：

	- 1个参数

		then只处理promise成功，失败由catch处理

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

	- 2个参数

		then同时处理promise成功或失败时传递过来的参数

		```javascript
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

1. resolve只用于第一次；then中不能直接写resolve，除非把resolve包在一个新Promise中

	- 因为then返回一个新的Promise，then括号中的第一个函数其实新Promise的resolve

	- 如果then中调用resolve，相当于resolve(resolve)，显然不对

2. 第一次必须调resolve，哪怕是不传参：resolve()，否则往下无法调用then

3. 除了最后一个then，其他then最后必须return，否则下一个then拿不到数据，res为undefined

4. 最后一定要有一个错误处理（catch或then带第二个参数），否则会遗漏错误捕获

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
