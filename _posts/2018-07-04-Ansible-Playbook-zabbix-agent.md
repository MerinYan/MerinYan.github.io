---
layout: default
title:  "Ansible Playbook 安装 zabbix agent"
date:   2018-07-04 17:27:40 +0800
---

说明：
由于内网访问不了zabbix yum源，所以我直接将rpm包放在了zabbix server的http目录下面，直接安装http rpm

### playbook 目录结构

```shell
[root@redhat74 ansible]# pwd
/etc/ansible
[root@redhat74 ansible]# tree roles/zabbix-agent/
roles/zabbix-agent/
└── tasks
    └── main.yml
```

### 主yaml文件
```yaml
[root@redhat74 ansible]# more zabbix-agent.yml 
- hosts: test
  remote_user: root
  roles:
    - zabbix-agent
```

### 主task内容
```yaml
[root@redhat74 ansible]# more roles/zabbix-agent/tasks/main.yml 
- name: install zabbix
  yum: name=http://192.168.226.200/zabbix-agent-3.0.19-1.el6.x86_64.rpm state=present
  become: true

- name: config zabbix_agentd.conf parameter Server
  lineinfile: path=/etc/zabbix/zabbix_agentd.conf state=present  regexp='^Server=' line='Server=192.168.226.200'
  become: true

- name: config zabbix_agentd.conf parameter ServerActive
  lineinfile: path=/etc/zabbix/zabbix_agentd.conf state=present  regexp='^ServerActive=' line='ServerActive=192.168.226.200'
  become: true

- name: restart zabbix agent
  service: name=zabbix-agent state=restarted enabled=yes
  become: true

- name: create zabbix hosts and link template
  local_action:
    module: zabbix_host
    server_url: http://192.168.226.200/zabbix/
    login_user: Admin
    login_password: zabbix
    host_name: '{{inventory_hostname}}'
    visible_name: '{{inventory_hostname}}'
    host_groups:
      - 'Linux servers'
    link_templates:
      - Template OS Linux
    status: enabled
    state: present
    interfaces:
    - type: 1
      main: 1
      useip: 1
      ip: '{{inventory_hostname}}'
      dns: ""
      port: 10050
  tags:
  - set_hosts
```

### 执行效果
```
[root@redhat74 ansible]# ansible-playbook zabbix-agent.yml                                  

PLAY [test] *************************************************************************************************

TASK [Gathering Facts] **************************************************************************************
ok: [192.168.226.201]

TASK [zabbix-agent : install zabbix] ************************************************************************
ok: [192.168.226.201]

TASK [zabbix-agent : config zabbix_agentd.conf parameter Server] ********************************************
ok: [192.168.226.201]

TASK [zabbix-agent : config zabbix_agentd.conf parameter ServerActive] **************************************
ok: [192.168.226.201]

TASK [zabbix-agent : restart zabbix agent] ******************************************************************
changed: [192.168.226.201]

TASK [zabbix-agent : create zabbix hosts and link template] *************************************************
changed: [192.168.226.201 -> localhost]

PLAY RECAP **************************************************************************************************
192.168.226.201            : ok=6    changed=2    unreachable=0    failed=0   

[root@redhat74 ansible]# 
```