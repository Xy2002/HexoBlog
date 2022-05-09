---
title: 基于Three.js的插件-PhotoSphereViewer的基础教程
date: 2021-5-21 00:00:00
author: Marshall
img: https://unsplash.it/1920/1080?random3
top: true
cover: true
toc: true
mathjax: false
summary: 一个JavaScript库，用于显示Photo Sphere全景照片
categories: Three.js
reprintPolicy: cc_by
tags:
- Three.js
- PhotoSphereViewer
---

前言：学校想开发一个全景地图，最后找到我这个工具人来做，无奈之下只好去学Three.js爆肝，本来是用Three.js直接开发的，做到一半发现了[Photo Sphere Viewer](https://photo-sphere-viewer.js.org/)这个基于Three.js的插件，于是研究完它的文档后，重构了这个项目

# 安装Photo Sphere Viewer

## 用npm或yarn

```sh
npm install photo-sphere-viewer

yarn add photo-sphere-viewer
```

## 通过CDN
[jsDelivr](https://www.jsdelivr.com/package/npm/photo-sphere-viewer)

# 依赖关系

## 必选项

- [Three.js](https://threejs.org/) (使用`build/three.min.js`文件)
- [uEvent 2](https://github.com/mistic100/uEvent)(使用`browser.js`文件)

## 可选项

- [promise-polyfill](https://github.com/taylorhakes/promise-polyfill) 与IE兼容 (使用`dist/polyfill.min.js`文件)

# 创建第一个全景图

```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/photo-sphere-viewer@4/dist/photo-sphere-viewer.min.css"/>

<script src="https://cdn.jsdelivr.net/npm/three/build/three.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/uevent@2/browser.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/photo-sphere-viewer@4/dist/photo-sphere-viewer.min.js"></script>

<div id="viewer"></div>

<style>
    /* the viewer container must have a defined size */
    #viewer {
        width: 100vw;
        height: 50vh;
    }
</style>

<script>
    var viewer = new PhotoSphereViewer.Viewer({
        container: document.querySelector('#viewer'),
        panorama: 'path/to/panorama.jpg'
    });
</script>
```

其中的`panorama`必须是方形的图片，但是官方给出了多种适配器以进行不同方式的加载



# 适配器

## 默认适配器

panorama为默认方形全景图片

## 立方体贴图适配器

Photo Sphere Viewer支持将立方体贴图作为六个不同的图像文件。这些文件可以作为对象或数组提供。使用立方体贴图时，完全支持Photo Sphere Viewer的所有功能。

例子

```js
new PhotoSphereViewer.Viewer({
  adapter: PhotoSphereViewer.CubemapAdapter,
  panorama: {
    left:   'path/to/left.jpg',
    front:  'path/to/front.jpg',
    right:  'path/to/right.jpg',
    back:   'path/to/back.jpg',
    top:    'path/to/top.jpg',
    bottom: 'path/to/bottom.jpg',
  },
});
// 也可以以数组形式导入
panorama: [
  'path/to/left.jpg',
  'path/to/front.jpg',
  'path/to/right.jpg',
  'path/to/back.jpg',
  'path/to/top.jpg',
  'path/to/bottom.jpg',
]

```



## 等矩形瓷砖适配器(直译)

> 通过将大全景图切成许多小图块，可以减少初始加载时间和使用的带宽。

```js
new PhotoSphereViewer.Viewer({
  adapter: PhotoSphereViewer.EquirectangularTilesAdapter,
  panorama: {
    width: 6000,
    cols: 16,
    rows: 8,
    baseUrl: 'low_res.jpg',
    tileUrl: (col, row) => {
      return `tile_${col}x${row}.jpg`;
    },
  },
});
```

### 配置

`width`(必选项)

- 类型:`number`

全景图的总宽度，高度始终为宽度/ 2。

`cols`(必选项)

- 类型:`number`

列数必须是2的幂（4、6、16、32、64），最大值是64。

`rows`(必选项)

- 类型:`number`

行数必须是2的幂（2、4、6、16、32），最大值是32。

`tileUrl`(必选项)

- 类型:`function: (col, row) => string`

用于构建图块URL的函数。

`baseUrl`(可选项)

- 类型:`string`

在加载图块时显示的低分辨率完整全景图的URL。

### 准备全景图

使用[ImageMagick](https://imagemagick.org)可以轻松生成图块

假设有一个分辨率为8192x4096像素的图片，要切割成32行和16列，计算可以得出8192/32=256,4096/32=256。命令如下

```bash
magick panorama.jpg -crop 256x256 tile_%04d.jpg
```



导入的代码为

```js
    let pano={
        desc: 'example',
        minFov: 30,
        base: `./img/lowQualityImg/example_low.JPG`,
        position: {longitude: 0, latitude: 0, zoom: 50,},
        config: {
            width: 8192,
            cols: 32,
            rows: 16,
            tileUrl: (col, row) => {
                const num = row * 32 + col
                return `./img/example/tile_${('000' + num).slice(-4)}.jpg`;
            },
        },
    }
```

## 个人觉得更为模块化的写法

将加载场景的一系列api打包为一个函数，每个场景作为数组存储，在需要切换场景时，只需要一个函数即可完成，下面是加载场景的函数(以等矩形瓷砖适配器为例)



```js
function loadPanorama(pano) {
    const loader = new THREE.ImageLoader();

    viewer.loader.show();

    return new Promise((resolve, reject) => loader.load(pano.base, resolve, undefined, reject))
        .then(image => {
            const canvas = document.createElement('canvas');
            canvas.width = image.width;
            canvas.height = image.height;

            const ctx = canvas.getContext('2d');
            ctx.drawImage(image, 0, 0);
            ctx.fillStyle = 'rgba(255, 255, 255, 0.3)';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            return canvas.toDataURL('image/png');
        })
        .then(baseUrl => {
            return viewer.setPanorama({
                ...pano.config,
                baseUrl: baseUrl,
            }, pano.position);
        })
        .then(() => {
            viewer.navbar.setCaption(pano.desc);
            viewer.setOption('minFov', pano.minFov);
        });
}
```

定义场景

```js
var panos=[
        {
        desc: 'viewer1',
        minFov: 30,
        base: `./assets/img/lowQualityImg/viewer_low.JPG`,
        position: {longitude: 0, latitude: 0, zoom: 50,},
        config: {
            width: 8192,
            cols: 32,
            rows: 16,
            tileUrl: (col, row) => {
                const num = row * 32 + col
                return `./assets/img/viewer1/tile_${('000' + num).slice(-4)}.jpg`;
            },
        },
    },
    {
        desc: 'viewer2',
        minFov: 30,
        base: `./assets/img/lowQualityImg/viewer2_low.JPG`,
        position: {longitude: 1.8939487733283173, latitude: 0.03337838088394118, zoom: 50},
        config: {
            width: 8192,
            cols: 32,
            rows: 16,
            tileUrl: (col, row) => {
                const num = row * 32 + col
                return `./assets/img/viewer2/tile_${('000' + num).slice(-4)}.jpg`;
            },
        },
    }
]
```

切换场景时

```js
loadPanorama(panos[1])
```

