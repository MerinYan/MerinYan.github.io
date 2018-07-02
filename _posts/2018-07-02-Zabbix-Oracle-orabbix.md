---
layout: default
title:  "Zabbix监控Oracle"
date:   2018-07-02 15:42:54 +0800
---

Orabbix是一个第三方的插件，用于监控Oracle数据库的一些性能指标，如DBsize、Filesize、SGA内存使用率、缓存命中率等。  
Orabbix下载地址：[http://www.smartmarmot.com/product/orabbix/download/](http://www.smartmarmot.com/product/orabbix/download/)  
Orabbix相关文档：[http://www.smartmarmot.com/wiki/index.php?title=Orabbix](http://www.smartmarmot.com/wiki/index.php?title=Orabbix)  

插件架构：Orabbix通过JDBC协议获取Oracle性能信息，然后通过Zabbix协议将信息汇总到Zabbix中。  
![架构图]({{site.url}}/assets/images/2018-07-02-Orabbix_architecture.png)  

### 客户端创建Oracle用户并授权：
```sql
CREATE USER ZABBIX
IDENTIFIED BY REDHAT
DEFAULT TABLESPACE USERS
TEMPORARY TABLESPACE TEMP
PROFILE DEFAULT
ACCOUNT UNLOCK;
GRANT ALTER SESSION TO ZABBIX;
GRANT CREATE SESSION TO ZABBIX;
GRANT CONNECT TO ZABBIX;
ALTER USER ZABBIX DEFAULT ROLE ALL;
GRANT SELECT ON V_$INSTANCE TO ZABBIX;
GRANT SELECT ON DBA_USERS TO ZABBIX;
GRANT SELECT ON V_$LOG_HISTORY TO ZABBIX;
GRANT SELECT ON V_$PARAMETER TO ZABBIX;
GRANT SELECT ON SYS.DBA_AUDIT_SESSION TO ZABBIX;
GRANT SELECT ON V_$LOCK TO ZABBIX;
GRANT SELECT ON DBA_REGISTRY TO ZABBIX;
GRANT SELECT ON V_$LIBRARYCACHE TO ZABBIX;
GRANT SELECT ON V_$SYSSTAT TO ZABBIX;
GRANT SELECT ON V_$PARAMETER TO ZABBIX;
GRANT SELECT ON V_$LATCH TO ZABBIX;
GRANT SELECT ON V_$PGASTAT TO ZABBIX;
GRANT SELECT ON V_$SGASTAT TO ZABBIX;
GRANT SELECT ON V_$LIBRARYCACHE TO ZABBIX;
GRANT SELECT ON V_$PROCESS TO ZABBIX;
GRANT SELECT ON DBA_DATA_FILES TO ZABBIX;
GRANT SELECT ON DBA_TEMP_FILES TO ZABBIX;
GRANT SELECT ON DBA_FREE_SPACE TO ZABBIX;
GRANT SELECT ON V_$SYSTEM_EVENT TO ZABBIX;
GRANT SELECT ON V_$SESSION TO ZABBIX;
GRANT SELECT ON V_$LOG TO ZABBIX;
GRANT SELECT ON V_$LOCKED_OBJECT TO ZABBIX;
GRANT SELECT ON DBA_TABLESPACES TO ZABBIX;
GRANT SELECT ON DBA_OBJECTS TO TIVOLI;
- 如果是 Oracle 11G 则需要执行:
exec dbms_network_acl_admin.create_acl(acl => 'resolve.xml',description => 'resolve acl', principal =>'ZABBIX', is_grant => true, privilege => 'resolve');
exec dbms_network_acl_admin.assign_acl(acl => 'resolve.xml', host =>'*');
commit;
```

### Zabbix Server端安装Orabbix
```shell
[root@redhat74 ~]# unzip orabbix-1.2.3.zip -d /usr/local/orabbix
[root@redhat74 ~]# cp /usr/local/orabbix/init.d/orabbix /etc/init.d/
[root@redhat74 ~]# vi /etc/init.d/orabbix 
...
orabbix=/usr/local/orabbix
...
[root@redhat74 ~]# cd /usr/local/orabbix/conf/config.props.sample
[root@redhat74 ~]# cp config.props.sample config.props
[root@redhat74 ~]# vim config.props
#comma separed list of Zabbix servers
ZabbixServerList=ZabbixServer1

ZabbixServer1.Address=192.168.226.200
ZabbixServer1.Port=10051

#pidFile
OrabbixDaemon.PidFile=./logs/orabbix.pid
#frequency of item's refresh
OrabbixDaemon.Sleep=300
#MaxThreadNumber should be >= than the number of your databases
OrabbixDaemon.MaxThreadNumber=100

#put here your databases in a comma separated list
####注意这里的名称必须和Zabbix中添加的主机item名称相同
DatabaseList=192.168.226.201

#Configuration of Connection pool
#if not specified Orabbis is going to use default values (hardcoded)
#Maximum number of active connection inside pool
DatabaseList.MaxActive=10
#The maximum number of milliseconds that the pool will wait 
#(when there are no available connections) for a connection to be returned 
#before throwing an exception, or <= 0 to wait indefinitely. 
DatabaseList.MaxWait=100
DatabaseList.MaxIdle=1

#define here your connection string for each database
192.168.226.201.Url=jdbc:oracle:thin:@192.168.226.201:1521:XE
192.168.226.201.User=ZABBIX1
192.168.226.201.Password=ZABBIX1
#Those values are optionals if not specified Orabbix is going to use the general values
192.168.226.201.MaxActive=10
192.168.226.201.MaxWait=100
192.168.226.201.MaxIdle=1
192.168.226.201.QueryListFile=./conf/query.props

```

### 导入Zabbix模板
```
从Zabbix上导入模板文件：
orabbix-1.2.3/template/Orabbix_export_full.xml
```
![添加模板]({{site.url}}/assets/images/2018-07-02-Zabbix-orabbix-importtemplate.png)

![添加模板]({{site.url}}/assets/images/2018-07-02-Zabbix-orabbix-importtemplate1.png)

### 为Oracle主机关联模板
![添加主机]({{site.url}}/assets/images/2018-07-02-Zabbix-Orabbix-addhost.png)  

![关联模板]({{site.url}}/assets/images/2018-07-02-Zabbix-orabbix-addtemplate.png)

### 监控效果图
![效果图]({{site.url}}/assets/images/2018-07-02-Zabbix-orabbix-graph1.png)

![效果图]({{site.url}}/assets/images/2018-07-02-Zabbix-orabbix-graph2.png)

### 自定义SQL监控
如果需要添加自定义SQL监控，只需要修改query.props文件即可（需要为这个用户授权）。  
如：新增下面这个SQL监控  
```sql
SQL> select age from test;
       AGE
----------
        80
```

修改配置文件/usr/local/orabbix/conf/query.props
```
[root@redhat74 opt]# vim /usr/local/orabbix/conf/query.props
...
QueryList=...,test   # 在QueryList后面添加新查询的名称
test.Query=select age from test   # 定义查询内容
test.NoDataFound=none
...
```

修改模板，添加一个指标（item）  
![效果图]({{site.url}}/assets/images/2018-07-02-Zabbix-orabbix-add-custom-key.png)
添加对应的阈值设置（trigger）  
![效果图]({{site.url}}/assets/images/2018-07-02-Zabbix-orabbix-add-custom-key1.png)
报警效果图
![效果图]({{site.url}}/assets/images/2018-07-02-Zabbix-orabbix-add-custom-key2.png)

