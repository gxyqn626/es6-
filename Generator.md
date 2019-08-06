# Generator函数

> ES6 提供的一种异步编程解决方案。
>
> Generator是一个状态机，封装了多个内部状态。可以依次遍历Generator函数内部的每一个状态，是一个遍历器对象生成函数。

## 基本使用

- 调用Generator函数后，函数不执行，返回一个指向内部状态的指针对象。（*[[GeneratorStatus]]*: "suspended"）
- 调用`next()`方法，开始执行Generator函数。遇到`yield`表达式，将紧跟在`yield`后面的那个表达式的值，作为返回的对象的`value`属性值。暂停执行后面的操作。
- 下一次调用`next`方法时，再继续往下执行，直到遇到下一个`yield`表达式。
- 如果没有再遇到新的`yield`表达式，就一直运行到函数结束，直到`return`语句为止，并将`return`语句后面的表达式的值，作为返回的对象的`value`属性值。
- 如果该函数没有`return`语句，则返回的对象的`value`属性值为`undefined`。

## 与Iterator关系

>任意一个对象的`Symbol.iterator`方法，等于该对象的遍历器生成函数，调用该函数会返回该对象的一个遍历器对象

**可以把生成器函数赋给对象的Symbol.iterator属性，从而使对象有Iterator接口**

```javascript
var myIterable = {}
    console.log(myIterable)
    myIterable[Symbol.iterator] = function* (){
        yield {value:'222',done:false};
        yield 3+2;
        yield 999;
    };
    console.log(...myIterable)
```

```javascript
function* gen(){
  // some code
}

var g = gen();  //遍历器函数执行后返回遍历器对象

g[Symbol.iterator]() === g //遍历器对象的Symbol.iterator执行后返回自身
// true
```

## yield与next

看了一早上yield与next都没有弄明白，现在开始记住两句话：

- yield表达式本身返回值为undefined。
- next（参数），这个参数会被当作上一个yield表达式的返回值。
- 第一次使用Next()方法传递的参数是无效的。
- next（）方法的结果是碰到yield停止后的表达式的值 (yield（表达式）)。

先看第一个例子

```javascript
function *f(){
    for(var i = 0; true; i++){
        var reset = yield i;
        if(reset){
            i = -1;
        }
    }
}
var g = f();  //生成器对象，还没有开始执行
g.next() //开始执行 执行到yield i 。 yield后表达式i的值为0 所以 {value:0,done:false}
g.next() //从上一步断掉的yield i开始执行。因为没有参数，所以上一个yield表达式的返回值为undefined。if语句跳过向后执行i++，再遇见了yield i,此时i的值为1，所以{value:1,done:false}
g.next(true)//从上一步断掉的yield i开始执行。此时有参数为ture,所以上一步断掉的yield 表达式的返回值为true，If语句执行，i=-1,i++,i=0,再遇到yield i,此时i为0，所以{value:0,done:false}
```

再看第二个例子

```javascript
function* foo(x) {
var y = 2 * (yield (x + 1));
var z = yield (y / 3);
return (x + y + z);
}

var a = foo(5);
console.log(a.next())//执行到yield(x+1) x+1的值为a.next()的value {value:6,done:false}
console.log(a.next())//没有参数，所以上一个yield表达式的值为undefined y = NAN,执行到yield(y/3) y/3 = NAN {value:NAN,done:false}
console.log(a.next())//没有参数 上一个yield表达式的值为undefined z = undefined ，执行到x+y+z = 5+NAN+undefined = NAN。
不过NAN不能直接参与计算，必须是a = NAN  consolr.log(a + undefined)这样才可以
```





## 使用案例

### 异步操作的同步化表达

```javascript
function(函数){
    代码...
    ajax(xxx,function(){
        代码...
    })；
}
function *函数(){
    代码..
    
    yield ajax(xxx); //暂停 等待请求数据完成
    
    代码...
}
```

1、回调

```
$.ajax({
    url:xxx,
    dataType;'json',
    success(data2){
        $.ajax({
            url:xxx
        })
    }
    error(){
        
    }
})
```

2、Promise

> 将回调函数的嵌套，改成链式调用，连续读取多个文件。是回调函数的改进。

```
Promise.all([
    $.ajax({url:xxx,dataType:'json'}),
      $.ajax({url:xxx,dataType:'json'}),
     $.ajax({url:xxx,dataType:'json'})
]).then(resulets=>{
    
},err=>{
    
})
```

