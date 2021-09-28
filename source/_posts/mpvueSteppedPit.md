---
title: mpvue踩过的坑
date: 2021-9-29 00:00:00
author: Marshall
img: https://unsplash.it/1920/1080?random5
top: true
cover: true
coverImg: https://unsplash.it/1920/1080?random5
toc: true
mathjax: false
summary: mpvue是一个很棒的框架
categories: mpvue
reprintPolicy: cc_by
tags:
- mpvue
- Vue.js
---

前言：暑假期间，为了完成一个校内项目，需要开发一个小程序，因为有Vue的基础，所以采用了[mpvue](http://mpvue.com/) 框架进行开发，本篇博客记录了一些我碰到的坑以及注意事项

## 为什么我新增了页面，没有反应？

因为 webpack 编译的文件是由配置的 entry 决定的，新增的页面并没有添加进 entry，所以需要手动 `npm run dev` 一下，考虑不是高频操作，所以还可以忍受

## 如何获取小程序在 page onLoad 时候传递的 options?

在所有 页面 的组件内可以通过 this.$root.$mp.query 进行获取。

## 如何正确引用并使用微信小程序的原生UI库?

利用npm安装，或者git clone后将dist目录移到相关目录后，在根目录下的`app.json`文件中，对`usingComponents`进行相关配置对应对路径即可。以[vant-weapp](https://youzan.github.io/vant-weapp/#/home) 中的van-button组件为例

```json
{
  ...,
  "usingComponents": {
    "van-button": "/static/vant/button/index",
    ...
  },
  ...
}
```

在使用时，需要对组件进行双向绑定对时候，须遵守mpvue的相关语法，此处用van-field为例，代码如下
>官方示例
```html
<van-field
    value="{{ value }}"
    placeholder="请输入用户名"
    border="{{ false }}"
    bind:change="onChange"
/>
```
```javascript
Page({
    data: {
        value: '',
    },

    onChange(event) {
        console.log(event.detail);
    },
});
```

>实际使用
```vue
<van-field
    :value="value"
    placeholder="请输入用户名"
    :border="false"
    @input="value=$event.mp.detail"
/>
```
```javascript
export default{
    data(){
        return{
            value: ''
        }
    }
}
```

很显然，我们需要做的就是将微信小程序中双向绑定的语法改为mpvue的双向绑定语法，在微信小程序中，attribute的风格为mustache style，既`{{value}}`的形式，进行修改时，不需要带`{{}}`，并且如果是绑定的是属性，需要在属性前加上`:`，如果绑定的是监听DOM事件，需要加上`@`，例如
```vue
<input value="{{value}}" />
//需改为
<input :value="value" />

<view id="tapTest" bindtap="tapName"> Click me! </view>
//需改为
<view id="tapTest" @click="tapName"> Click me! </view>
```

> 注意！监听DOM事件的修改方式需按下表进行修改

这是事件映射表，修改时需对照此表进行修改，左侧为 WEB 事件，右侧为 小程序 对应事件
```json
{
    click: 'tap',
    touchstart: 'touchstart',
    touchmove: 'touchmove',
    touchcancel: 'touchcancel',
    touchend: 'touchend',
    tap: 'tap',
    longtap: 'longtap',
    input: 'input',
    change: 'change',
    submit: 'submit',
    blur: 'blur',
    focus: 'focus',
    reset: 'reset',
    confirm: 'confirm',
    columnchange: 'columnchange',
    linechange: 'linechange',
    error: 'error',
    scrolltoupper: 'scrolltoupper',
    scrolltolower: 'scrolltolower',
    scroll: 'scroll'
}
```

## 生命周期的区别
总所周知，Vue的生命周期是精髓所在，下图为mpvue的生命周期图示
![mpvue](http://mpvue.com/assets/img/lifecycle.a8762770.jpg)
详细的解释请看[mpvue使用手册](http://mpvue.com/mpvue/) 和 [微信小程序生命周期](https://developers.weixin.qq.com/miniprogram/dev/framework/app-service/page.html)  
这里要注意的是，如无特殊需求，请不要使用小程序的生命周期钩子，善用好生命周期会对项目有极大的帮助！

## 踩过的一些坑
### click事件捕捉问题
[van-tabs组件@click事件捕捉问题](https://github.com/Rychou/mpvue-vant/issues/47)  
```javascript
// 修改前
this.trigger('click', index);
// 修改后
this.trigger('vclick', index);
```

```vue
<van-tabs :active="active" @vclick="handleClick">
```

```javascript
handleClick ({ mp }) {
  console.log(mp.detail.index)
}
```
使用van-tabs组件时，事件捕获的信息丢失，经过一番查找后才发现是mpvue的问题,这里引用一位开发者的回答
> `detail`信息被封装到`e.mp.detail`中去了。这种`vant`组件库用原生写的组件是可以捕获到`click`这种事件的。文档中指的是`vue`组件在引用时无法使用`@click`。  

因此，如果遇到click事件出现问题，请参照上面的方法对组件进行修改
### background-color在`main.json`配置没有生效
一般来说，`style tag`会自动加上`scoped`属性，`scoped`是为了解决全局css冲突的问题，因此，你的css selector是会被带上hash的，因此无法生效，例如
```css
/*你的代码*/
page{
background-color:red;
}
/*编译结果*/
page.data-v-fe19b4ea{
background-color:red;
}
```
最好不要去掉`style tag`里的`scoped`属性，我是这样解决的
```css
/*最外层的div加上一个class cover*/
.cover {
  background-color: #e5e5e5;
  height: 100%;
  position: fixed;
  left: 0;
  top: 0;
  width: 100%;
  overflow: auto;
  overscroll-behavior-y: contain;
}

```
或者
```html
<style lang="stylus" rel="stylesheet/stylus" scoped>
@import "../../../static/stylus/mixin.styl"
body
  background-color #fff
</style>
```

### 一些建议
#### request
在微信小程序中，建议对request函数进行封装，不仅提升开发时的效率，代码也会更容易理解，将`wx.request`函数进行配置后封装，返回`Promise`对象。
```javascript
//我是这样封装的，进行开发时需根据实际需求进行修改
import Config from './config'
import Token from './token'

function request (url, method, data, header = {}, noRefetch = false) {
  wx.showLoading({
    title: '加载中' //  数据请求前loading
  })
  return new Promise((resolve, reject) => {
    wx.request({
      url: Config.baseUrl + url,
      method: method,
      data: data,
      header: {
        'token': wx.getStorageSync('token'),
        'content-type': 'application/json'
      },
      success: function (res) {
        let code = res.statusCode.toString()
        let startChar = code.charAt(0)
        if (startChar === '2') {
          resolve(res.data)
        } else {
          if (code === '401') {
            if (!noRefetch) {
              Token.getTokenFromServer().then((token) => {
                //  获取token之后重新发送请求
                request(url, method, data, {}, true).then((res) => {
                  resolve(res)
                })
              })
            }
          } else {
            reject(res)
          }
        }
      },
      fail: function (error) {
        reject(error)
      },
      complete: function () {
        wx.hideLoading()
      }
    })
  })
}
function get (obj) {
  return request(obj.url, 'GET', obj.data)
}
function post (obj) {
  return request(obj.url, 'POST', obj.data)
}

export default {
  request,
  get,
  post
}

```
#### promisify
将微信小程序的api进行封装，转化为promise
```javascript
/**
 * 用于把微信原生callback的api，转换为promise
 *
 * 示例：
 *     import {promisify} from '...' // 这一部可以在全局引入或挂载
 *     const res= await promisify(wx.showModal,{content:'确认删除？'})
 */
export function promisify( api, options, ...params ) {
  return new Promise( (resolve, reject) => {
    api( Object.assign( {},  options,  {
      success: resolve,
      fail: reject
    } ), ...params )
  } ).catch( err=>console.log(err) )
}

// 暴露的是一个对象
export default {
  promisify
}

```
