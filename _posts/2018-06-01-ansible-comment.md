---
layout: default
title:  "Ansible笔记"
date:   2018-06-01 22:47:22 +0800
---



* **Linux被管机要求**
Python 版本 2.4 或以上.如果版本低于 Python 2.5 ,还需要额外安装一个模块:python-simplejson
不支持Python3，可以通过命令安装Python2
ansible myhost --sudo -m raw -a "yum install -y python2 python-simplejson"
如果被管节点上开启了SElinux,你需要安装libselinux-python

* **查看ansible模块帮助**  
ansible-doc <<模块名字>>

* **敏感数据加密**  
```shell
[root@haproxy demo]# cat default/main.yaml #源文件内容
password: 123456
[root@haproxy demo]# cat secret.txt #加密解密的密码
redhat
[root@haproxy demo]# ansible-vault encrypt default/main.yaml  #加密文件
New Vault password: 
Confirm New Vault password: 
Encryption successful
[root@haproxy demo]# cat default/main.yaml #加密后的文件
$ANSIBLE_VAULT;1.1;AES256
36393064373430643032386637366361653638646661613831303434633665303661336238383233
3563663132343464393732633736656337343439356563340a306234373730353461383437336635
34353761346430396265326537663632333234623465653131666132653334326265363233336462
3634643239323964350a636136643637383263636666383164386137313236353866346237336163
39623738363864393733323134393035326438613537663461313534616332323131
[root@haproxy demo]# ansible-vault view default/main.yaml #查看加密的文件
Vault password: 
password: 123456
[root@haproxy demo]# ansible-vault edit default/main.yaml #编辑加密的文件
```

* **在playbook中使用加密文件**  
```
	[root@haproxy demo]# ansible-playbook hello-word.yaml --vault-password-file secret.txt

	PLAY [say 'hello word'] *********************************************************

	TASK [Gathering Facts] **********************************************************
	ok: [192.168.1.107]

	TASK [echo 'hello world'] *******************************************************
	changed: [192.168.1.107]

	TASK [print sdtout] *************************************************************
	ok: [192.168.1.107] => {
	    "msg": 123456
	}

	PLAY RECAP **********************************************************************
	192.168.1.107              : ok=3    changed=1    unreachable=0    failed=0   
```

* **动态主机清单**  
动态主机清单实际就是一个脚本，这个脚本必须支持2个参数(--list 和 --host)，返回内容为json格式的host信息，执行ansible命令或playbook时必须加-i参数指定这个脚本的位置

* **常用模块** 
* yum  
[root@tower ~]# ansible web -m yum -a "name=httpd state=present"
* user  
[root@tower ~]# ansible web -m user -a "name=user1 group=root"
* git  
[root@tower ~]# ansible web -m git -a "repo=https://github.com/cakephp/cakephp.git version=3.6.0 dest=~/test/"

* **本地执行module**
一个自定义的module，调用系统nc命令检查端口是否被占用，但被管机未安装nc命令，那么可以通过在本地执行这个module。有两种方式可以实现：
1. 通过代理的方式（推荐）
- name: smoke test
  can_reach: host={{ansible_ssh_host}} port={{pla_port}}
  delegate_to: localhost
2. 通过local_action模块实现
- name: smoke test
  local_action: can_reach host={{ansible_ssh_host}} port={{pla_port}}

* **Windows被管机**
1. 管理节点（必须是Linux）安装pywinrm模块
pip install http://github.com/diyan/pywinrm/archive/master.zip#egg=pywinrm
2. Windows被管节点必须是PowerShell 3.0以上（Windows 7 SP1 ,Windows Server 2008 SP1以上）
在Powershell中执行https://github.com/cchurch/ansible/blob/devel/examples/scripts/upgrade_to_ps3.ps1
3. Windows被管节点需要打开winrm服务
在Powershell中执行https://github.com/ansible/ansible/blob/devel/examples/scripts/ConfigureRemotingForAnsible.ps1


winrm远程连接测试
winrs -r:http://192.168.1.81:5985/wsman -u:administrator -p:'1qaz@WSX' ipconfig

hosts文件样例
```ini
[root@bkcmdb ~]# cat /etc/ansible/hosts 
[win]
192.168.1.81 ansible_ssh_host=192.168.1.81 ansible_ssh_port=5985 ansible_ssh_user='administrator' ansible_ssh_pass='1qaz@WSX' ansible_connection=w
inrm ansible_winrm_server_cert_validation=ignore
[root@bkcmdb ~]# ansible all -m win_ping
192.168.1.81 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
```

* **连接VCenter 通过template创建虚拟机**  

