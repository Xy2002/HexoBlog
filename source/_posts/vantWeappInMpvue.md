---
title: 在mpvue中使用vant-weapp组件库
date: 2021-4-15 21:30:00
author: Marshall
img: https://unsplash.it/1920/1080?random
top: false
cover: false
toc: true
mathjax: false
summary: mpvue可，vant-weapp可，两者在一起嘛...
categories: mpvue
reprintPolicy: cc_by
tags:
- mpvue
---

前言：因为在开发一个小程序，又正好想学学前端，于是用了mpvue框架来进行开发，思来想去又想引入一些css框架来美化页面，找了半天，一开始想用tailwind的，但是导入了半天都没办法正常使用，后面看到别人推荐[Vant Weapp](https://vant-contrib.gitee.io/vant-weapp/#/home) 。虽然过程不是很顺利，但是经过不懈摸索，还是整出来了。

## 如何在mpvue中使用Vant Weapp？

### 1.安装组件库
> 以下两种安装方式没有区别，都是将组件库复制到static目录下。 

#### npm安装
``` bash
$ npm i @vant/weapp -S --production
```
安装完后，将 `node_modules/vant-weapp/dist` 目录下的所有文件，copy 至 `static/vant` 目录下。
#### git clone
在任意非项目目录中，执行以下命令
```bash
$ git clone https://github.com/youzan/vant-weapp.git
```
克隆到本地后，将 `dist` 目录下的所有文件复制到你项目的 `/static/vant/` 目录下。
### 2.使用
#### 2.1 引入组件
##### 全局引入
在`src\app.json`下如下示例引入。
```json
"usingComponents": {
"van-button": "/static/vant/button/index",
}
```
###### 局部引入
在需要引入组件的页面目录下的 `main.json` 文件中，引入对应组件，引入方式同上。

#### 2.2 修改 project.config.json
需要手动在 `project.config.json` 内添加如下配置，使开发者工具可以正确索引到 npm 依赖的位置。
```json
{
  ...
  "setting": {
    ...
    "packNpmManually": true,
    "packNpmRelationList": [
      {
        "packageJsonPath": "./package.json",
        "miniprogramNpmDistDir": "./miniprogram/"
      }
    ]
  }
}
```

#### 2.3 组件使用
直接在.vue页面中写相应标签即可。

### 注意事项
具体组件 api 文档参考[Vant Weapp](https://vant-contrib.gitee.io/vant-weapp/#/home)
#### 使用方式
因mpvue和微信小程序的方式有所不同，所以写的时候也要看[mpvue](http://mpvue.com/)的文档
##### 1.数据绑定
原生小程序使用方式为
###### 按钮
```html
<button value="{{value}}">按钮</button>
```
mpvue 使用方式
```html
<button v-bind:value="value">按钮</button>
<button :value="value">按钮</button> 
```
###### van-field 双向绑定
示例：绑定输入框完成注册功能
```html
<van-field :value="registerform.username" label="用户名" name="username" placeholder="在此处填写" @input="registerform.username=$event.mp.detail"></van-field>
<button type="primary" @click="register">微信登录</button>
```
```js
export default{
    ...,
    data(){
        return{
            registerform:{
                username:'',
                ...
            }
            
        }
    },methods:{
        async register(){
            let{
                username,
                ...
            }={
                username:this.$data.registerform.username,
                ...
            }
                ...;
        }
        
    }
        
}
```

##### 2.事件绑定
原生小程序使用方式
```vue
bind:click="onClick"
```
mpvue 使用方式
```vue
@click="onClick"
```

##### 3.获取 event 事件对象中值
值得注意的是，mpvue中获取event值与原生小程序有所不同。举例：
```javascript
onChange(event) { // 获取表单组件filed的值
  console.log(event.mp.detail) // 注意加入mp
}
```

