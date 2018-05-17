---
layout: default
title:  "Ansible Playbook编译安装Nginx"
date:   2018-05-17 14:24:51 +0800
---

蓝鲸智云配置平台(blueking cmdb)是腾讯开源的CMDB应用  
官网：	[http://bk.tencent.com](http://bk.tencent.com)  
GitHub：	[https://github.com/Tencent/bk-cmdb](https://github.com/Tencent/bk-cmdb)

### playbook 目录结构

```shell
[root@tower projects]# pwd
/var/lib/awx/projects
[root@tower projects]# tree install_nginx/
install_nginx/
├── installnginx.yaml
└── roles
    └── nginx
        ├── defaults
        ├── files
        │   ├── nginx
        │   ├── nginx-1.14.0.tar.gz
        │   └── nginx.conf
        ├── handlers
        ├── meta
        ├── tasks
        │   └── main.yml
        ├── templates
        └── vars
```

### 主yaml文件
```yaml
[root@tower projects]# more install_nginx/installnginx.yaml 
- hosts: all
  remote_user: root
  roles:
    - nginx
```

### 主task内容
```yaml
[root@tower projects]# more install_nginx/roles/nginx/tasks/main.yml 
- name: copy nginx package to remote host  
  copy: src=nginx-1.14.0.tar.gz  dest=/tmp/nginx-1.14.0.tar.gz
  tags: cppkg
- name: untar nginx
  shell: cd /tmp;tar -zxvf nginx-1.14.0.tar.gz
- name: install rpm pakger
  yum: name={{ item }} state=latest
  with_items:
    - openssl-devel
    - pcre-devel
    - gcc
- name: install nginx
  shell: cd /tmp/nginx-1.14.0;./configure --user=nginx --group=nginx --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-pcre; make&& make install
- name: copy conf file nginx.conf          
  copy: src=nginx.conf dest=/usr/local/nginx/conf/nginx.conf
  tags: ngxconf
- name: copy nginx service file
  copy: src=nginx dest=/etc/init.d/nginx mode=755
- name: create user
  user: name="nginx"
- name: start nginx service
  shell: service nginx start
```

### 系统服务脚本（install_nginx/roles/nginx/files/nginx）
```
#! /bin/bash
# chkconfig: - 85 15
PATH=/usr/local/nginx
DESC="nginx daemon"
NAME=nginx
DAEMON=$PATH/sbin/$NAME
CONFIGFILE=$PATH/conf/$NAME.conf
PIDFILE=$PATH/logs/$NAME.pid
SCRIPTNAME=/etc/init.d/$NAME
set -e
[ -x "$DAEMON" ] || exit 0
do_start() {
  $DAEMON -c $CONFIGFILE || echo -n "nginx already running"
}
do_stop() {
  $DAEMON -s stop || echo -n "nginx not running"
}
do_reload() {
  $DAEMON -s reload || echo -n "nginx can't reload"
}
case "$1" in
start)
echo -n "Starting $DESC: $NAME"
do_start
echo "."
;;
stop)
echo -n "Stopping $DESC: $NAME"
do_stop
echo "."
;;
reload|graceful)
echo -n "Reloading $DESC configuration..."
do_reload
echo "."
;;
restart)
echo -n "Restarting $DESC: $NAME"
do_stop
do_start
echo "."
;;
*)
echo "Usage: $SCRIPTNAME {start|stop|reload|restart}" >&2
exit 3
;;
esac
exit

```

### 执行效果
```
[root@tower projects]# ansible-playbook install_nginx/installnginx.yaml 

PLAY [all] ********************************************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************************************
ok: [192.168.1.90]

TASK [nginx : copy nginx package to remote host] ******************************************************************************************************************************************
ok: [192.168.1.90]

TASK [nginx : untar nginx] ****************************************************************************************************************************************************************
changed: [192.168.1.90]

TASK [nginx : install rpm pakger] *********************************************************************************************************************************************************
ok: [192.168.1.90] => (item=[u'openssl-devel', u'pcre-devel', u'gcc'])

TASK [nginx : install nginx] **************************************************************************************************************************************************************
changed: [192.168.1.90]

TASK [nginx : copy conf file nginx.conf] **************************************************************************************************************************************************
ok: [192.168.1.90]

TASK [nginx : copy nginx service file] ****************************************************************************************************************************************************
ok: [192.168.1.90]

TASK [nginx : create user] ****************************************************************************************************************************************************************
ok: [192.168.1.90]

TASK [nginx : start nginx service] ********************************************************************************************************************************************************
 [WARNING]: Consider using service module rather than running service

changed: [192.168.1.90]

PLAY RECAP ********************************************************************************************************************************************************************************
192.168.1.90               : ok=9    changed=3    unreachable=0    failed=0   

[root@tower projects]# 
```