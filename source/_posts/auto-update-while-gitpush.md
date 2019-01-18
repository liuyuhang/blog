---
title: Hexo+Github在push后服务端自动pull和重新构建发布blog
tags: git
categories: other
date: 2019-01-11
---
本文我只是提供了一个实现的思路和简单的实现，可以根据此思路进行更细致的实现和扩展。同时，本文的方法不仅限于Hexo blog自动发布，也适用于需要git pull代码后自动重新提供服务的所有服务组件。

## 需求及实现目标

这个需求主要是在我搭建这个blog时遇到的一个自动化问题。

我的Blog是搭建在Goolge的vps上的，代码在Github服务器，撰写Blog则是在我的笔记本上进行，当我每次有文章更新的时候，我要先将本地的新代码Push到Github服务器上，然后再登录到Google的vps上，Pull一次代码，然后再重新发布，整个过程非常繁琐麻烦。

所以我的目的是很清晰的，就是我只要在我的笔记本上Push代码，Google的虚拟机上就会自动检测到，并触发Pull新代码并自动构建发布的功能。

## 问题分析

其实解决方案有很多种，其中一种最简单的办法是将我的构建的发布版本作为github IO的发布方式进行基于github.io的发布，使用cname绑定域名，就可以解决我的需求，但这种只适合静态页面发布，如果我后续有较复杂的功能，或者是其他产品的f发布，就不能够实现了，于是我去寻找一种更通用的解决办法。

平时我的手动工作主要描述为：

1. Push：上传代码
2. Pull：在vps服务器上下载代码
3. Publish：重新构建和发布

重点主要在2和3，自动化的检测到Pull和自动化的触发构建。

经过调研，发现github有一个非常符合需求的功能，叫webhook，它可以在push的时候给一个指定的url发送post请求，我们可以根据这个post请求去触发自动化构建过程。

webhook所调用的post请求需要在vps上进行监听，由于之前我一直在使用golang编程，所以用go语言实现一个web服务用来监听webhook的请求是我的选择，各位可以根据自己的所长进行自己的选择构建web服务。

下面我以我的blog自动构建和发布作为例子向大家展示一个我的解决方案。

## 配置Github webhook

1. 进入自己的github repository，进入setting tab页，点击左侧的webhooks，如下图所示：

   ![chose-webhooks](/images/chose-webhooks.png)

2. 添加一个webhook，并配置：

   其中红框内是主要内容，liuyuhang.tech是我vps的DNS，8080是我准备接收post请求的webserver端口号，blog-webhook是我的请求路径，整体构成了webhook的URL，github默认的时候是有Push的时候，就会向这个URL发一个post请求，用以进行通知。

   ![webhook-config](/images/webhook-config.png)

3. 配置后：

   ![webhook-after-config](/images/webhook-after-config.png)

## 部署Blog发布程序

我是使用Hexo部署的Blog发布系统，具体安装和使用可以参考[HEXO](https://hexo.io)，这里不再详述，我的blog目录是/opt/blog，稍后的自动构建会使用到此目录。

我的构建过程就是进入到blog目录中，然后pull代码，执行hexo generate就可以了：

```bash
cd /opt/blog
git pull
hexo generate
```

generate之后会在/opt/blog/public下生成网站的文件，将其连接到httpd的目录，就可以使用网页访问了（后续有配置过程）

下面我们通过web程序监听8080端口，自动化执行上述功能

## 创建Web Server监听程序

我比较喜欢写go的webserver，大家可以使用其他的编程语言写，主要目的就是监听8080端口，在有post请求时能够触发调用系统命令，对待发布项目（我这里是blog）进行重新发布。

我的google vps虚拟机是centos7的，配置过程如下

1. 安装golang

   ```bash
   yum install golang
   ```
   安装httpd，将blog的发布文件夹与httpd的文件夹进行软连接

   ```bash
   yum install httpd  
   systemctl enable httpd
   systemctl start httpd
   mv /var/www/html /var/www/html.bak
   ln -s /opt/blog/public /var/www/html
   ```

   这样，每次使用hexo生成的blog，就会被httpd发布到80端口了

2. 创建项目文件夹

   ```bash
   mkdir -p /opt/blog-webhook/src
   ```

3. 编写监听程序代码

   ```bash
   vim /opt/blog-webhook/src/main.go
   ```

   ```go
   package main
   
   import (
       "log"
       "net/http"
       "os/exec"
   )
   
   func webhook(rw http.ResponseWriter, req *http.Request) {
       log.Println("start webhook function.")
       cmd := exec.Command("/bin/sh", "-c", "cd /opt/blog;git pull;hexo generate") // 自动重新构建，并发布
       err := cmd.Run()
       if err!=nil{
           log.Println(err.Error())
       }
   }
   
   func main() {
       http.HandleFunc("/blog-webhook", webhook) // 接收post请求，处理函数为webhook
       log.Fatal(http.ListenAndServe(":8080", nil)) // 监听8080端口
   }
   ```

   这里有几个部分要说明一下，由于我这个只是个示例程序，所以我完全没有获取github调用的post中的任何数据数据，这里只要是有请求，我就认为被触发了，就重新构建，大家可以根据自己的语言和框架，对这部分进行丰富，go语言方面我比较推荐gin框架，速度快，还好用。

   webhook方法中我直接执行了系统程序，首先进入到blog生成目录，然后pull一次代码，并使用hexo进行重新blog发布版，发布版的网页都会存储在/opt/blog/public文件夹下，通过软链接被httpd发布

4. 设定go语言环境变量

   要想运行go语言刚才编写的main. go，需要设定go的环境变量

   ```bash
   export GOPATH=/opt/webhook
   ```

   如果想让其长期有效，也可以使用下述方法放到.bash_profile中

   ```bash
   echo "export GOPATH=/opt/webhook" >> ~/.bash_profile
   source ~/.bash_profile
   ```

5. 运行并进行测试

   进入到go的main.go同级目录，运行go run main.go就可以启动go程序了

   ```bash
   cd /opt/webhook/src
   go run main.go
   ```

   如果没有报错，并一直处于等待状态，即为成功

6. 添加为系统service

   由于这样手动运行没有办法在系统启动时，或者终端关闭后一直运行，所以我们将其做成service进行管理

   1. 生成可执行文件

      ```bash
      cd /opt/webhook/src
      go build -o webhook main.go
      ```

      这样在/opt/webhook/src 路径下会生成一个webhook的可执行文件

   2. 编写service文件 /etc/systemd/system/blog-webhook.service

      ```bash
      [Unit]
      Description=AutoUpdateBlog
      
      [Service]
      TimeoutStartSec=0
      ExecStart=/opt/webhook/src/webhook
      Type=simple
      Restart=always
      KillMode=process
      
      [Install]
      WantedBy=multi-user.target
      ```

   3. 设定开机启动和启动service

      ```bash
      systemctl enable blog-webhook
      systemctl start blog-webhook
      ```

以上所有的工作就已经完成了，下面开始测试

## 测试

测试的时候主要是要在笔记本上进行blog的更改，然后push到github上，如果触发了消息，在github上会有记录，可以查验，同时在blog-webhook service的status状态中，会打印触发后的webhook函数中打印的值"start webhook function."，最后blog会被重新构建，网页重新刷新后，则能看到此次blog修改后的内容了。

## 注意事项

有些vps的网络安全组会屏蔽一些端口，请确定你的vps网络安全组已经放开8080端口

centos7默认是开启防火墙的，可以开放8080端口或者关闭防火墙