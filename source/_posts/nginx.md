---
layout: post
title: nginx安装及设置自启动
date: 2017-06-30 15:25:34
tags: nginx
categories: 开发环境
---


安装过程参考 http://www.nginx.cn/install
nginx相关文档 http://www.nginx.cn/doc/index.html

#### 安装过程
* **准备环境**
  * 安装make：`yum -y install gcc automake autoconf libtool make`
  * 安装g++：`yum install gcc gcc-c++`

<!-- more -->

* **1、选定源码目录** `/usr/local/src`，存储源码

* **2、安装PCRE库**，*注意：若pcre-8.40路径找不到，到ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/下选取版本号*
```sh
cd /usr/local/src
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.40.tar.gz
tar -zxvf pcre-8.40.tar.gz
cd pcre-8.40
./configure
make
make install
```

* **3、安装zlib库**，*注意：http://zlib.net/ 下选取最新的版本号*
```sh
cd /usr/local/src
wget http://zlib.net/zlib-1.2.11.tar.gz
tar -zxvf zlib-1.2.11.tar.gz
cd zlib-1.2.11
./configure
make
make install
```

* **4、安装ssl**
```sh
cd /usr/local/src
wget https://www.openssl.org/source/openssl-1.0.1t.tar.gz
tar -zxvf openssl-1.0.1t.tar.gz
```
* **5、安装nginx**
```sh
cd /usr/local/src
wget http://nginx.org/download/nginx-1.4.2.tar.gz
tar -zxvf nginx-1.4.2.tar.gz
cd nginx-1.4.2
./configure --sbin-path=/usr/local/nginx/nginx \
--conf-path=/usr/local/nginx/nginx.conf \
--pid-path=/usr/local/nginx/nginx.pid \
--with-http_ssl_module \
--with-pcre=/usr/local/src/pcre-8.40 \
--with-zlib=/usr/local/src/zlib-1.2.11 \
--with-openssl=/usr/local/src/openssl-1.0.1t
make
make install
```

<br />
* **6、启动nginx**，  安装成功后nginx在 `/usr/local/nginx`目录下
   保证80端口未被占用，运行`/usr/local/nginx/nginx`，启动nginx，访问ip，出现Welcome to nginx!



<br /><br />


> ### 设置nginx为服务，开机自启动###

* 在`/etc/init.d`目录下新建nginx启动脚本，其中脚本内容中的modify部分需要根据自己的安装目录来设置：
```sh
#!/bin/bash
# nginx Startup script for the Nginx HTTP Server
# it is v.1.12.0 version.
# chkconfig: - 85 15
# description: Nginx is a high-performance web and proxy server.
#              It has a lot of features, but it's not for everyone.
# processname: nginx
# pidfile: /var/run/nginx.pid
# config: /usr/local/nginx/conf/nginx.conf
#modify
nginxd=/usr/local/nginx/nginx                                   #nginx的启动文件
nginx_config=/usr/local/nginx/nginx.conf                        #
nginx_pid=/usr/local/nginx/nginx.pid                            #
#modify
RETVAL=0
prog="nginx"
# Source function library.
. /etc/rc.d/init.d/functions
# Source networking configuration.
. /etc/sysconfig/network
# Check that networking is up.
[ ${NETWORKING} = "no" ] && exit 0
[ -x $nginxd ] || exit 0
# Start nginx daemons functions.
start() {
if [ -e $nginx_pid ];then
   echo "nginx already running...."
   exit 1
fi
   echo -n $"Starting $prog: "
   daemon $nginxd -c ${nginx_config}
   RETVAL=$?
   echo
   [ $RETVAL = 0 ] && touch /var/lock/subsys/nginx
   return $RETVAL
}
# Stop nginx daemons functions.
stop() {
        echo -n $"Stopping $prog: "
        killproc $nginxd
        RETVAL=$?
        echo
        [ $RETVAL = 0 ] && rm -f /var/lock/subsys/nginx /var/run/nginx.pid
}
# reload nginx service functions.
reload() {
    echo -n $"Reloading $prog: "
    #kill -HUP `cat ${nginx_pid}`
    killproc $nginxd -HUP
    RETVAL=$?
    echo
}
# See how we were called.
case "$1" in
start)
        start
        ;;
stop)
        stop
        ;;
reload)
        reload
        ;;
restart)
        stop
        start
        ;;
status)
        status $prog

        RETVAL=$?
        ;;
*)
        echo $"Usage: $prog {start|stop|restart|reload|status|help}"
        exit 1
esac
exit $RETVAL
```
<br />
* 添加执行权限 ` chmod  +x  nginx`，设为服务`chkconfig  --add nginx`，即可开机自启动。