3、Generator

```
runner(function*(){
    let data1 = yield $.ajax({url:xxx,dataType:'json'});
     let data1 = yield $.ajax({url:xxx,dataType:'json'});
      let data1 = yield $.ajax({url:xxx,dataType:'json'});
})
```

//带逻辑 --Promise 比普通回调还麻烦 。  一次读一堆适合

//带逻辑--generaotr 非常方便 。              逻辑性

```
runner(function*(){
    let userData = yield $.ajax({url:'getUserData',dataType:'json'});
    if(userData.type = 'VIP'){
        let items = yield $.ajax({url:'getVIPItems',dataType:'json'});
    }else{
         let items = yield $.ajax({url:'getItems',dataType:'json'});
    }
})
```

#### 协程

多个线程互相协作，完成异步任务。

- 第一步，协程`A`开始执行。
- 第二步，协程`A`执行到一半，进入暂停，执行权转移到协程`B`。
- 第三步，（一段时间后）协程`B`交还执行权。
- 第四步，协程`A`恢复执行。

```javascript
function* asyncJob() {
  // ...其他代码
  var f = yield readFile(fileA);
  // ...其他代码
}
```

yield命令是异步两个阶段的分界线，执行到此处，执行权将交给其他协程。

#### Generator函数实现协程

- 暂停执行
- 恢复执行
- 函数体内外的数据交换
  - `next`返回值的 value 属性，是 Generator 函数向外输出数据；`next`方法还可以接受参数，向 Generator 函数体内输入数据。
- 错误处理机制
  - 部署错误处理代码，捕获函数体外抛出的错误。

```javascript
var fetch = require('node-fetch');

function* gen(){
  var url = 'https://api.github.com/users/github';
  var result = yield fetch(url);
  console.log(result.bio);
}

var g = gen();
var result = g.next(); //返回一个Promise对象

result.value.then(function(data){
  return data.json();    //转为JSON
}).then(function(data){
  g.next(data);         //调用下一个next方法
});
```



**为什么 Node 约定，回调函数的第一个参数，必须是错误对象`err`（如果没有错误，该参数就是`null`）？**

原因是执行分成两段，第一段执行完以后，任务所在的上下文环境就已经结束了。在这以后抛出的错误，原来的上下文环境已经无法捕捉，只能当作参数，传入第二段。

### 部署Iterator接口

>`for...of`循环可自动遍历 Generator 函数运行时生成的`Iterator`对象，不需要调用`next`方法。
>
>原生对象没有`Iterator`接口，不能用`for..of`遍历。可以通过`Generator函数`加上遍历接口就可以使用`for...of`遍历。 `将Generator函数加到对象的Symbol.iterator属性上`

```javascript
function* objectEntries() {
  let propKeys = Object.keys(this);// ?????

  for (let propKey of propKeys) {
    yield [propKey, this[propKey]];
  }
}

let jane = { first: 'Jane', last: 'Doe' };

jane[Symbol.iterator] = objectEntries;

for (let [key, value] of jane) {
  console.log(`${key}: ${value}`);
}
// first: Jane
// last: Doe
```

> `...` `Array.from` `解构赋值`调用的都是Generator返回的Iterator对象

```javascript
function* numbers () {
  yield 1
  yield 2
  return 3
  yield 4
}

// 扩展运算符
[...numbers()] // [1, 2]

// Array.from 方法
Array.from(numbers()) // [1, 2]

// 解构赋值
let [x, y] = numbers();
x // 1
y // 2

// for...of 循环
for (let n of numbers()) {
  console.log(n)
}
// 1
// 2
```



### 控制流管理

```javascript
function* longRunningTask(value1) {
  try {
    var value2 = yield step1(value1);
    var value3 = yield step2(value2);
    var value4 = yield step3(value3);
    var value5 = yield step4(value4);
    // Do something with value4
  } catch (e) {
    // Handle any error from step1 through step4
  }
}

scheduler(longRunningTask(initialValue));
function scheduler(task) {
  var taskObj = task.next(task.value);
  // 如果Generator函数未结束，就继续调用
  if (!taskObj.done) {
    task.value = taskObj.value
    scheduler(task);
  }
}
```

所有的`task`都必须是同步的，不能有异步操作。因为这里的代码一得到返回值，就继续往下执行，没有判断异步操作何时完成

### 提供类似数组的接口



