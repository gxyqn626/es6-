# Iterator（遍历器）

## 作用

- 为各种数据结构提供统一接口
- 使数据结构成员按次序排列
- 支持`for...of`

## 遍历过程

- 创建指针对象，指向数据结构的起始位置。
- 不断调用指针对象的`next`方法，直到指向数据结构的结束位置。

  - 每次调用`next`方法会返回一个对象，含有两个属性`value` `done`

    ```javascript
    function makeIterator(array) {
      var nextIndex = 0;
      return {
        next: function() {
          return nextIndex < array.length ?
            {value: array[nextIndex++]} :
            {done: true};
        }
      };
    }
    ```

## 默认Iterator接口

> 为所有数据结构，提供for...of循环
>
> 一个数据结构如果有`[Symbol.iterator]`属性，那么“可遍历”

```javascript
const obj = {
  [Symbol.iterator] : function () { //属性值为一个函数
    return {
      next: function () {
        return {				   // 返返回一个对象
          value: 1,
          done: true
        };
      }
    };
  }
};
```

### 原生具备 Iterator 接口的数据结构：

- Array
- Map
- Set
- String
- TypedArray
- 函数的 arguments 对象
- NodeList 对象

**原生具备Iterator的数据结构，for...of自动遍历，其他需要在Symbol.iterator上部署**

### 对象部署Iterator接口

> 不是很必要，因为有Map结构。
>
> Symbol.iterator属性上部署遍历生成器方法
>
> 原型链上的对象具有该方法也可以

```javascript
//原型链上
class RangeIterator {
  constructor(start, stop) {
    this.value = start;
    this.stop = stop;
  }
  [Symbol.iterator]:function() { return this; } //???????

  next() {
    var value = this.value;
    if (value < this.stop) {
      this.value++;
      return {done: false, value: value};
    }
    return {done: true, value: undefined};
  }
}
function range(start, stop) {
  return new RangeIterator(start, stop);
}
for (var value of range(0, 3)) {
  console.log(value); // 0, 1, 2
}
```

## 什么时候调用Iterator

- 对数组和Set进行解构赋值            扩展运算符

```javascript
let set = new Set().add('a').add('b').add('c')
let [x,y] = set 
console.log([hhh,...set]) //['a']     ['b'，'c']
```

- yield*

```javascript
let generator = function* () {
  yield 1;
  yield* [2,3,4];
  yield 5;
};

var iterator = generator();

iterator.next() // { value: 1, done: false }
iterator.next() // { value: 2, done: false }
iterator.next() // { value: 3, done: false }
iterator.next() // { value: 4, done: false }
iterator.next() // { value: 5, done: false }
iterator.next() // { value: undefined, done: true }
```

- 其他
  - for ... of
  - Array.from()
  - Map() Set() WeakMap() WeakSet()
  - Promise.all()
  - Promise.race()

# for...of循环

> `for...of`循环内部调用的是数据结构的`Symbol.iterator`方法
>
> 适用范围：数组、Set 和 Map 结构、某些类似数组的对象（比如`arguments`对象、DOM NodeList 对象）、后文的 Generator 对象，以及字符串。

## 数组

```
// for..of循环读取键值
const arr = ['red', 'green', 'blue'];
for(let v of arr) {
  console.log(v); // red green blue
}
//for..in循环读取键名
for(let p in arr){
    console.log(p);//0 1 2
}
//如果要通过for...of循环，获取数组的索引，可以借助数组实例的entries方法和keys方法
```

`for...of`循环调用遍历器接口，数组的遍历器接口只返回具有数字索引的属性。跟`for...in`循环不一样。

```javascript
let arr = [3, 5, 7];
arr.foo = 'hello';

for (let i in arr) {
  console.log(i); // "0", "1", "2", "foo"
}

for (let i of arr) {
  console.log(i); //  "3", "5", "7"
}
```

## Set Map

> `Set`结构返回的是值       `for(var e of engined)`
>
> `Map`结构返回的是数组    `for(var [name,value] of engined)`

