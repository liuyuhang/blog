---
title: Git Push后触发Blog自动Pull和发布
tags: linux git golang
date: 2019-01-11
typora-root-url: ../../source
---
## 实现目标

这个需求主要是在我搭建这个blog时遇到的一个自动化问题。

我的Blog是搭建在Goolge的VPS上的，代码保管在Github服务器上，撰写Bolg则是在我的笔记本上进行，当我每次有文章更新的时候，我要先将本地的新代码Push到Github服务器上，然后再登录到Google的虚拟机上，Pull一次代码，然后再重新发布，整个过程非常繁琐麻烦。

所以我的目的是很清晰的，就是我只要在我的笔记本上Push代码，Google的虚拟机上就会自动检测到，并触发Pull新代码并自动构建发布的功能。

## 问题分析

其实解决方案有很多种，其中一种最简单的办法是将我的构建的发布版本作为github IO的发布方式进行基于github.io的发布，使用cname绑定域名，就可以解决我的需求，但这种只适合静态页面发布，如果我后续有较复杂的功能，或者是其他产品的f发布，就不能够实现了，于是我去寻找一种更通用的解决办法。

平时我的手动工作主要描述为：

1. Push：上传代码
2. Pull：在vps服务器上下载代码
3. Publish：重新构建和发布

重点主要在2和3，自动化的检测到Pull和自动化的触发构建。

经过调研，发现github有一个非常符合需求的功能，叫webhook，它可以在push的时候给一个你指定的url发一个post请求，我们可以根据这个post请求去触发自动化构建过程。

webhook所调用的post请求需要在vps上进行监听，由于之前我一直在使用golang编程，所以用go语言实现一个web服务用来监听webhook的请求是我的选择，各位可以根据自己的所长进行自己的选择构建web服务。

下面我以我的blog自动构建和发布作为例子向大家展示一个我的解决方案。

## 配置Github webhook

1. 进入自己的github repository，进入setting tab页，点击左侧的webhooks，如下图所示：

   ![chose-webhooks](/images/chose-webhooks.png)

2. 添加一个webhook，并配置：

   ![webhook-config](/images/webhook-config.png)




## Quick Start7

### Create a new post

``` bash
$ hexo new "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)

### Run server

``` bash
$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
$ hexo deploy
```

More info: [Deployment](https://hexo.io/docs/deployment.html)

```json
{
   "code": 200,
   "error": "",
   "success": true,
   "message": {
      "token": {
         "access_token": "a677d03a-4706-4773-b5ed-c0f1b7c61bd7.a0042a46-7313-4b60-bcee-9f23fd31da8c",
         "expiry": 1547018284,
         "token_type": "Brarer",
         "user": "a677d03a-4706-4773-b5ed-c0f1b7c61bd7"
      },
      "user": {
         "active": true,
         "created_by": "OS",
         "created_time": 1546950159,
         "description": "CreateByDefault",
         "email": "admin@uxsino.com",
         "end_time": 4102444800,
         "id": "a677d03a-4706-4773-b5ed-c0f1b7c61bd7",
         "modified_by": "OS",
         "modified_time": 1546950159,
         "name": "admin",
         "tel": "13900001111",
         "true_name": "SuperAdmin",
         "type": 1
      }
   }
}
```

```go
package setting

import (
	"github.com/go-ini/ini"
	"log"
	"time"
)
// test
type Server struct {
	RunMode      string
	HttpPort     int
	ReadTimeout  time.Duration
	WriteTimeout time.Duration
}

var ServerSetting = &Server{}

type App struct {
	PrefixUrl       string
	RuntimeRootPath string
	LogSavePath     string
	LogSaveName     string
	LogFileExt      string
	TimeFormat      string
	TokenExpiry     int
	TokenStorage    string
	SingleLogin     bool
}
```
