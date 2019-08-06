# Promise对象

## 含义

1、异步：操作之间没关系，可以同时进行多个操作(数据请求)。代码复杂

2、同步：同时只能做一件事。代码简单

如果用异步的方法，会出现**回调地域**

```
ajax('/banners',function(){
    //成功
    ajax('/hotItems',function(){
        //成功
    },function(){
        //失败
    })
},function(){
    //失败
})
```

> Promise优点——消除异步操作，可以用同步的方式，来书写异步代码。
>
> Promise缺点——无法取消`Promise`,一旦新建就会立即执行，无法中途取消。 如果没有回调函数，Promise内部抛出的错误不会反映到外部。当处于`Pending`状态时，无法得知目前进展到哪一个状态。



### Promise生命周期

异步操作完成前：`Pending-----进行中`

异步操作完成后：`pending->fulfilled-----已成功`

​				      `pending—>rejected-----已失败`

我们无法以编程方式判断 `Promise` 到底处于哪种状态。不过你可以使用`then()`方法在 Promise 的状态改变时执行一些特定操作。

## 用法

### Promise实例

```javascript
const promise = new Promise(function(resolve,reject){
    
        // ... some code
        if(/*异步操作成功*/){
            resolve(value)  //pending->resolved  把异步操作的结果作为参数传递出去
        }else{
            reject(error) //pending->rejected   把异步操作报出的错误作为参数传递出去
        }
    })
```

`Promise`实例生成以后，可以用`then`方法分别指定`resolved`状态和`rejected`状态的回调函数

```javascript
promise.then(function(value) {
  // success
}, function(error) {
  // failure
});

promise.then(function(value) {  
  // success
});
promise.then(null,function(error) {  
  // failure
});
```

### Promise.prototype.then()

> `then`方法作用：为 Promise 实例添加状态改变时的回调函数。
>
> `then`方法参数：第一个参数是`resolved`状态的回调函数，第二个参数（可选）是`rejected`状态的回调函数
>
> `then`方法返回值：返回一个新的Promise实例，所以可以在then方法后再调用then方法

```javascript
getJSON("/post/1.json").then(function(post){
    return getJSON(post.commentURL)
}).then(function funcA(comments){
    console.log("resolved:",comments)
},function funcB(error){   
	console.log("rejected:",err)
})
//第一个then方法指定的回调函数，返回的是另一个Promise对象。
//这时，第二个then方法指定的回调函数，就会等待这个新的Promise对象状态发生变化。如果变为resolved，就调用funcA，如果状态变为rejected，就调用funcB。
```

### Promise.prototype.catch()

> 用于指定发生错误时的回调函数
>
> `.then(null.rejection)` 或`.then(undefined,rejection)`

**一般来说，不要在`then`方法里面定义 Reject 状态的回调函数（即`then`的第二个参数），总是使用`catch`方法。**

```
//写法一
promise.then(function(data){
    //success
},function(err){
    //error
});

//写法二
promise
.then(function(data){
    //success
})
.catch(function(err){
    //error
})


```

**Promise 对象后面要跟`catch`方法，这样可以处理 Promise 内部发生的错误。`catch`方法返回的还是一个 Promise 对象，因此后面还可以接着调用`then`方法。**

### Promise.prototype.finally()

> `finally`方法：用于指定不管 Promise 对象最后状态如何，都会执行的操作
>
> `finally`方法的回调函数不接受任何参数

```
promise
.then(result => {···})
.catch(error => {···})
.finally(() => {···});
```

### Promise.all()

> `Promise.all`方法用于将多个 Promise 实例，包装成一个新的 Promise 实例。
>
> 如果不是，就会先调用下面讲到的`Promise.resolve`方法，将参数转为 Promise 实例，再进一步处理。
>
> 参数：Promise.all[数组]
>
> `p`的状态由`p1`、`p2`、`p3`决定，分成两种情况。
>
> （1）只有`p1`、`p2`、`p3`的状态都变成`fulfilled`，`p`的状态才会变成`fulfilled`，此时`p1`、`p2`、`p3`的返回值组成一个数组，传递给`p`的回调函数。
>
> （2）只要`p1`、`p2`、`p3`之中有一个被`rejected`，`p`的状态就变成`rejected`，此时第一个被`reject`的实例的返回值，会传递给`p`的回调函数。

```
//如果作为参数的 Promise 实例，自己定义了catch方法，那么它一旦被rejected，并不会触发Promise.all()的catch方法。
const p1 = new Promise((resolve, reject) => {
  resolve('hello');
})
.then(result => result)
.catch(e => e);

const p2 = new Promise((resolve, reject) => {
  throw new Error('报错了');
})
.then(result => result)
.catch(e => e);

Promise.all([p1, p2])
.then(result => console.log(result))
.catch(e => console.log(e));
// ["hello", Error: 报错了]
```

p1有自己的catch，p2也有自己的catch。catch方法返回的也是一个新的Promise实例，p2实际上指向的是这个实例。

当p2执行完catch方法后，状态会变成resolved。

那么Promise.all()参数里面的两个实例都会resolved，所以会直接调用then方法，而不会调用catch方法。

如果`p2`没有自己的`catch`方法，就会调用`Promise.all()`的`catch`方法。

### Promise.race()

> ```javascript
> const p = Promise.race([p1, p2, p3]);
> ```
>
> Promise.race()中的参数也是一个实例数组，如果不是Promise实例，会先调用`Promise.resolve`方法，将参数转为 Promise 实例，再进一步处理。
>
> 只要`p1`、`p2`、`p3`之中有一个实例率先改变状态，`p`的状态就跟着改变。那个率先改变的 Promise 实例的返回值，就传递给`p`的回调函数。
>
> 也就是说谁先变`fulfilled`结果就是`fulfilled`,谁先变`rejected`，结果就是`rejected`

### Promise.resolve()

> `Promise.resolve`：将现有对象转为 Promise 对象。
>
> 状态为resolved

- 参数为原始值

```javascript
let p = Promise.resolve('foo')
//等价于
let p = new Promise(resolve=>resolve('foo'))

p.then(function(txt){
    console.log(txt)
})
```

- 不带参

```javascript
//Promise.resolve()方法允许调用时不带参数，直接返回一个resolved状态的 Promise 对象。
let p = Promise.resolve()
//等价于
let p = new Promise(resolve=>resolve())

p.then(()=>{
    
})
```

- 参数是具有then方法的对象

```
let thenable = {
    then:function(resolve,reject){
        resolve(42)
    }
}
//Promise.resolve方法会将这个对象转为 Promise 对象，然后就立即执行thenable对象的then方法。
let p1 = Promise.resolve(thenable)
p1.then(function(value){
    console.log(value)//42
})
```

### Promise.reject()

> `Promise.reject(reson)`返回新的Promise实例，实例状态为rejected
>
> Promise.reject()中传入的参数是什么，最后抛出的就是什么。如果说Promise.reject参数是一个thenable对象，那么catch得到的参数就是这个thenable对象。

```
let p = Promise.reject('错啦~~~')
//等同于
let p = new Promise((resolve,reject)=>reject('出错了'))

p.then(null,function(error){
    console.log(error) //‘错啦~~~’
})
```

### Generator与Promise结合

嗯...还没看Generator，看完了再填坑吧~~~

