---
title: 'JavaScript 基础关注点笔记'
date: 2020-10-31T11:14:40+08:00
tags:
  - javascript
description: 'JavaScript 踩坑大全。'
image: '/images/2020/js-extends-note/header.webp'
---

JavaScript 基础关注点笔记，随着自己开发中遇到的各种问题而逐渐更新。

<!--more-->

## 数据类型

七种基本数据类型，一种复杂数据类型：

- `number`：+-(2^53-1) 范围内的数字
- `bigint`：任意长度的整数。
- `string`：字符串
- `boolean`：true 和 false
- `null`：未知的值
- `undefined`：未定义的值
- `symbol`：唯一标识符
- `object`：复杂的数据结构

`typeof` 运算符：

- `typeof null` 会返回 `"object"`，但实际上它并不是一个对象。

### 类型转换

#### 数字类型转换

```js
Number(''); // 0
Number('   123   '); // 123
Number('123z'); // NaN
Number(undefined); // NaN
Number(null); // 0
```

一元运算符 `+` 有相同的效果。注意 `null` 变成数字 `0`，`undefined` 变成 `NaN`。

#### 布尔转换

```js
Boolean(0); // false
Boolean(''); // false
Boolean('0'); // true
```

#### 运算转换

```js
alert(2 + 2 + '1'); // 41，4 + '1' = 41
alert(6 - '2'); // 4，将 '2' 转换为数字
alert('6' / '2'); // 3，将两个运算元都转换为数字
```

## 值比较

比较字符串时，使用字典顺序进行判定。按照字符串每一位依次比较，出现不同即可得出结果，若一直到有一方结束依旧未出结果，则较长的一方为大。

```js
alert(null > 0); // false
alert(null == 0); // false
alert(null >= 0); // true
alert(null === undefined); // false
```

相等性检查 `==` 和普通比较符 `> < >= <=` 的代码逻辑是相互独立的。进行值的比较时，`null` 会被转化为数字，因此它被转化为了 `0`。这就是为什么（3）中 `null >= 0` 返回值是 `true`，（1）中 `null > 0` 返回值是 `false`。

另一方面，`undefined` 和 `null` 在相等性检查 `==` 中不会进行任何的类型转换，它们有自己独立的比较规则，所以除了它们之间互等外，不会等于任何其他的值。这就解释了为什么（2）中 `null == 0` 会返回 `false`。

## 字符串

这些符号的长度是 2：

```js
alert('𝒳'.length); // 2
alert('😂'.length); // 2
alert('𩷶'.length); // 2
```

`String.fromCodePoint` 和 `str.codePointAt` 是几种处理它们少数方法。原先的 `String.fromCharCode` 和 `str.charCodeAt` 并不能很好的处理这些 UTF-16 字符。它们的作用是相同的。

同时用于循环可迭代对象的 `for of` 也能正确处理这类字符。

## 数组

在数组中搜索：

```js
const arr = [NaN];
alert(arr.indexOf(NaN)); // -1（应该为 0，但是严格相等 === equality 对 NaN 无效）
alert(arr.includes(NaN)); // true（这个结果是对的）
```

简单排序：

```js
arr.sort((a, b) => a - b);
```

## 对象

整数属性会被进行排序，其他属性则按照创建的顺序显示。

简单对象克隆：`Object.assign({}, obj);`

### 可选链

可选链 `?.` 是一种访问嵌套对象属性的防错误方法。即使中间的属性不存在，也不会出现错误。

```js
alert(user.address && user.address.street);
alert(user?.address?.street);
```

可选链 `?.` 不是一个运算符，而是一个特殊的语法结构。它还可以与函数和方括号一起使用。

```js
user.exportData?.();
user?.['assas'];
```

### 对象与数组复制

```js
// 数组
let arr = [1, 2, 3];
let arrCopy = [...arr];
let arrCopy = Object.assign([], arr);
// 对象
let obj = { a: 1, b: 2, c: 3 };
let objCopy = { ...obj };
let objCopy = Object.assign({}, obj);
```

以上代码等价，但都是浅拷贝。

### 原型链属性

`for..in` 与 `Object.hasOwnProperty()` 的不同之处在于 `for..in` 会将从原型链继承的内容一并循环。

## Symbol

```js
let id1 = Symbol('id');
let id2 = Symbol('id');
alert(id1 === id2); // false
alert(id1.toString()); // Symbol(id)
alert(id1.description); // id
// 从全局注册表中读取
let id = Symbol.for('id'); // 如果该 Symbol 不存在，则创建它
let idAgain = Symbol.for('id');
alert(id === idAgain); // true
```

如果我们使用的是属于第三方代码的某个对象，我们想要给它们添加一些标识符，这种情况下就可以给它们使用 Symbol 键而不用担心覆盖原有属性的问题。`Object.keys()` 和 `for in` 都会跳过 Symbol。

## Map 和 Set

在数组和对象之外还有两种存储数据的结构，Map 和 Set。

Map 是一个带键的数据项的集合，和对象很类似，它们的最大差别是 Map 允许使用任何类型的数据作为键，而不只是字符串和 Symbol。注意虽然可行，但 `map[key]` 不是使用 Map 的正确方式，而应该使用 `set()`、`get()`、`has()` 等方法；Set 是值的集合，它不存在键并且每一个值只能出现一次。

WeakMap 是类似于 Map 的集合，它仅允许对象作为键，并且一旦通过其他方式无法访问它们，便会将它们与其关联值一同删除；WeakSet 是类似于 Set 的集合，它仅存储对象，并且一旦通过其他方式无法访问它们，便会将其删除。它们都仅允许单个操作。

