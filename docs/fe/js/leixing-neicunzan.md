# 深入理解js数据类型与堆栈内存

## 前言

在JavaScript中，它的内存分为三种类型：代码空间、栈空间、堆空间，其中代码空间用于存放可执行代码。

本文带大家来深入理解下栈空间与堆空间（堆内存与栈内存），欢迎各位感兴趣的开发者阅读本文。

## 理解数据类型

最新的 ECMAScript 标准定义了 9 种数据类型:

- 6 种

  原始类型，使用 typeof

  运算符检查

  - [undefined](https://developer.mozilla.org/en-US/docs/Glossary/Undefined)：`typeof instance === "undefined"`
  - [boolean](https://developer.mozilla.org/en-US/docs/Glossary/Boolean)：`typeof instance === "boolean"`
  - [number](https://developer.mozilla.org/en-US/docs/Glossary/Number)：`typeof instance === "number"`
  - [string](https://developer.mozilla.org/en-US/docs/Glossary/String)：`typeof instance === "string`
  - [bigInt](https://developer.mozilla.org/en-US/docs/Glossary/BigInt)：`typeof instance === "bigint"`
  - [symbol](https://developer.mozilla.org/en-US/docs/Glossary/Symbol) ：`typeof instance === "symbol"`

- [null](https://developer.mozilla.org/en-US/docs/Glossary/Null)：`typeof instance === "object"`

- [object](https://developer.mozilla.org/en-US/docs/Glossary/Object)：`typeof instance === "object"`，任何构造函数对象实例的特殊非数据结构类型，也用做数据结构：new [Object](https://developer.mozilla.org/en-US/docs/Glossary/Object)，new [Array](https://developer.mozilla.org/en-US/docs/Glossary/Array)，new [Map](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fen-US%2Fdocs%2FGlossary%2FMap)，new [Set](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fen-US%2Fdocs%2FGlossary%2FSet)，new [WeakMap](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakMap)，new [WeakSet](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakSet)，new [Date](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date)，和几乎所有通过`new`关键词创建的东西。

- [function](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fen-US%2Fdocs%2FGlossary%2FFunction)：非数据结构，尽管 typeof 操作的结果是：`typeof instance === "function"`。这个结果是为 Function 的一个特殊缩写，尽管每个 Function 构造器都由 Object 构造器派生

> `typeof` 操作符的唯一目的就是检查数据类型，如果我们希望检查任何从 Object 派生出来的结构类型，使用 `typeof` 是不起作用的，因为总是会得到 `"object"`。检查 Object 种类的合适方式是使用 [instanceof](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fen-US%2Fdocs%2FGlossary%2Finstanceof) 关键字。但即使这样也存在误差。

### 动态类型

JavaScript 是一种**弱类型**或者说**动态**语言。我们不需要提前声明变量的类型，在程序运行过程中，类型会被自动确定。这也意味着我们可以使用同一个变量保存不同类型的数据：

```js
var info = "字符串类型"; // string类型
info = 20; // number类型
info = true; // boolean类型
```

### 隐式转换

- `+`和`-`运算符转换

```js
console.log("20" + 6) // "206" 字符串拼接 string + number = string
console.log("16" - 6) // 10 减法运算 string - number = number
```

- 比较运算符

```js
// ==（等于），会自动转换数据类型再比较
// ===（严格等于），不会自动转换数据类型，如果数据类型不一致，返回false；如果一致，再比较。
false == 0; // true
false === 0; // false
undefined == null; // true，（undefined是null的子集）
```

- NaN（Not a Number）这个特殊的Number与所有其他值都不相等，包括它自己：

```js
NaN === NaN; // false
isNaN(NaN);  // true (isNaN() 函数用于判断NaN)
```

- 浮点数相等比较

```js
1 / 3 === (1 - 2 / 3); // false
// 浮点数在运算过程中会产生误差，因为计算机无法精确表示无限循环小数。要比较两个浮点数是否相等，只能计算它们之差的绝对值，看是否小于某个阈值
Math.abs(1 / 3 - (1 - 2 / 3)) < 0.0000001; // true
```

### 包装对象

在JavaScript中，**一切皆对象**。 `Array`（数组）和 `Function`（函数）本质上都是对象，就连三种原始类型的值 — — `Number`（数值）、`String`（字符串）、`Boolean`（布尔值） — — 在一定条件下，也会自动转为对象，也就是原始类型的 **包装对象**。

一般来说，只有对象是可以对属性进行读写操作的，但是我们平常用的很多的字符串方法和属性，都是通过`.`操作符访问的，例如：

```js
console.log("神奇的程序员".length);
console.log("我是大白".indexOf("白"));
```

如上述代码所示，在我们调用这些方法和属性时，**JS内部已经隐式地帮我们帮创建了一个包装对象**了，上述代码JS在运行时会处理成这样：

```js
console.log(new String("神奇的程序员").length);
console.log(new String("我是大白").indexOf("白"));
```

浏览器自己隐式创建的包装对象和我们显式创建的包装对象不严格相等，我们举个例子说明下：

```js
var name =  "神奇的程序员";
var info = new String("神奇的程序员");
console.log(name == info);    // true
console.log(name === info);   // false
```

运行结果如下：

![af3fc521e7a04322be79370db64778e0](https://static.developers.pub/af3fc521e7a04322be79370db64778e0)

### 类型检测

接下来我们来学习下js中几个常用的类型检测方法。

#### typeof运算符

```
typeof`可以检测变量的数据类型，返回如下6种字符串`number`、`string`、`boolean`、`object`、`undefined`、`function
```

我们举个例子说明下：

```js
var age = 1;
console.log(typeof age);  // number

var info = undefined;
console.log(typeof info);  // undefined

var title = null;
console.log(typeof title);  // object，（null是空对象引用/或者说指针）。

var obj = new Object();
console.log(typeof obj);  // object

var arr = [1,2,3];
console.log(typeof arr);  // object 

var fn = function(){}
console.log(typeof fn);  // function
```

运行结果如下：

![40a78c460ae94ae1b82f8d3c92371907](https://static.developers.pub/40a78c460ae94ae1b82f8d3c92371907)

#### instanceof运算符

- `instanceof`，用于检测某个对象的原型链是否包含某个构造函数的 `prototype` 属性。
- `instanceof` 适用于检测对象，它是基于原型链运作的。
- `instanceof` 除了适用于任何 `object` 的类型检查之外，也可以用来检测内置对象，比如：`Array`、`RegExp`、`Object`、`Function`
- `instanceof` 对基本数据类型检测不起作用，主要是因为基本数据类型没有原型链。

我们举个例子来说明下：

```js
console.log([1, 2, 3] instanceof Array); // true
console.log(/abc/ instanceof RegExp); // true
console.log({} instanceof Object); // true
console.log(function() {} instanceof Function); // true
```

运行结果如下：

![b8d78aaf845d419199af6b2285aa559c](https://static.developers.pub/b8d78aaf845d419199af6b2285aa559c)

#### constructor属性

构造函数属性,可确定当前对象的构造函数，我们举个例子说明下：

```js
var o = new Object();
console.log(o.constructor == Object); // true
var arr = new Array();
console.log(arr.constructor == Array); // true
```

运行结果如下：

![f6e7c66844794d528f086c5f9197929d](https://static.developers.pub/f6e7c66844794d528f086c5f9197929d)

#### hasOwnProperty属性

判断属性是否存在于当前对象实例中（而不是原型对象中），我们举个例子来说明下：

```js
const info = { title: "书", name: "大白" };
console.log(info.hasOwnProperty("title")); // true
```

运行结果如下：

![92b54f9b2c704b77a96c522526081d8d](https://static.developers.pub/92b54f9b2c704b77a96c522526081d8d)

## 堆栈内存空间

接下来，我们看下什么是堆、栈内存空间。

### 栈内存空间

见名知意，**栈内存空间** 就是用栈作为数据结构在内存中所申请的空间。

对栈这种数据结构不了解的开发者，请移步我的另一篇文章：[数据结构:栈与队列](https://juejin.cn/post/6844904069102829581)。

我们来回顾下**栈**的特点：

- 后进先出，最后添加进栈的元素最先出。
- 访问栈底元素，必须拿掉它上面的元素。

我们画个图来描述下栈，如下所示：

![43d4a57a9f92421db4dcbaec38caf2e0](https://static.developers.pub/43d4a57a9f92421db4dcbaec38caf2e0)

### 堆内存空间

同样的，见名知意，**堆内存空间**就是用堆作为数据结构在内存中所申请的空间。

对堆这种数据结构不了解的开发者，请移步我的另外两篇文章：[数据结构:堆](https://juejin.cn/post/6844904070969294856)、[实现二叉堆](https://juejin.cn/post/6854573211197046791)

通常情况下，我们所说的 **堆** 数据结构指的是 **二叉堆** ，我们来回顾下二叉堆的特点：

- 它是一颗完全二叉树
- 二叉堆不是最小堆就是最大堆

我们画个图来描述下 **最大堆** 与 **最小堆** ，如下所示：

![4600cd06e77a429686aff4050a4bca99](https://static.developers.pub/4600cd06e77a429686aff4050a4bca99)

## 变量类型与堆栈内存的关系

### 基本数据类型

我们知道JS的基本数据类型有7种：

- `string`
- `number`
- `boolean`
- `null`
- `undefined`
- `symbol`
- `bigInt`

基本数据类型变量保存在栈内存中，因为基本数据类型占用空间小、大小固定，通过值来访问，属于被频繁使用的数据。

接下来，我们通过一个例子来讲解下，基本数据类型在栈内存中的存储：

```js
let name = "大白";
let age = 20;
```

上述代码中，我们定义了2个变量：

- name为`string`类型
- age为`number`类型

我们画个图来描述下它在栈内存的存储：

![1a75f2807968410aa001383bd7402bd1](https://static.developers.pub/1a75f2807968410aa001383bd7402bd1)

> 注意⚠️：闭包中的基本数据类型变量是保存在堆内存里的，当函数执行完弹出调用栈后，返回一个内部函数的一个引用，这时候函数的变量就会转移到堆上，因此内部函数依然能访问到上一层函数的变量。

### 引用数据类型

除了上个章节提到的基本数据类型外，其他的都属于引用数据类型，例如：`Array`、`Function`、`Object`等。

引用数据类型存储在堆内存中，引用数据类型占据空间大、大小不固定，如果存储在栈中，将影响程序的运行性能。

引用数据类型会在栈中存储一个指针，这个指针指向堆内存空间中该实体的起始地址。

当解释器寻找引用值时，会先检索其在栈中的地址，取得地址后，从堆中获得实体。

我们举个例子来描述下上述话语：

```js
// 基本数据类型-栈内存
let name = "大白";
// 基本数据类型-栈内存
let age = 20;
// 基本数据类型-栈内存
let info = null;
// 对象指针存放在栈内存中，指针指向的对象放在堆内存中
let msgObj = {msg: "测试", id: 5};
// 数组的指针存放在栈内存中，指针指向的数组存放在堆内存中
let ages = [19, 22, 57]
```

上述代码中：

- 我们创建了两个变量`msgObj`、`ages`，他们的值都是引用类型(object、array)
- 堆内存空间采用`二叉堆`作为数据结构，`msgObj`与`ages`的具体值会存在堆内存空间中
- 存储完成后，堆内存空间会返回这两个值的引用地址(指针)
- 拿到引用地址后，这个引用地址会和它的变量名对应起来，存放在栈内存空间中
- 在查找变量`msgObj`与`ages`的具体值时，会先从栈内存空间中获取它的引用地址
- 获取到引用地址后，通过引用地址在堆内存空间的二叉堆中查找到对应的值。

我们画个图来描述下上述话语，如下所示：

堆内存空间中的`Object`，表示的是存储在空间中的其他对象的引用值。

![3c67c84fcefb45e79ccc4e4e3c1ec844](https://static.developers.pub/3c67c84fcefb45e79ccc4e4e3c1ec844)

> 我们来理解下堆内存空间与堆内存的区别：
>
> 堆内存空间：相当于一个采用二叉堆作为数据结构的容器。
>
> 堆内存：指的是一个引用类型的具体值。
>
> 堆内存存在于堆内存空间中。

## 变量复制

接下来，我们从内存角度来看下变量复制。

### 基本数据类型的复制

我们通过一个例子来看下基本类型的复制，代码如下所示：

```js
let name = "神奇的程序员";
let alias = name;
alias = "大白";
```

上述代码中：

- `name`、`alias`都是基本类型，它们的值存储在栈内存。
- 它们分别有各自独立的栈空间
- 因此，修改`alias`的值，`name`不受影响

我们画个图来描述下：

![9ffb2bfa3bf2435d97eb1fa52d66f610](https://static.developers.pub/9ffb2bfa3bf2435d97eb1fa52d66f610)

### 引用数据类型的复制

接下来，我们通过一个例子来看下引用类型的复制，代码如下所示：

```js
let book = {title:"书", id: 12}
let info = book;
info.title = "故事书";
console.log(book.title); // 故事书
```

上述代码中：

- `info`、`book`都是引用类型，它们的引用存在栈内存，值存在堆内存
- 它们的值指向同一块堆内存，栈内存中会复制一份相同的引用

我们画个图来描述下：

![9ee14681c7f446bc8d302ea2b1816cdd](https://static.developers.pub/9ee14681c7f446bc8d302ea2b1816cdd)

## 深拷贝与浅拷贝

通过上述章节的学习，我们了解到引用数据类型在复制时，改了其中一个数据的值，另一个数据的值也会跟着改变，这种拷贝方式我们称为**浅拷贝**。

在实际开发中，我们希望引用类型复制到新的变量后，二者是独立的，不会因为一个的改变而影响到另一个。这种拷贝方式就称为**深拷贝**。

深拷贝，实际上就是重新在堆内存中开辟一块新的空间，把原对象的数据拷贝到这个新地址空间里来，通常来说，我们有两种方法：

- 转一遍JSON再转回来 ,但是这个办法有一个问题，这只能转化一般常见数据，function，undefined等类型都无法通过这种变回来
- 手动去写循环遍历

我们来看下第一种方法，代码如下所示：

```js
const data = { name: "大白" };
const obj = JSON.parse(JSON.stringify(data));
obj.age = 20;
console.log("data = ", data);
console.log("obj = ", obj);
```

运行结果如下：

![3912d88a1ab345c891301f88b52568be](https://static.developers.pub/3912d88a1ab345c891301f88b52568be)

最后，我们来看下第二种写法，代码如下所示：

```js
const data = [{ name: "大白" }];
let obj = data.map(item => item);
obj.push({ name: "神奇的程序员" });
console.log("data = ", data);
console.log("obj = ", obj);
```

运行结果如下：

![44797fee8a0c47a1a6a72f219955bf8e](https://static.developers.pub/44797fee8a0c47a1a6a72f219955bf8e)

## 代码地址