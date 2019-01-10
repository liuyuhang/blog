---
title: Hello World
tags: hello
date: 2019-01-10
---
Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

## Quick Start4

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
