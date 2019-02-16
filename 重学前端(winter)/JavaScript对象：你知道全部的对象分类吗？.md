# JavaScript对象：你知道全部的对象分类吗？

## JavaScript 中的对象分类


* 宿主对象（host Objects）：由 JavaScript 宿主环境提供的对象，它们的行为完全由宿主环境决定。

* 内置对象（Built-in Objects）：由 JavaScript 语言提供的对象。

    * 固有对象（Intrinsic Objects ）：由标准规定，随着 JavaScript 运行时创建而自动创建的对象实例。
    * 原生对象（Native Objects）：可以由用户通过 Array、RegExp 等内置构造器或者特殊语法创建的对象。
    * 普通对象（Ordinary Objects）：由{}语法、Object 构造器或者 class 关键字定义类创建的对象，它能够被原型继承。
    
## 宿主对象

在浏览器环境中，我们都知道全局对象是 window，window 上又有很多属性，如 document。

实际上，这个全局对象 window 上的属性，一部分来自 JavaScript 语言，一部分来自浏览器环境。

JavaScript 标准中规定了全局对象属性，w3c 的各种标准中规定了 Window 对象的其它属性。

宿主对象也分为固有的和用户可创建的两种，比如 document.createElement 就可以创建一些 dom 对象。

宿主也会提供一些构造器，比如我们可以使用 new Image 来创建 img 元素。

## 内置对象·固有对象

固有对象是由标准规定，随着 JavaScript 运行时创建而自动创建的对象实例。

固有对象在任何 JS 代码执行前就已经被创建出来了，它们通常扮演者类似基础库的角色。我们前面提到的“类”其实就是固有对象的一种。

## 内置对象·原生对象

我们把 JavaScript 中，能够通过语言本身的构造器创建的对象称作原生对象。在 JavaScript 标准中，提供了 30 多个构造器。按照我的理解，按照不同应用场景，我把原生对象分成了以下几个种类。

![原生对象](img/objects.png)

通过这些构造器，我们可以用 new 运算创建新的对象，所以我们把这些对象称作原生对象。

几乎所有这些构造器的能力都是无法用纯 JavaScript 代码实现的，它们也无法用 class/extend 语法来继承。

这些构造器创建的对象多数使用了私有字段, 例如：

* Error: [[ErrorData]]
* Boolean: [[BooleanData]]
* Number: [[NumberData]]
* Date: [[DateValue]]
* RegExp: [[RegExpMatcher]]
* Symbol: [[SymbolData]]
* Map: [[MapData]]

这些字段使得原型继承方法无法正常工作，所以，我们可以认为，所有这些原生对象都是为了特定能力或者性能，而设计出来的“特权对象”。

## 用对象来模拟函数与构造器：函数对象与构造器对象

我在前面介绍了对象的一般分类，在 JavaScript 中，还有一个看待对象的不同视角，这就是用对象来模拟函数和构造器。

事实上，JavaScript 为这一类对象预留了私有字段机制，并规定了抽象的函数对象与构造器对象的概念。

* 函数对象的定义是：具有 [[call]] 私有字段的对象
* 构造器对象的定义是：具有私有字段 [[construct]] 的对象。

JavaScript 用对象模拟函数的设计代替了一般编程语言中的函数，它们可以像其它语言的函数一样被调用、传参。任何宿主只要提供了“具有 [[call]] 私有字段的对象”，就可以被 JavaScript 函数调用语法支持。

> [[call]] 私有字段必须是一个引擎中定义的函数，需要接受 this 值和调用参数，并且会产生域的切换，这些内容，我将会在属性访问和执行过程两个章节详细讲述。

我们可以这样说，任何对象只需要实现 [[call]]，它就是一个函数对象，可以去作为函数被调用。而如果它能实现 [[construct]]，它就是一个构造器对象，可以作为构造器被调用。

对于为 JavaScript 提供运行环境的程序员来说，只要字段符合，我们在上文中提到的宿主对象和内置对象（如 Symbol 函数）可以模拟函数和构造器。

当然了，用户用 function 关键字创建的函数必定同时是函数和构造器。不过，它们表现出来的行为效果却并不相同。

对于宿主和内置对象来说，它们实现 [[call]]（作为函数被调用）和 [[construct]]（作为构造器被调用）不总是一致的。比如内置对象 Date 在作为构造器调用时产生新的对象，作为函数时，则产生字符串，见以下代码：

```
typeof new Date() //object
typeof Date()  // string
```

而浏览器宿主环境中，提供的 Image 构造器，则根本不允许被作为函数调用。

```
console.log(new Image);
console.log(Image());// 抛出错误
```

基本类型（String、Number、Boolean），它们的构造器被当作函数调用，则产生类型转换的效果。

在 ES6 之后 => 语法创建的函数仅仅是函数，它们无法被当作构造器使用，见以下代码：

```
new (a => 0) // error
```

