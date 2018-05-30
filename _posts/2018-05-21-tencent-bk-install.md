---
layout: default
title:  "腾讯蓝鲸安装"
date:   2018-05-21 16:46:34 +0800
---  

蓝鲸智云配置平台(blueking cmdb)是腾讯开源的CMDB应用  
官网：	[http://bk.tencent.com](http://bk.tencent.com)  


* 机器信息：  

| IP 			| HostName	| CPU 	  | Memory		|
|:--------------|:----------|:--------|:------------|
| 192.168.1.41  | bk1.bk.com | 4C	  | 32G			|
| 192.168.1.42  | bk2.bk.com | 4C	  | 32G			|
| 192.168.1.43  | bk3.bk.com | 4C	  | 32G			|
| 192.168.1.44  | bk4.bk.com | 4C	  | 32G			|
| 192.168.1.45  | bk5.bk.com | 4C	  | 32G			|

* 中控机配置免密码登录其他主机
```shell
[root@bk1 install]# ssh-keygen -t rsa -b 2048 -N "" -f $HOME/.ssh/id_rsa
[root@bk1 install]# for ip in $(awk '{print $1}' install.config); do ssh-copy-id $ip; done
```
* 服务端所有机器和客户机配置域名解析(通过DNS或者通过hosts文件)  
```
192.168.1.43	paas.bk.com  
192.168.1.43	job.bk.com  
192.168.1.43	cmdb.bk.com  
```

* 配置时间同步
```shell
[root@bk1 install]# ansible all -a "ntpdate cn.pool.ntp.org"
```

* 关闭SELINUX，关闭防火墙
```shell
[root@gse-1 ~]# ansible all -m raw -a 'sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config'
[root@gse-1 ~]# ansible all -a "systemctl stop firewalld"
[root@gse-1 ~]# ansible all -a "systemctl disable firewalld"
```

* 配置Nginx源yum源
```shell
[root@gse-1 opt]# ansible all -m raw -a "wget  http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm -O /tmp/nginx-release-centos-7-0.el7.ngx.noarch.rpm"
[root@gse-1 opt]# ansible all -m raw -a "rpm -ivh /tmp/nginx-release-centos-7-0.el7.ngx.noarch.rpm"
```

安装文件准备
```shell
[root@bk1 opt]# cd /data/
[root@bk1 data]# tar xf bkee-global_src-1.2.303_sfcloud.tgz 
[root@bk1 data]# tar xf install_ee-master-1.3.3.tgz
[root@bk1 data]# ls -l
total 1906308
-rw-r--r--  1 root root 1932854875 May 21 13:28 bkee-global_src-1.2.303_sfcloud.tgz
drwxr-xr-x  9 root root       4096 Apr 11 22:36 install
-rw-r--r--  1 root root   19196806 May 21 13:28 install_ee-master-1.3.3.tgz
drwxrwxr-x 16 root root        255 Apr 11 22:32 src
[root@bk1 install]# cp install.config.5ip.sample install.config   #修改ip
[root@bk1 install]# vi globals.env  #修改域名即可
```

数据基础模块 开源组件补全
```shell
[root@rbtnode1 install]# cd ..
[root@rbtnode1 data]# wget http://central.maven.org/maven2/ch/qos/logback/logback-core/1.1.7/logback-core-1.1.7.jar -O src/bkdata/databus/lib/company/logback-core-1.1.7.jar
[root@rbtnode1 data]# wget http://central.maven.org/maven2/mysql/mysql-connector-java/5.1.38/mysql-connector-java-5.1.38.jar -O src/bkdata/databus/lib/company/mysql-connector-java-5.1.38.jar
[root@rbtnode1 data]# wget http://central.maven.org/maven2/com/github/dblock/waffle/waffle-jna/1.7.5/waffle-jna-1.7.5.jar -O src/bkdata/databus/lib/company/waffle-jna-1.7.5.jar
[root@rbtnode1 data]# wget http://central.maven.org/maven2/org/tukaani/xz/1.5/xz-1.5.jar -O src/bkdata/databus/lib/company/xz-1.5.jar
[root@rbtnode1 data]# wget https://pypi.python.org/packages/a5/e9/51b544da85a36a68debe7a7091f068d802fc515a3a202652828c73453cad/MySQL-python-1.2.5.zip#md5=654f75b302db6ed8dc5a898c625e030c -O src/bkdata/support-files/pkgs/MySQL-python-1.2.5.zip
[root@rbtnode1 data]# wget https://pypi.python.org/packages/0c/1c/44849e293e367a157f1ad863cee02b4b865840543254d8fae3ecdebdbdb9/uwsgi-2.0.12.tar.gz#md5=1451cab954bad0d7d7429e4d2c84b5df -O src/bkdata/support-files/pkgs/uwsgi-2.0.12.tar.gz
```

作业平台 开源组件补全
```shell
[root@rbtnode1 data]# cd src/job/job
[root@rbtnode1 job]# mkdir -p WEB-INF/lib
[root@rbtnode1 job]# cd WEB-INF/lib/
[root@rbtnode1 lib]# wget http://central.maven.org/maven2/com/rabbitmq/amqp-client/4.0.3/amqp-client-4.0.3.jar
[root@rbtnode1 lib]# wget http://central.maven.org/maven2/c3p0/c3p0/0.9.1.1/c3p0-0.9.1.1.jar
[root@rbtnode1 lib]# wget http://central.maven.org/maven2/org/eclipse/jdt/ecj/3.12.3/ecj-3.12.3.jar
[root@rbtnode1 lib]# wget http://central.maven.org/maven2/javax/servlet/javax.servlet-api/3.1.0/javax.servlet-api-3.1.0.jar
[root@rbtnode1 lib]# wget http://central.maven.org/maven2/org/jboss/marshalling/jboss-marshalling/1.3.0.CR9/jboss-marshalling-1.3.0.CR9.jar
[root@rbtnode1 lib]# wget http://central.maven.org/maven2/org/jboss/marshalling/jboss-marshalling-serial/1.3.0.CR9/jboss-marshalling-serial-1.3.0.CR9.jar
[root@rbtnode1 lib]# wget http://central.maven.org/maven2/net/sourceforge/jchardet/jchardet/1.0/jchardet-1.0.jar
[root@rbtnode1 lib]# wget http://central.maven.org/maven2/ch/qos/logback/logback-access/1.1.11/logback-access-1.1.11.jar
[root@rbtnode1 lib]# wget http://central.maven.org/maven2/ch/qos/logback/logback-classic/1.1.11/logback-classic-1.1.11.jar
[root@rbtnode1 lib]# wget http://central.maven.org/maven2/ch/qos/logback/logback-core/1.1.11/logback-core-1.1.11.jar
[root@rbtnode1 lib]# wget http://central.maven.org/maven2/mysql/mysql-connector-java/5.1.39/mysql-connector-java-5.1.39.jar
[root@rbtnode1 job]# cd ../../
[root@rbtnode1 job]# zip -u0 job-exec.war WEB-INF/lib/*.jar
  adding: amqp-client-4.0.3.jar (stored 0%)
  adding: c3p0-0.9.1.1.jar (stored 0%)
  adding: ecj-3.12.3.jar (stored 0%)
  adding: javax.servlet-api-3.1.0.jar (stored 0%)
  adding: jboss-marshalling-1.3.0.CR9.jar (stored 0%)
  adding: jboss-marshalling-serial-1.3.0.CR9.jar (stored 0%)
  adding: jchardet-1.0.jar (stored 0%)
  adding: logback-access-1.1.11.jar (stored 0%)
  adding: logback-classic-1.1.11.jar (stored 0%)
  adding: logback-core-1.1.11.jar (stored 0%)
  adding: mysql-connector-java-5.1.39.jar (stored 0%)
```



卸载蓝鲸
```
#登录paas和job所在机器，umount所有的nfs目录
[root@rbtnode1 install]# ./bkeec stop all  #停止服务 如果配置了cron，需要先bkeec clean cron
[root@rbtnode1 install]# ./bkeec clean all  #卸载所有服务

```

