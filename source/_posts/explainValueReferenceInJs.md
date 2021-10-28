---
title: JavaScript的值传递和引用传递
date: 2021-10-28 08:30:00
author: Marshall
img: https://unsplash.it/1920/1080?random7
top: true
cover: false
coverImg: https://unsplash.it/1920/1080?random7
toc: true
mathjax: false
summary: 在JavaScript中，到底是值传递，还是引用传递呢？函数参数是按什么传递呢？
categories: JavaScript
reprintPolicy: cc_by
tags:
- JavaScript
---

在不断的学习中，碰到一个有趣的问题:“在JavaScript中，到底是值传递，还是引用传递呢？函数参数是按什么传递呢？”

引用文章:

* [JavaScript的值传递和引用传递](https://blog.fundebug.com/2017/08/09/explain_value_reference_in_js/)
* [JS中的值是按值传递，还是按引用传递呢？](https://segmentfault.com/a/1190000005794070)
* [JavaScript深入之参数按值传递](https://github.com/mqyqingfeng/Blog/issues/10)
* [JavaScript 深入了解基本类型和引用类型的值](https://segmentfault.com/a/1190000006752076)

## JavaScript的数据类型
JavaScript 有 5 种基本的数据类型（值类型），分别是:`Boolean`、`null`、`undefined`、`String` 和 `Number`。还有3种引用数据类型:`Object`、`Array`、`Function`，它们通过引用来传递。
### 基本数据类型
如果一个基本的数据类型绑定到某个变量，我们可以认为该变量包含这个基本数据类型的值。
并且基本类型的变量是存放在栈内存（Stack）里的
```JavaScript
var x = 10;
var y = "abc";
var z = null;
```
当我们使用=将这些变量赋值到另外的变量，实际上是将对应的值拷贝了一份，然后赋值给新的变量。我们把它称作值传递。
```JavaScript
var x = 10;
var y = "abc";

var a = x;
var b = y;

console.log(x, y, a, b); // 10, 'abc', 10, 'abc'
```
`a`和`x`都包含 10，`b`和`y`都包含'abc'，并且它们是完全独立的拷贝，互不干涉。如果我们将a的值改变，x不会受到影响。这就是值传递，不会因为被赋值后的变量改变而导致原变量改变。
### 引用数据类型
如果一个变量绑定到一个非基本数据类型(Array, Function, Object)，那么它只记录了一个内存地址，该地址存放了具体的数据。注意之前提到指向基本数据类型的变量相当于包含了数据，而现在指向非基本数据类型的变量本身是不包含数据的。
并且引用类型的值是保存在堆内存（Heap）中的对象（Object）。
与其他编程语言不同，JavaScript 不能直接操作对象的内存空间（堆内存）。
> 栈内存中保存了变量标识符和指向堆内存中该对象的指针
> 堆内存中保存了对象的内容
```JavaScript
var reference = [1];
var refCopy = reference;
reference.push(2);
console.log(reference, refCopy); // [1, 2], [1, 2]
```
`reference`和`refCopy`指向同一个数组。 如果我们更新`reference`，`refCopy`也会受到影响。可见，引用后的对象更新后，被引用的对象也更新了。
关键点:
>运算符=就是创建或修改变量在内存中的指向.
>初始化变量时为创建,重新赋值即为修改

举一个例子
```JavaScript
var a = {b: 1};// a = {b: 1}
var c = a;// c = {b: 1}
a = 2;// 重新赋值a
console.log(c);// {b: 1}
```
接着是上一段代码在内存中的分布:  

| 栈   | 堆   |
| ---- | ---- |
|   a,c   | b |

然后一步一步执行代码:

1. 创建变量a指向对象{b: 1};
2. 创建变量c指向对象{b: 1};
3. a重新指向常量区的2;  

| 栈 | 堆 | 常量区 |
| ---- | ----| ----|
| a | | 2|
| c |{b:1} | |

所以c从始至终都是指向对象{b: 1}.


### ==和===的比较
对于基本类型的变量，==会对变量进行隐式转换，可以对不同类型的变量进行比较，===不仅进行值得比较，还要进行数据类型的比较。
对于引用类型的变量，==和===只会判断引用的地址是否相同，而不会判断对象具体里属性以及值是否相同。因此，如果两个变量指向相同的对象，则返回`true`。
## 函数参数按值传递
### 定义
在《JavaScript高级程序设计》第三版 4.1.3，讲到传递参数：
> ECMAScript中所有函数的参数都是按值传递的。
>
### 按值传递
举个简单的例子：
```JavaScript
var value = 1;
function foo(v) {
    v = 2;
    console.log(v); //2
}
foo(value);
console.log(value) // 1
```
很好理解，当传递 value 到函数 foo 中，相当于拷贝了一份 value，假设拷贝的这份叫 _value，函数中修改的都是 _value 的值，而不会影响原来的 value 值。
### 按共享传递 call by sharing
准确的说，JS中的基本类型按值传递，对象类型按共享传递的(call by sharing，也叫按对象传递、按对象共享传递)。最早由Barbara Liskov. 在1974年的GLU语言中提出。该求值策略被用于Python、Java、Ruby、JS等多种语言。
该策略的重点是：调用函数传参时，函数接受对象实参引用的副本(既不是按值传递的对象副本，也不是按引用传递的隐式引用)。 它和按引用传递的不同在于：在共享传递中对函数形参的赋值，不会影响实参的值。如下面例子中，不可以通过修改形参o的值，来修改obj的值。

```JavaScript
var obj = {x : 1};
function foo(o) {
    o = 100;
}
foo(obj);
console.log(obj.x); // 仍然是1, obj并未被修改为100.
```
然而，虽然引用是副本，引用的对象是相同的。它们共享相同的对象，所以修改形参对象的属性值，也会影响到实参的属性值。
```JavaScript
var obj = {x : 1};
function foo(o) {
    o.x = 3;
}
foo(obj);
console.log(obj.x); // 3, 被修改了!
```
乍一看，函数的参数为对象时，好像是引用传递，但是红宝书说ECMAScript中所有函数的参数都是按值传递的, 这是没错的。但是**拷贝副本也是一种值的拷贝**，这时会觉得是引用传递，但实际上还是值的传递。理解这个问题的关键在于如何理解值传递和引用类型，这里要强调的是**数据类型**和**方法参数**的传递方式没有**半毛钱关系**。  
简单来说, **引用类型的值传递**, 在方法内部如果对形参重新赋值, 哪怕是同一个类的对象, 在赋值后修改对象的属性, 实参的对应的属性值都不会改变, 同时实参指向的对象也不变, 而形参在重新赋值后已经指向一个新的对象了; **而引用类型的引用传递**, 在方法内部如果对形参重新赋值, 那么实参也跟着重新赋值, 实参最初所指向的那个对象将不被任何变量所指向。其实就是说形参也会被分配一个新的地址，指向实参的值。

## 一道简单的题
```JavaScript
function changeAgeAndReference(person) {
    person.age = 25;
    person = {
        name: "John",
        age: 50
    };

    return person;
}
var personObj1 = {
    name: "Alex",
    age: 30
};
var personObj2 = changeAgeAndReference(personObj1);
console.log(personObj1); // -> ?
console.log(personObj2); // -> ?
```

最后，若有错误的地方，欢迎大家指出来！
