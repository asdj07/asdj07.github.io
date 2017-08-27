---
layout: post
title: Linux 下 的ssh免密登录
date: 2017-06-08 15:37:27
categories: 开发环境
tags:
    - linux
    - ssh
---

##### 在``` /root/.ssh/```目录下：
```sh
[root@localhost .ssh]# tree -a
.
├── authorized_keys
├── id_dsa
├── id_dsa.pub
├── id_rsa
├── id_rsa_2048_9417.pub
├── id_rsa.pub
└── known_hosts
```

<!-- more -->

* 若没有则创建.ssh文件，则创建并设置权限：```$ chmod  700  ~/.ssh```
<br />
* 在``.ssh``目录下生成公钥私钥：```$ ssh-keygen -t rsa```
其中```id_dsa,id_rsa```是私钥，```id_rsa.pub,id_dsa.pub```是公钥
<br />
* ```authorized_keys```文件存放其他主机的公钥，其他主机即可ssh登录该机，此文件的权限：```$ chmod 644 ~/.ssh/authorized_keys ```

* ```know_hosts```记录主机登陆过的其他主机的公钥信息

##### 示例A和B免密登录C
* 分别在A和B下生成私钥密钥对，执行命令```$ ssh-keygen -t rsa```，生成```id_rsa```和```id_rsa.pub```
* 分别将生成的A的id_rsa.pub和B的id_rsa.pub内容追加到C主机的authorized_keys中
A和B可以通过```$ ssh C ```登录，```exit```退出。
  ```
$ cat authorized_keys
ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAvqtPwT7kjfKzycR7r0FLe+UgxOSoeOW9EVb6eUb9MsO5IHvlVBKJK6F62dc5BNgemkwR8/UUDQ6tK4DMWipHI+t8naxgyXl9Kdc7oh78c/ADW1svBkrV3qOfxey/z+8ykN+kCgk7q65NytllpQH3FAi7b/0mO3cAEQWGSAC5wSG7XOamMmL4CLjhhLGwLwIAni50nOTBVVBjrXVn10EW4Bwcv+tH7KAIlZ+kZuatOUMIYyuBWleBokJzgQm2joQfe9RiO2Ayja6O4CpJSj0g3Efkb0bdxaOxYrgigp/0w400DQfjQil6I9lS1jQGazu8WIVJjSQU4HEFw5x4ocuzQQ== root@localhost.localdomain
ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAuiOuiFKim4VV74rkxYBqsmfEAbb5oMmsp/u36N/eO0o5sPEf/a6ZwiJAPT/XAAaw+ICVxZraJz+2lNyzAIvvBcqiL6gQev5MdOXtf+341iUhH1Ut8KWHxrGckbZc8+deDifwR6gUlv65wx+YWlAcp/emkanhiom0Y6+l7Ol3hb7+Ae8/uQow0dKof81xVDC3XZDH+qbWpGtBi9jlWSnwngedy0CPNL/b2orJ+XNEniZ3gCPif4uMSdiNmkXizz9Pqe00uXBAfxZYtMGfvW8nkwUKO+OkDqTAJnDHiCVJZV/nGB1AHIR6Lq6Yia3yu5zbCBJgX27aKRxHdvHfzDwmTw== root@localhost.localdomain
  ```
<br />

**总结：** *要想免密登录哪一台主机就需要自己生成一个公钥私钥对并将公钥内容追加到那一台主机的authorized_keys内容中*
