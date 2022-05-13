---
title: JavaScript基础题集合  
date: 2022-05-10 01:30:00  
author: Marshall  
img: https://unsplash.it/1920/1080?random9 
top: false  
cover: false  
toc: true  
mathjax: false  
summary: 刷面经时碰到到一些有趣的JavaScript题目 
categories: JavaScript  
reprintPolicy: cc0  
tags:
- JavaScript
---

## 原型链
[MDN原型链介绍](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)  

>  每个实例对象（`object`）都有一个私有属性（称之为 `__proto__` ）指向它的构造函数的原型对象（`prototype`）。该原型对象也有一个自己的原型对象（`__proto__`），层层向上直到一个对象的原型对象为 `null`。根据定义，`null` 没有原型，并作为这个原型链中的最后一个环节。 

![关系图 ](https://camo.githubusercontent.com/9a69b0f03116884e80cf566f8542cf014a4dd043fce6ce030d615040461f4e5a/68747470733a2f2f63646e2e6a7364656c6976722e6e65742f67682f6d717971696e6766656e672f426c6f672f496d616765732f70726f746f74797065352e706e67)  

### 手写bind
```javascript
Function.prototype.bind1 = function() {
    // 将参数拆解为数组
    const args = Array.prototype.slice.call(arguments) // 变成数组
    
    // 获取 this（数组第一项）
    const t = args.shift()
    
    // fn1.bind(...) 中的 fn1
    const self = this
    
    // 返回一个函数
    return function() {
        return self.apply(t, args)
    }
}
function fn1(a, b, c){
    console.log('this', this)
    console.log(a, b, c)
    return 'this is fn1'
}
const fn2 = fn1.bind1({x: 100}, 10, 20, 30)
const res = fn2()
console.log(res)
```
### 手写new
```javascript
function funcNew(obj, ...args) {
    const newObj = Object.create(obj.prototype);
    const result = obj.apply(newObj, args);
    return (typeof result === 'object' && result !== null) ? result : newObj;
}
```
### 手写instanceof
```javascript
function newInstanceOf (leftValue, rightValue) {
    if (typeof leftValue !== 'object' || rightValue == null) { 
        return false;
    }
    
    let rightProto = rightValue.prototype;
    leftValue = leftValue.__proto__;
    
    while (true) {
        if (leftValue === null) return false;
        if (leftValue === rightProto) return true;
        leftValue = leftValue.__proto__;
    }
}

/*
 * --- 验证 ---
 */

const a = [];
const b = {};

function Foo () {}

var c = new Foo()
function Child () {}
function Father() {}
Child.prototype = new Father()
var d = new Child()

console.log(newInstanceOf(a, Array)) // true
console.log(newInstanceOf(b, Object)) // true
console.log(newInstanceOf(b, Array)) // false
console.log(newInstanceOf(a, Object)) // true
console.log(newInstanceOf(c, Foo)) // true
console.log(newInstanceOf(d, Child)) // true
console.log(newInstanceOf(d, Father)) // true
console.log(newInstanceOf(123, Object)) // false 
console.log(123 instanceof Object) // false
```
### 测验题
```javascript
function Animal(){ 
  this.type = "animal"
}
   
function Dog(){ 
  this.name = "dog"
}
 
Dog.prototype = new Animal()
 
var PavlovPet = new Dog(); 
 
console.log(PavlovPet.__proto__ === Dog.prototype) //true
console.log(Dog.prototype.__proto__ === Animal.prototype) //true

```

## 变量

[MDN变量](https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript/First_steps/Variables)  
[MDN变量提升](https://developer.mozilla.org/zh-CN/docs/Glossary/Hoisting)  

### 每隔一秒输出一个数字
```javascript
for (let i = 0; i < 10; i++) {
    setTimeout(() => {
        console.log(i);
    }, 1000 * i)
}
```
```javascript
// 闭包
for (var i = 0; i < 10; i++) {
    (function(j) {
        setTimeout(() => {
            console.log(j);
        }, 1000 * j)
    })(i)
}
```
### 测验题
```javascript
var a = 1
function output () {
    console.log(a)
    var a = 2
    console.log(a)
}
console.log(a)
output()
console.log(a)

// 1 undefined 2 1
// 第一个输出：全局的 var a
// 第二个输出：output 函数中声明的 var a变量提升，还未赋值
// 第三个输出：output 函数局部作用域的 a 已赋值
// 第四输出：全局的 var a 没有变
```

```javascript
function foo() {
    let a = b = 0
    a++
    return a
}
 
foo()
console.log(typeof a)
console.log(typeof b)
// undefined number
// let a 是一个局部变量。typeof a 检查的是未声明的变量。
// b 是个全局变量，它在 foo 函数中被赋值。
```

## 数据类型
[MDN数据类型](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Data_structures)
```javascript
console.log(+true)
console.log(!"ConardLi")
// 1 false
// + 运算符首先会尝试将 boolean 类型转换为数字类型，true 被转换为 1，false 被转换为 0。
// 字符串 'ConardLi' 是一个真值，所以 !'ConardLi' 为 false。
```

## 异步
[MDN异步](https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript/Asynchronous)  
[MDN事件循环](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/EventLoop)

```javascript
for (let i = 0; i < 3; i++) {
  const log = () => {
    console.log(i)
  }
  setTimeout(log, 100)
}

// 0 1 2
//定时器是异步执行，浏览器会优先执行同步任务，在遇到定时器时会先把它们暂存在一个宏任务队列中，待当前宏任务队列的所有任务执行完毕后才会去执行队列中的任务，此时循环已执行完毕，i 已经是 3。

```

## 数组 (Array)
[MDN数组](https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript/First_steps/Arrays)  
### 数组扁平化
```javascript
// 递归
var arr = [1, [2, [3, 4]]];

function flatten(arr) {
    var result = [];
    for (var i = 0, len = arr.length; i < len; i++) {
        if (Array.isArray(arr[i])) {
            result = result.concat(flatten(arr[i]))
        }
        else {
            result.push(arr[i])
        }
    }
    return result;
}


console.log(flatten(arr))
```

```javascript
// 拓展运算符(es6)
var arr = [1, [2, [3, 4]]];

function flatten(arr) {

    while (arr.some(item => Array.isArray(item))) {
        arr = [].concat(...arr);
    }
    return arr;
}

console.log(flatten(arr))
```

### 测验题
```javascript
const clothes = ['shirt', 'socks', 'jacket', 'pants', 'hat']
clothes.length = 0
 
console.log(clothes[3])
// undefined
// 将数组的长度赋值为 0 就相当于从数组中删除所有元素。
```

```javascript
var arr = [5, 22, 14, 9];

console.log(arr.sort());

// [14, 22, 5, 9]
// sort() 方法用于对数组的元素进行排序,并返回数组。默认排序顺序是根据字符串Unicode码点。
// 详情请见V8引擎Array源码710行处 https://github.com/v8/v8/blob/ad82a40509c5b5b4680d4299c8f08d6c6d31af3c/src/js/array.js
```

## 对象(Object)

[MDN对象](https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript/Objects)  

### 深拷贝
```javascript
function deepClone(obj = {}) {
    if (typeof obj !== 'object' || obj == null) {
        // obj 是 null 或者不是对象和数组，直接返回
        return obj;
    }
    let res;
    if (obj instanceof Array) {
        res = [];
    } else {
        res = {};
    }

    for (let key in obj) {
        // 判断自身中是否包含自身属性
        if (obj.hasOwnProperty(key)) {
            res[key] = deepClone(obj[key])
        }
    }
    return res;
}
// 验证
o = {a: 1, d: {c: '4'}};
res = deepClone(o);
console.log(res);
console.log(res == o);
```

### 浅拷贝
```javascript
const hero = {
  name: 'Batman',
  city: 'Gotham'
};
// **********************方法一**********************
const heroEnhancedClone = {
  ...hero,
  name: 'Batman Clone',
  realName: 'Bruce Wayne'
};

// 验证
heroEnhancedClone;  // { name: 'Batman Clone', city: 'Gotham', realName: 'Bruce Wayne' }

// **********************方法二**********************
const { ...heroClone } = hero;

// 验证
heroClone; // { name: 'Batman', city: 'Gotham' }
hero === heroClone; // => false

// **********************方法三**********************
const hero = {
  name: 'Batman',
  city: 'Gotham'
};

// 验证
const heroClone = Object.assign({}, hero);
heroClone; // { name: 'Batman', city: 'Gotham' }
hero === heroClone; // => false
```

### 测验题
```javascript
const a = {};
const b = { key: "b" };
const c = { key: "c" };

a[b] = 123;
a[c] = 456;

console.log(a[b]);

// 456
// 对象不能做对象的 key，实际上是:
// a["Object object"] = 123;
// a["Object object"] = 456;
```

## 应用问题

### 防抖
```javascript
function debounce(func, wait) {
  let timer = null;

  return function () {
    if (timer) {
      clearTimeout(timer);
      timer = null;
    }

    let self = this;
    let args = arguments;

    timer = setTimeout(function () {
      func.apply(self, args);
      timer = null;
    }, wait);
  };
}
```

### 节流
```javascript
function throttle(func, wait) {
  let lastTime = 0;
  let timer = null;

  return function () {
    if (timer) {
      clearTimeout(timer);
      timer = null;
    }

    let self = this;
    let args = arguments;
    let nowTime = +new Date();

    const remainWaitTime = wait - (nowTime - lastTime);

    if (remainWaitTime <= 0) {
      lastTime = nowTime;
      func.apply(self, args);
    } else {
      timer = setTimeout(function () {
        lastTime = +new Date();
        func.apply(self, args);
        timer = null;
      }, remainWaitTime);
    }
  };
}
```

### 实现ajax的post
```javascript
function ajax_post(url, data) {
    // 1. 异步对象 ajax
    var ajax = new XMLHttpRequest();
    
    // 2. url 方法
    ajax.open('post', url);
    
    // 3. 设置请求报文
    ajax.setRequestHeader('Content-type', 'text/plain');
    
    // 4. 发送
    if (data) {
        ajax.send(data);
    } else {
        ajax.send();
    }
    
    // 5. 注册事件
    ajax.onreadystatechange = function () {
        if (ajax.readyState === 4 && ajax.status === 200) {
            console.log(ajax.respenseText);
        }
    }
}
```