```yml
# more create_vm.yaml 
---
- name: demo
  hosts: all
  gather_facts: no
  tasks:
    - name: Create a VM from a template
      vmware_guest:
        hostname: vcenter.infodt.com    # VCenter地址
        username: administrator@infodt.com  # VCenter账号
        password: "1qazXDR%"  # VCenter密码
        validate_certs: no  # 不验证VCenter SSL证书有效性
        datacenter: Datacenter1  # 新虚拟机对应的datacenter名称
        name: testvm_1  # 新虚拟机名称
        folder: /Datacenter1/vm  # 新虚拟机所在的目录
        esxi_hostname: 192.168.1.58  # 新虚拟机对应的ESXi名称
        state: poweredon  # 克隆完成后打开虚拟机
        template: centos-temp  # 虚拟机模板
        disk:
        - size_gb: 10
          type: thin
          datastore: datastore1
        hardware:
          memory_mb: 800
          num_cpus: 1
        networks:
        - name: VM Network
          ip: 192.168.1.61
          netmask: 255.255.255.0
          gateway: 192.168.1.1
        customization:
          autologon: yes
          dns_servers:
            - 192.168.1.30
            - 8.8.8.8
          password: new_vm_password
          hostname: vm2  # 新虚拟机的主机名
          domain: infodt.com  # 新虚拟机的域名
        wait_for_ip_address: yes
      delegate_to: localhost
      register: deploy
# more remove_vm.yaml 
---
- name: demo
  hosts: all
  gather_facts: no
  tasks:
    - name: Remove a VM by name
      vmware_guest:
        hostname: vcenter.infodt.com
        username: administrator@infodt.com
        password: "1qazXDR%"
        validate_certs: no
        force: true
        folder: /Datacenter1/vm
        name: testvm_1
        state: absent
      delegate_to: localhost
```
```shell
MBookPro:ansible mervin$ ansible-playbook -i hosts create_vm.yaml 

PLAY [demo] ****************************************************************************

TASK [Create a VM from a template] *****************************************************
changed: [192.168.1.59 -> localhost]

PLAY RECAP *****************************************************************************
192.168.1.59               : ok=1    changed=1    unreachable=0    failed=0   

MBookPro:ansible mervin$ ansible-playbook -i hosts remove_vm.yaml 

PLAY [demo] ****************************************************************************

TASK [Remove a VM by name] *************************************************************
changed: [192.168.1.59 -> localhost]

PLAY RECAP *****************************************************************************
192.168.1.59               : ok=1    changed=1    unreachable=0    failed=0   
```


* **常见问题1**
```shell
[root@tower ~]# ansible all -m ping
[WARNING]: Found both group and host with same name: localhost
localhost | FAILED! => {
    "msg": "Using a SSH password instead of a key is not possible because Host Key 
    checking is enabled and sshpass does not support this.  Please add this host's 
    fingerprint to your known_hosts file to manage this host."
}
```
解决办法：禁用host_key 检查
```shell
[root@tower ~]# vi /etc/ansible/ansible.cfg 
host_key_checking = False
```

* **常见故障2**
管理Windows报错 the specified credentials were rejected by the server  
```shell
[root@bkcmdb ~]# ansible all -m win_ping
192.168.1.81 | UNREACHABLE! => {
    "changed": false, 
    "msg": "plaintext: the specified credentials were rejected by the server", 
    "unreachable": true
}
```

解决办法,允许Basic认证方式,允许未加密方式访问
```
Set-Item -Path WSMan:\localhost\Service\Auth\Basic -Value $true
winrm set winrm/config/service '@{AllowUnencrypted="true"}'
```

* **Ansible-playbook 注册参数**  

```bash
[root@bkcmdb ansible]# ansible win -m win_service -a "name=pla_mgmt"   # 获取一个Windows服务状态                              
192.168.1.81 | SUCCESS => {
    "can_pause_and_continue": false, 
    "changed": false, 
    "depended_by": [], 
    "dependencies": [], 
    "description": "", 
    "desktop_interact": false, 
    "display_name": "pla_mgmt", 
    "exists": true, 
    "name": "pla_mgmt", 
    "path": "C:\\PLA\\mgmt\\nssm.exe", 
    "start_mode": "auto", 
    "state": "stopped", 
    "username": "LocalSystem"
}
[root@bkcmdb ansible]# more roles/pla_mgmt/tasks/main.yml       
- name: get a service status
  win_service: name=pla_mgmt
  register: info    # 把查询结果保存到info中

- name: print service status
  debug: msg=服务名称为：{{ info.display_name}}  # 调用info中的子变量 display_name，msg可以不用引号

- name: print status
  debug: msg="服务已经存在"
  when: info.exists    # 调用info中的子变量 exists，调用的时候不需要{}
[root@bkcmdb ansible]# ansible-playbook pla_mgmt.yml 

PLAY [win] **********************************************************************

TASK [pla_mgmt : get a service status] ******************************************
ok: [192.168.1.81]

TASK [pla_mgmt : print service status] ******************************************
ok: [192.168.1.81] => {
    "msg": "服务名称为：pla_mgmt"
}

TASK [pla_mgmt : print status] **************************************************
ok: [192.168.1.81] => {
    "msg": "服务已经存在"
}

PLAY RECAP **********************************************************************
192.168.1.81               : ok=3    changed=0    unreachable=0    failed=0   
```