---
layout: default
title:  "RabbitMQ安装"
date:   2018-05-23 21:27:20 +0800
---

官网: [http://www.rabbitmq.com/install-rpm.html](http://www.rabbitmq.com/install-rpm.html)  


## erlang 安装:
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

## Rabbitmq 安装配置

1. Rabbitmq 安装:
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

4. 最终测试
```shell
(venv) [root@rbtnode1 ~]# python send.py    #启动发送程序，发送一条消息
 [x] Sent 'Hello World!'
(venv) [root@rbtnode1 ~]# python receive.py     #启动接收程序，将受到这条消息
 [*] Waiting for messages. To exit press CTRL+C
 [x] Received 'Hello World!'
```