```javascript
var engines = new Set(["Gecko", "Trident", "Webkit", "Webkit"]);
for (var e of engines) {
  console.log(e);
}
// Gecko
// Trident
// Webkit

var es6 = new Map();
es6.set("edition", 6);
es6.set("committee", "TC39");
es6.set("standard", "ECMA-262");
for (var [name, value] of es6) {
  console.log(name + ": " + value);
}
for (var a of es6) {
  console.log(a); //返回的是数组
}
// edition: 6
// committee: TC39
// standard: ECMA-262
```

## 对象

> 普通的对象不能使用`for...of`，必须经过部署才可以。
>
> 普通的对象可以用`for...in`遍历键名

```javascript
// for ... in 
let es6 = {
  edition: 6,
  committee: "TC39",
  standard: "ECMA-262"
};
for(let e in es6){
    console.log(e,es6[e]);
}
```

**使用`Object.keys`方法将对象的键名生成一个数组，然后遍历这个数组。**

```javascript
let es6 = {
  edition: 6,
  committee: "TC39",
  standard: "ECMA-262"
};
//Object.keys(es6)  ["edition", "committee", "standard"]
for(var key of Object.keys(es6)){
    console.log(key,es6[key])
}
```

**使用 Generator 函数将对象重新包装一下**?????????

```javascript
function* entries(obj) {
  for (let key of Object.keys(obj)) {
    yield [key, obj[key]];
  }
}

for (let [key, value] of entries(obj)) {
  console.log(key, '->', value);
}
// a -> 1
// b -> 2
// c -> 3
```

## 类似数组的对象

> `字符串`  `DOM  NodeList对象`   ` arguments对象`都可以直接使用for...of
>
> 有些可以用`Array.from`转为数组
>
> ```
> let arrayLike = { length: 2, 0: 'a', 1: 'b' };
> for (let x of Array.from(arrayLike)) {
>   console.log(x); // a  b
> }
> ```

## 计算生成的数据结构

>**ES6 的数组、Set、Map 方法**
>
>- entries()` 返回一个遍历器对象，用来遍历`[键名, 键值]`组成的数组。对于数组，键名就是索引值；对于 Set，键名与键值相同。Map 结构的 Iterator 接口，默认就是调用`entries`方法。
>- `keys()` 返回一个遍历器对象，用来遍历所有的键名。
>- `values()` 返回一个遍历器对象，用来遍历所有的键值。

```javascript
let arr = ['a', 'b', 'c'];
for (let pair of arr.entries()) {
  console.log(pair);
}
// [0, 'a']
// [1, 'b']
// [2, 'c']
```

## 几种遍历语法的比较

### for循环

> 麻烦

### forEach

>无法中途跳出`forEach`循环，`break`命令或`return`命令都不能奏效。

```javascript
let myArray = ["a","b","c"]
myArray.forEach(function (value) {
  console.log(value);   //  a  b  c
});

let myArray = [{'dd':'ss','11':"dd"},{1:'d'}]
myArray.forEach(function (value) {
  console.log(value);   //  {'dd':'ss','11':"dd"}   {1:'d'}
});
```

### for...in

> 可以遍历数组的键名，但`for...in`循环主要是为遍历对象而设计的，不适用于遍历数组。
>
> - 数组的键名是数字，但是`for...in`循环是以字符串作为键名“0”、“1”、“2”等等。
> - `for...in`循环不仅遍历数字键名，还会遍历手动添加的其他键，甚至包括原型链上的键。
> - 某些情况下，`for...in`循环会以任意顺序遍历键名。

```javascript
for (var index in myArray) {
  console.log(myArray[index]);
}
```

### for...of

>- 有着同`for...in`一样的简洁语法，但是没有`for...in`那些缺点。
>- 不同于`forEach`方法，它可以与`break`、`continue`和`return`配合使用。
>- 提供了遍历所有数据结构的统一操作接口。

