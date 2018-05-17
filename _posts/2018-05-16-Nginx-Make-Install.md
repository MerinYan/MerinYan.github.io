---
layout: default
title:  "Nginx编译安装"
date:   2018-05-16 15:29:54 +0800
---


安装编译环境
```shell
[root@tower nginx-1.14.0]# yum install -y gcc gcc-c++
[root@tower opt]# yum install -y pcre pcre-devel openssl openssl-devel zlib zlib-devel
```

```shell
[root@tower opt]# wget http://nginx.org/download/nginx-1.14.0.tar.gz
[root@tower opt]# tar zxvf nginx-1.14.0.tar.gz
[root@tower opt]# cd nginx-1.14.0/
```