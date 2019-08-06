# Set

> Set类似于数组，但是成员的值都是唯一的，没有重复的值。

- Set初始化

  ```javascript
  const items = new Set([1,2,3,4,5,5,5,5,5])
  console.log(items.size)
  console.log([...items])
  ```

- 数组去重

  ```javascript
  [...new Set(array)]
  
  function dedupe(array){
      return Array.from(new Set(array))
  }
  dedupe([1,1,2,3])
  //Array.from方法可以将 Set 结构转为数组。
  ```

- 字符串去重

  ```javascript
  [...new Set('ababbc')].join('')
  ["a","b","c"].join('')
  ```

- Set内部两个NaN相等，两个对象不相等

  ```
  let set = new Set();
  let a = NaN;
  let b = NaN;
  set.add(a);
  set.add(b);
  set // Set {NaN}
  
  let set = new Set();
  set.add({});
  set.size // 1
  set.add({});
  set.size // 2
  ```

  

> Set实例的属性和方法
>
> 属性：
>
> - `Set.prototype.constructor`：构造函数，默认就是`Set`函数。
> - `Set.prototype.size`：返回`Set`实例的成员总数。
>
> 方法：操作方法（用于操作数据）和遍历方法（用于遍历成员）。
>
> - 操作方法
>   - `Set.prototype.add(value)`：添加某个值
>   - `Set.prototype.delete(value)`：删除某个值，返回一个布尔值，表示删除是否成功。
>   - `Set.prototype.has(value)`：返回一个布尔值，表示该值是否为`Set`的成员。
>   - `Set.prototype.clear()`：清除所有成员
>
> - 遍历方法
>
>   - `Set.prototype.keys()`：返回键名的遍历器
>
>   - `Set.prototype.values()`：返回键值的遍历器(键名键值相同)
>
>   - `Set.prototype.entries()`：返回键值对的遍历器 (返回相同的键值对)
>
>     ```javascript
>     Set.prototype[Symbol.iterator] === Set.prototype.values
>     // true
>     // 可以省略values方法，直接用for...of循环遍历Set
>      let set = new Set(['red','green','blue'])
>         for(let x of set){
>             console.log(x)
>         }
>     ```
>
>   - `Set.prototype.forEach()`：使用回调函数遍历每个成员
>
>     ```
>     let set = new Set([1,4,9])
>     set.forEach((value,key)=>{
>         console.log(key+':'+value)
>     })
>     ```

其他方法：

- map

  ```
  let set = new Set([1, 2, 3]);
  set = new Set([...set].map(x => x * 2));
  ```

- filter

  ```
  let set = new Set([1, 2, 3, 4, 5]);
  set = new Set([...set].filter(x => (x % 2) == 0));
  // 返回Set结构：{2, 4}
  ```

- 并集、交集、叉积

  ```
  let union = new Set([...a, ...b]);
  let intersect = new Set([...a].filter(x => b.has(x)));
  let difference = new Set([...a].filter(x => !b.has(x)));
  ```

  

# WeakSet

> 与Set类似，是不重复值的集合。
>
> **成员只能是对象，不可以其他类型 **
>
> WeakSet参数：数组或类数组的对象是（`a`数组的成员成为 WeakSet 的成员，而不是`a`数组本身）
>
> WeakSet 没有`size`、`forEach`，没有办法遍历它的成员。
>
> 方法：
>
> - **WeakSet.prototype.add(value)**：向 WeakSet 实例添加一个新成员。
> - **WeakSet.prototype.delete(value)**：清除 WeakSet 实例的指定成员。
> - **WeakSet.prototype.has(value)**：返回一个布尔值，表示某个值是否在 

创建：

```
const a = [[1, 2], [3, 4]];
const ws = new WeakSet(a);
// WeakSet {[1, 2], [3, 4]}

const b = [3, 4];
const ws = new WeakSet(b);
// 报错
```

### 弱引用？？？

> 弱引用是指不能确保其引用的对象不会被垃圾回收器回收的引用。 一个对象若只被弱引用所引用，则被认为是不可访问（或弱可访问）的，并因此可能在任何时刻被回收。

在 JavaScript 中，一般我们创建一个对象，都是建立一个强引用：

```
var obj = new Object();
```

如果我们能创建一个弱引用的对象：

```
var obj = new WeakObject();
```

我们什么都不用做，只用静静的等待垃圾回收机制执行，obj 所引用的对象就会被回收。

**我们其实建立了 arr 对 key 所引用的对象(我们假设这个真正的对象叫 Obj)的强引用。
当你设置 key = null 时，只是去掉了 key 对 Obj 的强引用，并没有去除 arr 对 Obj 的强引用，所以 Obj 还是不会被回收掉。**

```javascript
let key = new Array(5*1024);
   let arr = [
       [key,1]
   ];
   console.log(arr) // [Array[2]]
   key = null;
   console.log(arr)//[Array[2]]

let map = new Map();
let key = new Array(5 * 1024 * 1024);
// 建立了 map 对 key 所引用对象的强引用
map.set(key, 1);
// key = null 不会导致 key 的原引用对象被回收
key = null;
console.log([...map])//[Array[2]]
```

**如果你想要让 Obj 被回收掉，你需要先` delete(key)` 然后再` key = null:`**  

```javascript
let map = new Map();
let key = new Array(5 * 1024 * 1024);
map.set(key, 1);
map.delete(key)
key = null;
console.log([...map])//[]
```

如果使用WeakMap ，当key=null时，切断了Key与Obj的强引用，而对于wm，由于它与key是弱引用关系，在垃圾回收机制运行的时候，它就会被回收掉。

