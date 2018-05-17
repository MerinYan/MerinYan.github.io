---
layout: default
title:  "Ansible Tower安装"
date:   2018-05-16 22:36:01 +0800
---  


Ansible Tower 是能够帮助任何IT团队更容易使用Ansible的一个web解决方案。  

安装手册：  
[http://docs.ansible.com/ansible-tower/latest/html/quickinstall/index.html](http://docs.ansible.com/ansible-tower/latest/html/quickinstall/index.html)  
下载地址：  
[https://releases.ansible.com/ansible-tower/setup/ansible-tower-setup-latest.tar.gz](https://releases.ansible.com/ansible-tower/setup/ansible-tower-setup-latest.tar.gz)  



### 环境要求
A. 最低硬件配置  

| CPU          | Memory              | Disk                  |
|:-------------|:--------------------|:----------------------|
| 2C/20forks   | 4G/100forks         | /var 20G  ansible工作空间数据库目录 150G(最低20G) |  

B. 64位操作系统（Redhat、Centos均可）

C. Postgresql 9.6 以上

### 安装ansible
yum install ansible

### 安装ansible-tower
```shell
[root@tower opt]# wget https://releases.ansible.com/ansible-tower/setup/ansible-tower-setup-latest.tar.gz
[root@tower opt]# tar xvzf ansible-tower-setup-latest.tar.gz
[root@tower opt]# cd ansible-tower-setup-3.2.4/
[root@tower ansible-tower-setup-3.2.4]# vi inventory 
[tower]
localhost ansible_connection=local

[database]

[all:vars]
admin_password='password'

pg_host=''
pg_port=''

pg_database='awx'
pg_username='awx'
pg_password='password'

rabbitmq_port=5672
rabbitmq_vhost=tower
rabbitmq_username=tower
rabbitmq_password='password'
rabbitmq_cookie=cookiemonster

# Needs to be true for fqdns and ip addresses
rabbitmq_use_long_name=false

# Isolated Tower nodes automatically generate an RSA key for authentication;
# To disable this behavior, set this value to false
# isolated_key_generation=true

[root@tower ansible-tower-setup-3.2.4]# ./setup.sh   
```

安装完成后，将会看到安装成功的提示
![installOK]({{site.url}}/assets/images/2018-05-16-ansible-tower-install.png)

### 证书导入
浏览器打开页面[https://tower-address](https://tower-address)
![登录]({{site.url}}/assets/images/2018-05-16-ansible-tower-login.png)
输入用户名admin密码password

![上传证书]({{site.url}}/assets/images/2018-05-16-ansible-tower-license-input.png)
点击BROWSE，上传证书，同意安装协议，然后提交

![主界面]({{site.url}}/assets/images/2018-05-16-ansible-tower-dashboard.png)
这就是ansible Tower的dashboard面板

