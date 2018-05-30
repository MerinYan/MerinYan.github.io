---
layout: default
title:  "RabbitMQ安装"
date:   2018-05-23 21:27:20 +0800
---

官网: [http://www.rabbitmq.com/install-rpm.html](http://www.rabbitmq.com/install-rpm.html)  


### erlang 安装:
```shell
[root@rbtnode1 ~]# vi /etc/yum.repos.d/rabbitmq-erlang.repo
[rabbitmq-erlang]
name=rabbitmq-erlang
baseurl=https://dl.bintray.com/rabbitmq/rpm/erlang/20/el/7
gpgcheck=1
gpgkey=https://dl.bintray.com/rabbitmq/Keys/rabbitmq-release-signing-key.asc
repo_gpgcheck=0
enabled=1
[root@rbtnode1 ~]# yum -y install erlang
```

### Rabbitmq 安装配置

#### 1. Rabbitmq 安装:
```shell
[root@rbtnode1 ~]# rpm --import https://dl.bintray.com/rabbitmq/Keys/rabbitmq-release-signing-key.asc
[root@rbtnode1 ~]# wget https://dl.bintray.com/rabbitmq/all/rabbitmq-server/3.7.5/rabbitmq-server-3.7.5-1.el7.noarch.rpm
[root@rbtnode1 ~]# yum -y install rabbitmq-server-3.7.5-1.el7.noarch.rpm
```

2. 服务启停：
```shell
[root@rbtnode1 ~]# /sbin/service rabbitmq-server start
[root@rbtnode1 ~]# /sbin/service rabbitmq-server stop
```

3. 创建用户与授权
```shell
[root@rbtnode1 ~]# rabbitmqctl add_user admin 123456
Adding user "admin" ...
[root@rbtnode1 ~]# rabbitmqctl set_user_tags admin administrator
Setting tags for user "admin" to [administrator] ...
```


4. 启用WEB管理界面
```shell
[root@rbtnode1 ~]# rabbitmq-plugins enable rabbitmq_management
The following plugins have been configured:
  rabbitmq_management
  rabbitmq_management_agent
  rabbitmq_web_dispatch
Applying plugin configuration to rabbit@rbtnode1...
The following plugins have been enabled:
  rabbitmq_management
  rabbitmq_management_agent
  rabbitmq_web_dispatch
started 3 plugins.
```  

5. 登录WEB管理端，为用户授权
[http://192.168.1.41:15672](http://192.168.1.41:15672)  
![rabbitmq]({{site.url}}/assets/images/2018-05-23-rabbitmq.jpg)


## Python Demo
1. 安装Python环境:
```shell
[root@rbtnode1 ~]# yum install -y python
[root@rbtnode1 ~]# yum -y install epel-release
[root@rbtnode1 ~]# yum -y install python-pip
[root@rbtnode1 ~]# pip install virtualenv
[root@rbtnode1 ~]# virtualenv --no-site-packages venv
[root@rbtnode1 ~]# source venv/bin/activate
(venv) [root@gse-2 ~]# pip install pika
```

2. 消息发送者  
```python
[root@rbtnode1 ~]# vi send.py
#!/usr/bin/env python
import pika
user_pwd = pika.PlainCredentials('admin', '123456')
connection = pika.BlockingConnection(pika.ConnectionParameters(host='localhost',credentials=user_pwd))
channel = connection.channel()
channel.queue_declare(queue='hello')
channel.basic_publish(exchange='',
                      routing_key='hello',
                      body='Hello World!')
print(" [x] Sent 'Hello World!'")
connection.close()
```

3. 消息接收者  
```python
[root@rbtnode1 ~]# vi receive.py
#!/usr/bin/env python
import pika
user_pwd = pika.PlainCredentials('admin', '123456')
connection = pika.BlockingConnection(pika.ConnectionParameters(host='localhost',credentials=user_pwd))
channel = connection.channel()
channel.queue_declare(queue='hello')
def callback(ch, method, properties, body):
    print(" [x] Received %r" % body)
channel.basic_consume(callback,
                      queue='hello',
                      no_ack=True)
print(' [*] Waiting for messages. To exit press CTRL+C')
channel.start_consuming()
```

4. 发送接收测试
```shell
(venv) [root@rbtnode1 ~]# python send.py    #启动发送程序，发送一条消息
 [x] Sent 'Hello World!'
(venv) [root@rbtnode1 ~]# python receive.py     #启动接收程序，将受到这条消息
 [*] Waiting for messages. To exit press CTRL+C
 [x] Received 'Hello World!'
```

## 集群配置

本测试集群一共3台机器，通过hosts文件解析每台机器
```shell
[root@rabbit1 ~]# more /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.1.41 rabbit1
192.168.1.42 rabbit2
192.168.1.43 rabbit3
```

1. 在每台服务器上安装Rabbitmq，启动服务，查看服务运行状态
```
[root@rabbit1 ~]# systemctl start rabbitmq-server.service
[root@rabbit1 ~]# rabbitmqctl cluster_status 
```

2. 设置不同节点间同一认证的Erlang Cookie  
```
[root@rabbit1 ~]# more  /var/lib/rabbitmq/.erlang.cookie 
ILGLLRDTETMESQPRNXBD
...
[root@rabbit2 ~]# rabbitmqctl stop
[root@rabbit2 ~]# echo 'ILGLLRDTETMESQPRNXBD' >  /var/lib/rabbitmq/.erlang.cookie
[root@rabbit2 ~]# systemctl start rabbitmq-server.service
...
[root@rabbit3 ~]# rabbitmqctl stop
[root@rabbit3 ~]# echo 'ILGLLRDTETMESQPRNXBD' >  /var/lib/rabbitmq/.erlang.cookie
[root@rabbit3 ~]# systemctl start rabbitmq-server.service
```

3. 将rabbit2、rabbit3加入到集群中
```
[root@rabbit2 ~]# rabbitmqctl stop
[root@rabbit2 ~]# rabbitmqctl join_cluster rabbit@rabbit1
[root@rabbit2 ~]# rabbitmqctl start_app
...
[root@rabbit3 ~]# rabbitmqctl stop
[root@rabbit3 ~]# rabbitmqctl join_cluster rabbit@rabbit1
[root@rabbit3 ~]# rabbitmqctl start_app
```
4. 设置集群的模式为HA
```
[root@rabbit3 ~]# rabbitmqctl set_policy ha-all '^' '{"ha-mode": "all"}'  #在任意一台服务器执行
```

## HAProxy 配置
#### 1. 安装haproxy  
```
[root@localhost ~]# yum install haproxy
[root@localhost ~]# more /etc/haproxy/haproxy.cfg
#---------------------------------------------------------------------
# Global settings [默认内容]
#---------------------------------------------------------------------
global
    log         127.0.0.1 local2
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon
    stats socket /var/lib/haproxy/stats
#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block [默认内容]
#---------------------------------------------------------------------
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000
#---------------------------------------------------------------------
# Haproxy 统计页面 [新增加的内容]
#---------------------------------------------------------------------
listen http_front
        bind 0.0.0.0:1080           #监听端口  
        mode http
        stats refresh 30s           #统计页面自动刷新时间  
        stats uri /haproxy?stats    #统计页面url  
        stats realm Haproxy Manager #统计页面密码框上提示文本  
        stats auth admin:admin      #统计页面用户名和密码设置  
        stats hide-version          #隐藏统计页面上HAProxy的版本信息
#---------------------------------------------------------------------
# RabbitMQ 管理界面 [新增加的内容]
#---------------------------------------------------------------------
listen rabbitmq_admin
    bind 0.0.0.0:8004
    server node1 192.168.1.41:15672
    server node2 192.168.1.42:15672
    server node3 192.168.1.43:15672
#---------------------------------------------------------------------
# RabbitMQ 集群 [新增加的内容]
#---------------------------------------------------------------------
listen rabbitmq_cluster
    bind 0.0.0.0:5672
    option tcplog
    mode tcp
    timeout client  3h
    timeout server  3h
    option          clitcpka
    balance roundrobin      #负载均衡算法（roundrobin 轮询 source 保存session值）
    server   node1 192.168.1.41:5672 check inter 5s rise 2 fall 3   
    server   node2 192.168.1.42:5672 check inter 5s rise 2 fall 3
    server   node3 192.168.1.43:5672 check inter 5s rise 2 fall 3
    #check inter 2000 是检测心跳频率，rise 2是2次正确认为服务器可用，fall 3是3次失败认为服务器不可用
[root@localhost ~]# service haproxy restart     
Stopping haproxy:                                          [  OK  ]
Starting haproxy:                                          [  OK  ]
```

#### 2. 访问测试  

##### 2.1. 浏览器访问HAProxy 统计页面：  
[http://192.168.1.47:1080/haproxy?stats](http://192.168.1.47:1080/haproxy?stats)  

![rabbitmq]({{site.url}}/assets/images/2018-05-23-rabbitmq-ha-status.png)

##### 2.2. 浏览器访问Rabbitmq WEB管理界面：  
[http://192.168.1.47:8004](http://192.168.1.47:8004/)
![rabbitmq]({{site.url}}/assets/images/2018-05-23-rabbitmq-status.png)

##### 2.3. Python 发送接收消息测试：  
将脚本send.py 和 receive.py中的localhost更换成HAProxy的IP
```
(venv) [root@rabbit1 ~]# python send.py 
 [x] Sent 'Hello World!'
(venv) [root@rabbit2 ~]# python receive.py 
 [*] Waiting for messages. To exit press CTRL+C
 [x] Received 'Hello World!'
```

## 常见故障

1. 加入集群失败，报错如下：
```
[root@rbtnode2 ~]# rabbitmqctl join_cluster rabbit@rbtnode1
Clustering node rabbit@rbtnode2 with rabbit@rbtnode1
Error: unable to perform an operation on node 'rabbit@rbtnode1'. Please see diagnostics information and suggestions below.
...
Most common reasons for this are:
...
 * Target node is unreachable (e.g. due to hostname resolution, TCP connection or firewall issues)
 * CLI tool fails to authenticate with the server (e.g. due to CLI tool's Erlang cookie not matching that of the server)
 * Target node is not running
...
In addition to the diagnostics info below:
...
 * See the CLI, clustering and networking guides on http://rabbitmq.com/documentation.html to learn more
 * Consult server logs on node rabbit@rbtnode1
...
DIAGNOSTICS
===========
...
attempted to contact: [rabbit@rbtnode1]
...
rabbit@rbtnode1:
  * connected to epmd (port 4369) on rbtnode1
  * epmd reports node 'rabbit' uses port 25672 for inter-node and CLI tool traffic 
  * TCP connection succeeded but Erlang distribution failed 
...
  * Authentication failed (rejected by the remote node), please check the Erlang cookie
...
...
Current node details:
 * node name: rabbitmqcli92@rbtnode2
 * effective user's home directory: /var/lib/rabbitmq
 * Erlang cookie hash: C+sb0Iaw+WgDlc6z98NMQQ==
```
可能的原因:
    * 防火墙问题（此节点连接不上其他节点）  
    * Rabbitmq服务异常（其他节点Rabbitmq服务没有正常启动）  
    * erlang cookie文件内容不一致（所有节点的/var/lib/rabbitmq/.erlang.cookie文件内容必须相同）  
