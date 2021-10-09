---
title: Vue和Electron的所有坑总结
date: 2021-6-10 00:00:00
author: Marshall
img: https://unsplash.it/1920/1080?random4
top: true
cover: true
coverImg: https://unsplash.it/1920/1080?random4
toc: true
mathjax: false
summary: 将Vue与Electron相结合,快速构建桌面级应用
categories: Electron
reprintPolicy: cc_by
tags:
- Electron
- Vue.js
---


# Electron
[Electron](https://www.electronjs.org/) 相当于一个浏览器的外壳，可以把网页程序嵌入到壳里面，可以运行在桌面上的一个程序，可以把网页打包成一个在桌面运行的程序，通俗来说就是软件，比如像QQ、优酷、网易音乐等等。功能的强大超出你的想象，可以构建跨平台桌面程序，本身支持node.js，可以使用node.js的一些模块。

## 一、项目搭建
### 1. 安装electron
```bash
vue add electron-builder
```

> 安装完成后，查看项目的目录结构，会自动在src目录下生成`background.js`并且修改`package.json`

### 2.运行

```bash
npm run electron:serve
```

在进行此步骤时，有很大的几率会报错`Error: post install error, please remove node_modules before retry!` ，此时需要将node_modules全部删掉，然后重新执行npm install。又有很大的几率会报错，内容大致是提示叫你重新安装electron，此时只需要将node_modules下的electron文件夹删除，然后执行npm install electron就可以了。在安装electron时的相关npm包时，官方推荐使用yarn，但是因为个人习惯原因，使用npm来进行安装。

### 3.打包

```bash
npm run electron:build
```

上述三个步骤看上去十分简单，但是实际进行操作时，出现了一堆错误，下面的内容就是总结所有踩过的坑。

## 二、总结遇到的坑

### 1. Vuex

#### 1.Vuex的安装

在开发时，我当时直接使用了`npm install vuex`来使用vuex，一切问题都没有，非常的正常，但是build后再打开，发现vuex失效了，后面经过查找才发现，在electron使用vuex时，应该使用下面的命令安装。

```bash
npm install vuex-electron --save
```

#### 2.Vuex的配置

第一步，先在`src`目录下新建一个文件夹`store`，然后在这个文件夹内创建文件`index.js`

根据官方的样例[Vuex Electron](https://github.com/vue-electron/vuex-electron-example)，`index.js`的内容如下

```javascript
import Vue from "vue"
import Vuex from "vuex"

import { createPersistedState, createSharedMutations } from "vuex-electron"

Vue.use(Vuex)

export default new Vuex.Store({
  state: {
    count: 0
  },

  actions: {
    increment(store) {
      store.commit("increment")
    },
    decrement(store) {
      store.commit("decrement")
    }
  },

  mutations: {
    increment(state) {
      state.count++
    },
    decrement(state) {
      state.count--
    }
  },

  plugins: [createPersistedState(), createSharedMutations()],
  strict: process.env.NODE_ENV !== "production"
})
```

但是这会引发一个问题，如果你的项目里的代码全都是`this.$sotre.commit`，会发现执行相关代码时根本没有反应，并且控制台会报错`Please, don't use direct commit's, use dispatch instead of this.`，这是因为在import时，将`createSharedMutations`也一并引入了。

如果你用不上多进程共享数据的话，可以选择不导入`createSharedMutations`，注释或者删除相关代码，或者使用dispatch来进行相关操作。

### 2. 构建遇到的问题

#### 1.在`.vue`文件中无法使用electronApi的问题

在`src/main.js`添加下面两行代码

```javascript
import electron from 'electron'
Vue.prototype.$electron = electron;
```

`vue.config.js`配置防止浏览器报错`'__dirname' is not defind`

```javascript
module.exports = {
    pluginOptions: {
        electronBuilder: {
            nodeIntegration: true
        }
    }
}
```

#### 2.在`.vue`文件中使用`this.$electron.remote`字段为`undefined`

需要“显”式的申明 enableRemoteModule: true
在项目目录`src`下的`background.js`中修改

```javascript
const win = new BrowserWindow({
        width: 400,
        height: 400,
        frame: false,
        resizable: false,
        webPreferences: {
            // Use pluginOptions.nodeIntegration, leave this alone
            // See nklayman.github.io/vue-cli-plugin-electron-builder/guide/security.html#node-integration for more info
            nodeIntegration: process.env.ELECTRON_NODE_INTEGRATION,
            enableRemoteModule: process.env.ELECTRON_NODE_INTEGRATION
        }
    });
```

在`vue.config.js`中添加

```javascript
module.exports = {
    pluginOptions: {
        electronBuilder: {
            nodeIntegration: true,
            enableRemoteModule: true
        }
    }
}
```




#### 3.构建好的项目想让别人也使用，因此想取消顶部工具栏

在`background.js`里，修改最上方的代码，将Menu也引入进来

```javascript
import {app, protocol, BrowserWindow, Menu, dialog} from 'electron'
```

然后修改 `createWindow()`

```javascript
  const win = new BrowserWindow({
    width: 1294,
    height: 800,
    webPreferences: {

      // Use pluginOptions.nodeIntegration, leave this alone
      // See nklayman.github.io/vue-cli-plugin-electron-builder/guide/security.html#node-integration for more info
      nodeIntegration: process.env.ELECTRON_NODE_INTEGRATION,
      contextIsolation: !process.env.ELECTRON_NODE_INTEGRATION,
      enableRemoteModule: process.env.ELECTRON_NODE_INTEGRATION
    }
  })
  Menu.setApplicationMenu(null)
```

#### 4.需要自定义build出来的exe等安装程序

在根目录下创建文件`vue.config.js`，然后进行相关配置，代码如下

```javascript
module.exports = {
    publicPath: process.env.NODE_ENV === 'production' ? './' : '/',
    pluginOptions: {
        electronBuilder: {
            nodeIntegration: true,
            builderOptions: {
                'appId': 'example',
                'productName': 'example', // 项目名，也是生成的安装文件名，即mzDemo.exe
                'copyright': 'Marshall Copyright © 2021', // 版权信息
                'files': [
                    './**/*'
                ],
                'directories': {
                    'output': './app_dist' // 输出文件路径
                },
                'win': { // win相关配置
                    'icon': './public/favicon.ico', // 图标，当前图标在根目录下，注意这里有两个坑
                    "requestedExecutionLevel": "requireAdministrator", //获取管理员权限
                    'target': [{
                        'target': 'nsis', // 利用nsis制作安装程序
                        'arch': [
                            'x64', // 64位
                            'ia32'
                        ]
                    }]
                },
                artifactName: '${productName}_Setup_${version}_${platform}.${ext}',
                'nsis': {
                    include: './installer.nsh',//默认安装路径
                    'oneClick': false, // 是否一键安装
                    'allowElevation': true, // 允许请求提升。 如果为false，则用户必须使用提升的权限重新启动安装程序。
                    'allowToChangeInstallationDirectory': true, // 允许修改安装目录
                    'installerIcon': './favicon.ico', // 安装图标
                    'uninstallerIcon': './favicon.ico', // 卸载图标
                    'installerHeaderIcon': './favicon.ico', // 安装时头部图标
                    'createDesktopShortcut': true, // 创建桌面图标
                    'createStartMenuShortcut': true, // 创建开始菜单图标
                    'shortcutName': 'nfuEcampusUtils' // 图标名称(项目名称)
                }
            }
        }
    }
}
```

在上面代码里,`nsis`的配置里出现了一个`include : './installer.nsh'` 这个文件是用来配置默认安装路径的。同样，在根目录创建文件`installer.nsh`，我是这样写的:

```nsis
!macro preInit
  SetRegView 64
  WriteRegExpandStr HKLM "${INSTALL_REGISTRY_KEY}" InstallLocation "C:\Program Files\${PRODUCT_NAME}"
  WriteRegExpandStr HKCU "${INSTALL_REGISTRY_KEY}" InstallLocation "C:\Program Files\${PRODUCT_NAME}"
  SetRegView 32
  WriteRegExpandStr HKLM "${INSTALL_REGISTRY_KEY}" InstallLocation "C:\Program Files (x86)\${PRODUCT_NAME}"
  WriteRegExpandStr HKCU "${INSTALL_REGISTRY_KEY}" InstallLocation "C:\Program Files (x86)\${PRODUCT_NAME}"
!macroend

```

#### 5.我想让我的项目能持续迭代更新，并且让使用者也能在使用时收到更新通知

本方法的使用条件是：你开发的是一个私有的 Electron 应用程序，或者你没有在 GitHub Releases 中公开发布。官方给了几种方法[部署更新服务器](https://www.electronjs.org/docs/tutorial/updates#%E9%83%A8%E7%BD%B2%E6%9B%B4%E6%96%B0%E6%9C%8D%E5%8A%A1%E5%99%A8)

但我觉得都太麻烦了，想直接将新版放到oss里面，并且速度也更客观，于是我进行查找后，得到了有效的方案。

先修改`vue.config.js`里的`builderOptions`

```javascript
module.exports={
    ...,
    pluginOptions:{
    	electronBuilder：{
    		···，
    		builderOptions:{
    			...,
    			publish:[{provider: 'generic',url:'https://yourdomain.com/app'}],
                 ...
			}		
		}
	}
}
```

安装依赖并且修改`background.js`

```bash
npm install electron-updater
```

```javascript
const {autoUpdater} = require('electron-updater')
async function createWindow() {
	const win = new BrowserWindow({
        ...,
        if (process.env.WEBPACK_DEV_SERVER_URL) {
        await win.loadURL(process.env.WEBPACK_DEV_SERVER_URL)
        if (!process.env.IS_TEST) win.webContents.openDevTools()
    }else{
        createProtocol('app')
        win.loadURL('app://./index.html')
        autoUpdater.checkForUpdates()
    }
    })
}
```

```javascript
// ======================================================================
// 更新模块
// ======================================================================
if (!process.env.WEBPACK_DEV_SERVER_URL) {
  autoUpdater.autoDownload = false

  // autoUpdater.signals.updateDownloaded(() => {})
  autoUpdater.on('error', (error) => {
    log.warn('检查更新失败: ' + error == null ? 'unknown' : (error.stack || error).toString())
    dialog.showErrorBox('Error: ', error == null ? 'unknown' : (error.stack || error).toString())
  })

  autoUpdater.on('update-available', (info) => {
    // var appInfo = {
    //   info: info.version,
    //   files: info.files,
    //   path: info.path,
    //   sha512: info.sha512,
    //   releaseDate: info.releaseDate
    // }
    console.log(info)
    dialog.showMessageBox({
      type: 'info',
      title: '更新提示',
      message: '软件需要更新，您是否立即更新？',
      buttons: ['推迟', '立即更新']
    }).then((res) => {
      log.warn('index:' + res.response)
      if (res.response === 1) {
        log.warn('选择升级')
        autoUpdater.downloadUpdate()
      } else {
        log.warn('选择不升级:')
      }
    })
  })

  // 检查更新时触发
  autoUpdater.on('update-available', (res) => {
    log.warn('检查更新时触发')
    log.warn(res)
    // dialog.showMessageBox({
    //   title: '检查更新',
    //   message: '正在检查更新'
    // })
  })

  // 没有可用更新
  autoUpdater.on('update-not-available', () => {
    log.warn('没有可用更新')
    // dialog.showMessageBox({
    //   title: '已是最新版',
    //   message: '当前版本是最新版本。'
    // })
  })

  // 安装更新
  autoUpdater.on('update-downloaded', (res) => {
    log.warn(res)
    // console.log(res)
    log.warn('下载完毕！提示安装更新')
    dialog.showMessageBox({
      title: '升级提示！',
      message: '已自动升级为最新版，请重启应用！'
    }, () => {
      log.warn('确认安装')
      setImmediate(() => autoUpdater.quitAndInstall(true, true))
    })
  })

  //下载进度
  autoUpdater.on('download-progress', (event) => {
    log.warn(event.percent)
  })
}
```

用build打包第一个版本(1.0.0)

打包后 `dist_electron` 目录中有 `*.blockmap` 格式的文件;

将文件复制到 ‘更新服务器’ (http://yourdomain.com/app/) 目录下;

然后再打包一个升级版(1.1.0)

打包后 `dist_electron` 有如下三个文件：

- `新版本安装包.exe`
- `新版本_v1.1.0.blockmap`，
- `latest.yml`

将上面三个文件复制到 ‘更新服务器’ (http://yourdomain.com/app/) 目录下;

以后每次有更新就复制这三个文件至 ‘更新服务器’，保留旧版本的 `*.blockmap` 文件，旧版本的应用的执行文件(`.exe`)可以删除。