对于用户使用 function 语法或者 Function 构造器创建的对象来说，[[call]] 和 [[construct]] 行为总是相似的，它们执行同一段代码。

```
function f(){
    return 1;
}
var v = f(); // 把 f 作为函数调用
var o = new f(); // 把 f 作为构造器调用
```

我们大致可以认为，它们 [[construct]] 的执行过程如下：

* 以 Object.protoype 为原型创建一个新对象；
* 以新对象为 this，执行函数的 [[call]]；
* 如果 [[call]] 的返回值是对象，那么，返回这个对象，否则返回第一步创建的新对象。

这样的规则造成了个有趣的现象，如果我们的构造器返回了一个新的对象，那么 new 创建的新对象就变成了一个构造函数之外完全无法访问的对象，这一定程度上可以实现“私有”。

```
function cls(){
    this.a = 100;
    return {
        getValue:() => this.a
    }
}
var o = new cls;
o.getValue(); //100
//a 在外面永远无法访问到
```

## 特殊行为的对象

除了上面介绍的对象之外，在固有对象和原生对象中，有一些对象的行为跟正常对象有很大区别。

它们常见的下标运算（就是使用中括号或者点来做属性访问）或者设置原型跟普通对象不同，这里我简单总结一下。

* Array：Array 的 length 属性根据最大的下标自动发生变化。
* Object.prototype：作为所有正常对象的默认原型，不能再给它设置原型了。
* String：为了支持下标运算，String 的正整数属性访问会去字符串里查找。
* Arguments：arguments 的非负整数型下标属性跟对应的变量联动。
* 模块的 namespace 对象：特殊的地方非常多，跟一般对象完全不一样，尽量只用于 import 吧。
* 类型数组和数组缓冲区：跟内存块相关联，下标运算比较特殊。
* bind 后的 function：跟原来的函数相关联。

## 总结

在这篇文章中，我们介绍了一些不那么常规的对象，还介绍了 JavaScript 中用对象来模拟函数和构造器的机制。

不使用 new 运算符，尽可能找到获得对象的方法。

```
let a;
    a = []
    a = {}
    a = /a/
    a = document.createElement('p')
    a = Object.create(null)
    a = Object.assign(null)
    a = JSON.parse('{}')
    a = Object(undefined)
    a = Object.prototype.constructor()
```

## 小实验：获取全部 JavaScript 固有对象

我们从 JavaScript 标准中可以找到全部的 JS 对象定义。

JS 语言规定了全局对象的属性。

三个值：
Infinity、NaN、undefined。


九个函数：

* eval
* isFinite
* isNaN
* parseFloat
* parseInt
* decodeURI
* decodeURIComponent
* encodeURI
* encodeURIComponent

一些构造器：

Array、Date、RegExp、Promise、Proxy、Map、WeakMap、Set、WeapSet、Function、Boolean、String、Number、Symbol、Object、Error、EvalError、RangeError、ReferenceError、SyntaxError、TypeError
URIError、ArrayBuffer、SharedArrayBuffer、DataView、Typed Array、Float32Array、Float64Array、Int8Array、Int16Array、Int32Array、UInt8Array、UInt16Array、UInt32Array、UInt8ClampedArray。



四个用于当作命名空间的对象：

* Atomics
* JSON
* Math
* Reflect

我们使用广度优先搜索，查找这些对象所有的属性和 Getter/Setter，就可以获得 JavaScript 中所有的固有对象。

```
var set = new Set();
var objects = [
    eval,
    isFinite,
    isNaN,
    parseFloat,
    parseInt,
    decodeURI,
    decodeURIComponent,
    encodeURI,
    encodeURIComponent,
    Array,
    Date,
    RegExp,
    Promise,
    Proxy,
    Map,
    WeakMap,
    Set,
    WeakSet,
    Function,
    Boolean,
    String,
    Number,
    Symbol,
    Object,
    Error,
    EvalError,
    RangeError,
    ReferenceError,
    SyntaxError,
    TypeError,
    URIError,
    ArrayBuffer,
    SharedArrayBuffer,
    DataView,
    Float32Array,
    Float64Array,
    Int8Array,
    Int16Array,
    Int32Array,
    Uint8Array,
    Uint16Array,
    Uint32Array,
    Uint8ClampedArray,
    Atomics,
    JSON,
    Math,
    Reflect];
objects.forEach(o => set.add(o));

for(var i = 0; i < objects.length; i++) {
    var o = objects[i]
    for(var p of Object.getOwnPropertyNames(o)) {
        var d = Object.getOwnPropertyDescriptor(o, p)
        if( (d.value !== null && typeof d.value === "object") || (typeof d.value === "function"))
            if(!set.has(d.value))
                set.add(d.value), objects.push(d.value);
        if( d.get )
            if(!set.has(d.get))
                set.add(d.get), objects.push(d.get);
        if( d.set )
            if(!set.has(d.set))
                set.add(d.set), objects.push(d.set);
    }
}
```




