```javascript
let wm = new WeakMap();
let key = new Array(5 * 1024 * 1024);
wm.set(key, 1);
key = null;
```

# Map

> 在以往的Object下，键只能是字符串。
>
> Map类似对象，是键值对的集合。键可以是各种类型的值  
>
> - new Map()创建
>
> - m.set(key,value)
>
> - m.get(key)
>
> - m.has(key)
>
> - m.delete(key)
>
> - m.clear()
>
> - m.size
>
>   ```
>   const m = new Map();
>   const o = {p: 'Hello World'};  
>   m.set(o, 'content')  //o为键， ‘content’为值
>   m.get(o) // "content"
>   
>   //也可以接受一个数组作为参数。数组的成员是表示`键值对`的数组
>   const map = new Map([
>     ['name', '张三'],
>     ['title', 'Author']
>   ]);
>   map.size // 2
>   map.has('name') // true
>   ```
>
>   **只有对同一个对象的引用，Map 结构才将其视为同一个键**
>
>   **Map的键实际上是跟内存地址绑定的，只要内存地址不一样，就视为两个键**
>
>   ```javascript
>   如何比较这两个对象是否指向相同的内存地址？
>   如果obj1 == obj2 为真，则两者指向相同的内存地址。
>   
>   const map = new Map();
>   const k1 = ['a'];
>   const k2 = ['a'];
>   map.set(k1, 111).set(k2, 222);
>   map.get(k1) // 111
>   map.get(k2) // 222
>   //k1、k2值一样，但是Map结构中是两个键
>   ```
>
>   - `0`和`-0`就是一个键
>   - 布尔值`true`和字符串`true`两个不同的键
>   - `undefined`和`null`两个不同的键
>   - `NaN`不严格相等于自身，Map 将其视为同一个键。
>
>   Map 本身没有`map`和`filter`方法,但可以用扩展运算符(...)先转成数组再使用

## 遍历方法

- `Map.prototype.keys()`：返回键名的遍历器。
- `Map.prototype.values()`：返回键值的遍历器。
- `Map.prototype.entries()`：返回所有成员的遍历器。
- `Map.prototype.forEach()`：遍历 Map 的所有成员。

```javascript
const map = new Map([
    ['F','no'],
    ['T','yes']
])
for(let key of map.keys()){
    console.log(key)
}
for(let value of map.values()){
    console.log(value)
}
for(let item of map.entries()){
    console.log(item[0],item[1])
}
for(let [key,value] of map.entries()){
    console.log(key,value)
}
map.forEach((key,value)=>{
    console.log(key,value)
})
```

## 数组 Map 对象 JSON 转换

- Map转为数组

  ```
  const map = new Map([
      [1,'one'],
      [2,'two'],
      [3,'three']
  ]);
  console.log([...map])
  console.log([...map.keys()])
  console.log([...map.values()])
  console.log([...map.entries()])
  ```

- 数组转为Map

  ```
  new Map([
    [true, 7],
    [{foo: 3}, ['abc']]
  ])
  ```

- Map转对象

  ```
  const map = new Map()
  map.set('yes',true)
  map.set('no',false)
  let obj = Object.create(null)
  for(let [k,v] of map){
      obj[k] = v;
  }
  console.log(obj)
  ```

- 对象转Map

  ```
  let obj = {
      yes:true,
      no:false
  }
  let map = new Map();
  for(let k of Object.keys(obj)){
      map.set(k,obj[k])
  }
  console.log(map)
  ```

- Map转JSON

  ```javascript
  // 如果键名都是字符串，可以先转为对象，再转为JSON
    let map = new Map()
      map.set('yes',true)
      map.set('no',false)
      let obj = Object.create(null)
      for(let [k,v] of map){
          obj[k] = v;
      }
      console.log(obj)
      console.log(JSON.stringify(obj))
  // 如果Map键名有非字符串，转为数组JSON
      let map = new Map()
      map.set({foo:3},['abc'])
      map.set(true,7)
      console.log([...map])
      console.log(JSON.stringify([...map])) //[[{"foo":3},["abc"]],[true,7]]
  ```


# WeakMap

> `WeakMap`结构与`Map`结构类似，用于生成键值对的集合。
>
> `WeakMap`只接受对象作为键名（`null`除外），不接受其他类型的值作为键名。

```
const wm1 = new WeakMap();
const key = {foo: 1};
wm1.set(key, 2);
wm1.get(key) // 2            对象作键名


const k1 = [1, 2, 3];
const k2 = [4, 5, 6];
const wm2 = new WeakMap([[k1, 'foo'], [k2, 'bar']]); // 参数为数组
wm2.get(k2) // "bar"
```

```
const map = new WeakMap();
map.set(1, 2)
// TypeError: 1 is not an object!
map.set(Symbol(), 2)
// TypeError: Invalid value used as weak map key
map.set(null, 2)
// TypeError: Invalid value used as weak map key
```

> WeakMap的键名所指向的对象，不计入垃圾回收机制。只要所引用的对象的其他引用被清除，垃圾回收机制就会释放该对象所占的内存。不需要手动删除，这样避免造成内存泄漏。

```javascript
const wm = new WeakMap();
let key = {};
let obj = {foo: 1};
wm.set(key, obj);
obj = null;
wm.get(key)
// Object {foo: 1}   键值仍为正常引用
```

> 语法：
>
> - 没有遍历操作（没有`keys()`、`values()`和`entries()`方法），也没有`size`属性。
> - 无法清空，即不支持`clear`方法。
> - 只有四个方法：`get()`、`set()`、`has()`、`delete()`