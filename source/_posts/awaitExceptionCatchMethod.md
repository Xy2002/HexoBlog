---
title: Node.js:async/await异常捕获
date: 2021-5-7 18:30:00
author: Marshall
img: https://unsplash.it/1920/1080?random2
top: false
cover: false
toc: true
mathjax: false
summary: JavaScript中优雅的处理await异常
categories: Node.js
reprintPolicy: cc_by
tags:
- Nodejs
---

Async/await 是 ES7 中的新特性，它可以让开发者编写异步代码像同步代码一样，的确它给我们带来了很多方便的地方，但是在 async/await 中如何来处理错误呢？在异步的调用中，会产生各种不同的错误，例如：HTTP 请求产生了错误、访问 DB 产生的异常、操作文件产生异常。在 Promise 的使用中，当遇到了错误，它会抛出一个异常，该异常将被捕获到一个方法回调中。在 async/await 中，我们又如何处理呢？
## Node.js:await异常捕获方法

### 一、使用try/catch捕获异常
```javascript
async function asyncFunc() {
    try {
        const token = await verifyToken(token);
        console.log(token)
    }
    catch(err) {
        console.log(err);
    }
}
```
回顾上面的代码，try/catch 的确可以来解决错误异常的处理，但是让代码非常的不干净，原本 `async/await` 的优势就是让代码更佳的简约，这样一来又违背了它的初衷，这让我们进入了新的思考。

### 二、使用to.js处理异常
#### 1.CommonJS环境下安装await-to-js
``` bash
$ npm i await-to-js --save
```
##### 1.引入
```js
const to = require('await-to-js').default;
```
##### 2.使用
```js
const[err,res]=await to(verifyToken(token))
if(err){
   console.log(err)
}else{
    console.log(res)
}
```
#### 2.自己制作to.js
```javascript
export default function to(promise) {
   return promise.then(data => {
      return [null, data];
   })
   .catch(err => [err]);
}
```

##### 使用
```javascript
import to from './to.js'
const[err,res]=await to(verifyToken(token))
if(err){
   console.log(err)
}else{
    console.log(res)
}
```