对于 Map：

```js
let john = { name: 'John' };
let map = new Map();
map.set(john, '...');
john = null;
// john 依旧被存储在 map 中，我们可以使用 map.keys() 来获取它
```

而对于 WeakMap：

```js
let john = { name: 'John' };
let weakMap = new WeakMap();
weakMap.set(john, '...');
john = null;
// john 被从内存中删除了！
```

## 函数

函数表达式是在代码执行到达时被创建，并且仅从那一刻起可用；函数声明由于声明提升，因此在代码段中被定义之前的位置它就可以被调用。

函数声明只在它所在的代码块中可见，因此若需要通过 `if` 判断条件赋值函数则需要表达式。

### rest 参数与 arguments

使用 `...rest` 于参数列表末尾访问剩余参数数组。在旧代码中，有一个名为 `arguments` 的特殊的类数组对象，该对象按参数索引包含所有参数。箭头函数没有 `arguments`。

### 装饰者模式和转发

#### 透明缓存

假设我们有一个 CPU 重负载的函数 `slow(x)`，但它的结果是稳定的。换句话说，对于相同的 `x`，它总是返回相同的结果。如果经常调用该函数，我们可能希望将结果缓存下来，以避免在重新计算上花费额外的时间。

```js
function slow(x) {
  // 这里可能会有重负载的 CPU 密集型工作
  alert(`Called with ${x}`);
  return x;
}
function cachingDecorator(func) {
  let cache = new Map();
  return function (x) {
    if (cache.has(x)) {
      return cache.get(x); // 从缓存中读取结果
    }
    let result = func(x); // 否则就调用 func
    cache.set(x, result); // 然后将结果缓存（记住）下来
    return result;
  };
}
slow = cachingDecorator(slow);
alert(slow(1)); // slow(1) 被缓存下来了
alert('Again: ' + slow(1)); // 一样的
```

`cachingDecorator(func)` 的结果就是一个 "包装器"。

#### 设定上下文

以上的包装不适用于对象方法。使用：

```js
let worker = {
  someMethod() {
    return 1;
  },
  slow(x) {
    alert('Called with ' + x); // 可怕的 CPU 过载任务
    return x * this.someMethod(); // (*)
  },
};
alert(worker.slow(1)); // 原始方法有效
worker.slow = cachingDecorator(worker.slow); // 现在对其进行缓存
alert(worker.slow(2)); // Error: Cannot read property 'someMethod' of undefined
```

使用 `func.call` 来解决：

```js
function cachingDecorator(func) {
  let cache = new Map();
  return function (x) {
    if (cache.has(x)) {
      return cache.get(x);
    }
    let result = func.call(this, x); // 现在 "this" 被正确地传递了
    cache.set(x, result);
    return result;
  };
}
worker.slow = cachingDecorator(worker.slow); // 现在对其进行缓存
alert(worker.slow(2)); // 工作正常
alert(worker.slow(2)); // 工作正常，没有调用原始函数（使用的缓存）
```

1. 在经过装饰之后，`worker.slow` 现在是包装器 `function (x) { ... }`
2. 因此，当 `worker.slow(2)` 执行时，包装器将 `2` 作为参数，并且 `this=worker` (它是点符号 `.` 之前的对象)
3. 在包装器内部，假设结果尚未缓存，`func.call(this, x)` 将当前的 `this` (worker) 和当前的参数传递给原始方法

### this 丢失

经典的 this 丢失问题：

```js
let user = {
  name: 'John',
  sayHi() {
    alert(`Hello, ${this.name}!`);
  },
};
setTimeout(user.sayHi, 1000); // Hello, !
// 等价于
let f = user.sayHi;
setTimeout(f, 1000); // 非严格模式下 this = window
```

`setTimeout` 获取到了函数 `user.sayHi`，但它和对象分离开了。

最简解决方法 (包装)：

```js
setTimeout(() => user.sayHi(), 1000); // Hello, John!
```

绑定 `this` (React 中 class 组件就需要这种方法)：

```js
let f = user.sayHi.bind(user);
setTimeout(f, 1000); // Hello, John!
```

### 箭头函数

- 没有 `this`
- 没有 `arguments`
- 不能使用 `new` 进行调用
- 它没有 `super`

```js
let group = {
  title: 'Our Group',
  students: ['John', 'Pete', 'Alice'],
  showListCorrect() {
    this.students.forEach((student) => alert(this.title + ': ' + student));
  },
  showListWrong() {
    this.students.forEach(function (student) {
      alert(this.title + ': ' + student); // Error: Cannot read property 'title' of undefined
    });
  },
};
```

## 类与继承

### 绑定方法

可以使用类字段来绑定类方法：

```js
class Button {
  constructor(value) {
    this.value = value;
  }
  click = () => {
    alert(this.value);
  };
}
const button = new Button('hello');
setTimeout(button.click, 1000); // hello
```

### 字段初始化

类字段是这样初始化的：

- 对于基类 (还未继承任何东西的那种)，在构造函数调用前初始化
- 对于派生类，在 `super()` **后**立刻初始化

因此在这种情况下两次的输出都是 `animal`：

```js
class Animal {
  name = 'animal';
  constructor() {
    alert(this.name); // (*)
  }
}
class Rabbit extends Animal {
  name = 'rabbit';
  // constructor(...args) {
  //   super(...args); // 此时 `Rabbit` 还没有自己的 `name` 字段
  // }
}
new Animal(); // animal
new Rabbit(); // animal
```
