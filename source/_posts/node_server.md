---
layout: post
title: Nodejs快速搭建服务器
tags: node
categories: 开发环境
date: 2017/7/13 20:46:25
---

## 安装node
##### Windows下安装：
* 官网下载[nodejs](https://nodejs.org/en/download/)，选择*Download for Windows (x64)*，下载好之后直接运行安装
* 将安装目录`C:\Program Files\nodejs\`添加到环境变量path中，在控制台中输入`node -v`可查看版本号。

<!-- more -->

##### Linux下安装：
*  官网下载[nodejs](https://nodejs.org/en/download/)，选择*Linux Binaries (x86/x64)*，将文件`node-v6.11.2-linux-x64.tar.xz`上传到`/usr/local/`目录下
* 在该目录下，解压：
`xz -d node-v6.11.2-linux-x64.tar.xz            #将压缩文件格式  .tar.xz 转换成  .tar`
`tar xvf node-v6.11.2-linux-x64.tar            #解压 .tar  `
* 创建个软链，链接名为node，指向刚解压出的文件*node-v6.11.2-linux-x64* : `ln -s node-v6.11.2-linux-x64  node`
* 把node的压缩包删掉`rm -f node-v6.11.2-linux-x64.tar`
* 设置环境变量：
  ` vi  /etc/profile`
  ` export NODE_HOME=/usr/local/node   #添加node路径`
 ` export PATH=$NODE_HOME/bin:$PATH   #添加到path中`
* 配置好之后，通过 `node -v`，`npm -v`即可查看安装好的node，和其自带的npm（node的包管理工具）的版本号。

## 开发工具webstorm
* [官网下载](http://www.jetbrains.com/webstorm/)，通过License server注册激活，参考[License server](http://blog.csdn.net/xx1710/article/details/51725012)

## 创建express项目

* 打开webstorm，file -> new project -> empty project，工程名为*node_demo*
* 在下方打开Terminal，初始化node项目，输入 `npm init`，输入工程相关信息，也可以一直按enter，之后会创建`package.json`文件，工程相关信息及依赖包描述文件。
* 安装express  : `npm  install express --save`，会在工程根目录下创建node_modules文件夹，其中包含了安装的模块。
* 根目录下创建index.js文件

```js
var express = require('express');   //引入express模块
const app = express();

var port = 8888;   

app.listen(port, () => { //开始监听端口
    console.log(`App listening at port ${port}`)
});

app.get('/', function(req, res){   //响应   http://localhost:8080/   的请求
    res.send('Hello world');       //返回
});
```

上述代码包含了es6的语法，webstorm默认支持es5，需要 file -> settings -> Languages&Frameworks -> JavaScript，选择 ECMAScript6
点击index.js，右键选择run..  运行，访问`localhost:8080`返回 hello world
* url的路径匹配

```js
#### index.js
var express = require('express');   //引入express模块
var router = require('./router/routers');   //  在根目录下创建 router/routers.js文件，用于匹配对应的url
const app = express();

var port = 8888;   
app.listen(port, () => {  
    console.log(`App listening at port ${port}`)
});
app.use('/v', router);   //与localhost:8080/v匹配的url会继续进入 router文件中继续向下匹配


#### /router/routers.js
var express = require('express');
var router = express.Router();

router.get('/index', function(req, res) {   //匹配  ~/index
    //允许跨域访问
    res.writeHead(200, {"Content-Type": "text/plain; charset=UTF-8;","Access-Control-Allow-Origin":"*"});
    res.end(JSON.stringify("{'key':'hello world'}"));
});

module.exports = router;
```

如上，将url匹配对应的响应函数放在/router/router.js文件中，访问[http://localhost:8080/v/index](http://localhost:8080/v/index)即可返回`"{'key':'hello world'}"`，详细的express关于路由的用法参考[router](http://www.expressjs.com.cn/guide/routing.html)
* url请求参数
获取get请求的参数通过`req.query.name`
获取post请求的参数通过`req.body.name`

* node的配置文件管理工具 [config](https://github.com/lorenwest/node-config)
安装：`npm  install config  --save`，在根目录下创建config/default.json文件，config默认读取该路径的配置文件，编辑该文件：

```json
{
    "server":{
        "port" : 8888
    }
}
```

在index.js中获取配置

```js
var express = require('express');   //引入express模块
var router = require('./router/routers');
var config = require('config');
const app = express();

app.listen(config.get('server.port'), () => {   //获取配置文件中的端口
    console.log(`App listening at port ${config.get('server.port')}`)
});

app.use('/v', router);  
```

* express访问静态文件

```js
const path = require('path');
app.use(express.static(path.join(__dirname, 'public')))
```

如上，在根目录下的public文件夹中的静态文件即可通过 *localhost:8080/xx.html* 直接访问

* 目录结构：

![](http://upload-images.jianshu.io/upload_images/1730128-03d5e4c6c45d7e50.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
如图default.json为配置文件，node_modules是安装的依赖包，public文件夹下是静态文件，router下是对响应http请求的相关函数，index.js是服务器的启动文件，package.json是项目和其依赖包等描述信息。

```js
######  default.json
{
  "server":{
    "port" : 8080
  },

  "scene1":{
    "id":1,
    "value":"abc"
  },

  "scene2":"scene2..."
}

######  index.js
var express = require('express');   //引入express模块
var router = require('./router/routers');
var config = require('config');
const path = require('path');
const app = express();
app.listen(config.get('server.port'), () => {
    console.log(`App listening at port ${config.get('server.port')}`)
});
app.use(express.static(path.join(__dirname, 'public')))
app.use('/v', router);

######  routers.js
var express = require('express');
var config = require('config');
var router = express.Router();
router.get('/index', function(req, res) {
    res.writeHead(200, {"Content-Type": "text/plain; charset=UTF-8;","Access-Control-Allow-Origin":"*"});
    if(req.query.name == "scene1"){
        res.end(JSON.stringify(config.get('scene1')));
    }
    if(req.query.name == "scene2"){
        res.end(JSON.stringify(config.get('scene2')));
    }
    res.end("param invalid");
});
module.exports = router;
```

修改了default.json和routers.js，访问 [localhost:8080/v/index?name=scene1](http://localhost:8080/v/index?name=scene1)，返回default.json中对应配置的内容。

## 将工程部署到服务器
* 直接将工程目录下的所有文件上传到服务器上，其中node_modules可不用上传。将node_demo上传到`/home/node_demo`中。
* 在`/home/node_demo`目录下，运行`npm install`，会根据package.json，自动下载相关的依赖包，生成node_modules文件夹。运行 `node index.js start`，即可执行index.js，启动服务。这种方式启动的进程是交互进程，会随着shell窗口关闭而进程终止。
* 进程管理工具 [pm2](https://github.com/Unitech/pm2)，在` /home/node_demo`目录下执行`pm2 start index.js --name="node_demo" --watch`，其中将应用命名为node_demo，--watch表示当文件改变是自动重启应用，这种方式启动的进程是守护进程，即在后台运行。
* pm2 管理的应用设置开机自启动。运行`pm2 startup`，即在`/etc/init.d/`目录下生成`pm2-root`的启动脚本，且自动将pm2-root设为服务。运行 `pm2 save`，会将当前pm2所运行的应用保存在`/root/.pm2/dump.pm2`下，当开机重启时，自动运行pm2-root服务脚本到/root/.pm2/dump.pm2下读取应用并启动。

## 示例程序
上文所示例的demo已部署到10.0.30.70服务器上的`/home/node_demo`中，由pm2管理进程，通过pm2 status可查看，更多命令参考 [pm2](https://github.com/Unitech/pm2)

![](http://upload-images.jianshu.io/upload_images/1730128-5c09520d2eab791b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可访问`10.0.30.70:8888/v/index?name=scene1`和`10.0.30.70:8888/v/index?name=scene1`返回json数据，访问静态文件`10.0.30.70:8888/index.html`