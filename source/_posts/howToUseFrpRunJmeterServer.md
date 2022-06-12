---
title: 如何利用Frp在没有公网的机器上运行Jmeter Server作为slave
date: 2022-06-13 04:30:00  
author: Marshall  
img: https://unsplash.it/1920/1080?random9
top: true  
cover: true  
toc: true  
mathjax: false  
summary: 愉快的分布式测试  
categories: Jmeter  
reprintPolicy: cc_by  
tags:
- Jmeter
---

在做测试的作业时，突然想到bin目录下有个jmeter-server，看到server我就猜想是否能组成分布式，经过一番查找，发现是可行的，正好手上有一些白嫖的内网服务器，于是想利用自己部署好的Frp来给这些白嫖的服务器焕发第二春，但是教程都是利用内网机器做成分布式，显然是不适用于当前情况的。于是，在看过rmi的原理和经过一番摸索后，终于实现了自己的想法。

## Slave  

### jmeter配置
修改`bin/jmeter.properties`  

```ini
remote_hosts=127.0.0.1
#重点在于修改这两项配置，端口自定义为自己想要的端口，但是两个端口需一致
server_port=6789 
server.rmi.localport=6789 
```  

启动命令  
```bash
jmeter-server -Djava.rmi.server.hostname=0.0.0.0 #将0.0.0.0修改为frps端的ip
```

### frpc配置  
frps配置就不多介绍了，可以自行查看文档，重点在于frpc如何配置。  
因为rmi是基于`tcp/ip`协议的，所以应该应该这样配置:
```ini
[jmeter]
type = tcp
local_ip = 127.0.0.1
local_port = 6789 #修改成自己想要的端口 
remote_port = 6789 #修改成自己想要的端口
```

## Master

### 修改jmeter配置

修改`bin/jmeter.properties`  

```ini
[jmeter]
#将0.0.0.0修改为frps端端ip，端口为Slave配置的端口
remote_hosts=0.0.0.0:6789
```


