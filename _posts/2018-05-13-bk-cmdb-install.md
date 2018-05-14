---
layout: default
title:  "腾讯蓝鲸CMDB安装"
date:   2018-05-14 16:34:57 +0800
---


## zookeeper 安装:

```
[root@bkcmdb medias]# tar zxvf zookeeper-3.4.11.tar.gz        
[root@bkcmdb medias]# mv zookeeper-3.4.11 /usr/local/zookeeper
[root@bkcmdb medias]# cd  /usr/local/zookeeper/
[root@bkcmdb zookeeper]# cp conf/zoo_sample.cfg conf/zoo.cfg
[root@bkcmdb zookeeper]# vi conf/zoo.cfg    修改其中的dataDir位置
[root@bkcmdb zookeeper]# bin/zkServer.sh start    启动服务
```


## redis 安装:
```
[root@bkcmdb medias]# tar zxvf redis-3.2.11.tar.gz 
[root@bkcmdb medias]# cd redis-3.2.11 
[root@bkcmdb redis]# make MALLOC=libc
[root@bkcmdb redis]# make install
[root@bkcmdb redis]# cd src
[root@bkcmdb src]# mv mkreleasehdr.sh redis-benchmark redis-check-aof redis-check-rdb redis-cli redis-sentinel redis-server redis-trib.rb /usr/local/redis/bin
[root@bkcmdb redis]# vi /usr/local/redis/etc/redis.conf     将daemonize 改为 yes ，后台启动redis
[root@bkcmdb redis]# /usr/local/redis/bin/redis-server /usr/local/redis/etc/redis.conf    启动redis
```

## mongod 安装:
```
[root@bkcmdb bin]# tar zxvf mongodb-linux-x86_64-rhel70-2.8.0-rc5.tgz 
[root@bkcmdb bin]# mv mongodb-linux-x86_64-rhel70-2.8.0-rc5 /usr/local/mongodb
[root@bkcmdb bin]# cd /usr/local/mongodb/bin/
[root@bkcmdb bin]# ./mongod --dbpath=/data/db --syslog --fork
```
	
## redis打开认证，且需要监听0.0.0.0：
```sql
修改/usr/local/redis/etc/redis.conf
取消注释requirepass foobared
将bind改为 bind 0.0.0.0
停止服务
	[root@bkcmdb bin]# ./redis-cli shutdown
启动服务
	[root@bkcmdb bin]# /usr/local/redis/bin/redis-server /usr/local/redis/etc/redis.conf
设置密码
	[root@bkcmdb bin]# /usr/local/bin/redis-cli -h 127.0.0.1 -p 6379 -a foobared
	127.0.0.1:6379> config set requirepass "foobared"
	OK
	127.0.0.1:6379> CONFIG GET requirepass
	1) "requirepass"
	2) "foobared"
	127.0.0.1:6379> quit
```

## MongoDB创建数据库和账号
```sql
[root@bkcmdb bin]# /usr/local/mongodb/bin/mongo
MongoDB shell version: 2.8.0-rc5
connecting to: test
Welcome to the MongoDB shell.
For interactive help, type "help".
For more comprehensive documentation, see
        http://docs.mongodb.org/
Questions? Try the support group
        http://groups.google.com/group/mongodb-user
Server has startup warnings: 
2018-05-08T03:50:16.182-0400 I CONTROL  [initandlisten] ** WARNING: You are running this process as the root user, which is not recommended.
2018-05-08T03:50:16.182-0400 I CONTROL  [initandlisten] 
2018-05-08T03:50:16.183-0400 I CONTROL  [initandlisten] 
2018-05-08T03:50:16.183-0400 I CONTROL  [initandlisten] ** WARNING: /sys/kernel/mm/transparent_hugepage/enabled is 'always'.
2018-05-08T03:50:16.183-0400 I CONTROL  [initandlisten] **        We suggest setting it to 'never'
2018-05-08T03:50:16.183-0400 I CONTROL  [initandlisten] 
2018-05-08T03:50:16.200-0400 I STORAGE  [initandlisten] 
2018-05-08T03:50:16.200-0400 I STORAGE  [initandlisten] ** WARNING: Readahead for /data/db is set to 4096KB
2018-05-08T03:50:16.200-0400 I STORAGE  [initandlisten] **          We suggest setting it to 256KB (512 sectors) or less
2018-05-08T03:50:16.200-0400 I STORAGE  [initandlisten] **          http://dochub.mongodb.org/core/readahead
> use cmdb
switched to db cmdb
> db.createUser({user: "cc",pwd: "cc",roles: [ { role: "readWrite", db: "cmdb" } ]})
Successfully added user: {
        "user" : "cc",
        "roles" : [
                {
                        "role" : "readWrite",
                        "db" : "cmdb"
                }
        ]
}
```
	
## cmdb配置：
```
[root@bkcmdb medias]# tar zxvf cmdb-3.0.6.tar.gz -C /data/
[root@bkcmdb medias]# cd /data/cmdb/
[root@bkcmdb cmdb]# python init.py --discovery 192.168.1.54:2181 --database cmdb --redis_ip 192.168.1.54 --redis_port 6379 --redis_pass foobared --mongo_ip 192.168.1.54 --mongo_port 27017 --mongo_user cc --mongo_pass cc --blueking_cmdb_url http://192.168.1.54:8083 --listen_port 8083
[root@bkcmdb cmdb]# ./start.sh     启动服务
[root@bkcmdb cmdb]# bash ./init_db.sh    初始化数据库
```

## 访问地址测试：
	http://192.168.1.54:8083/
	
## API调用：
### 添加新主机：
```shell
curl -l -H "Content-type: application/json" -H "HTTP_BLUEKING_SUPPLIER_ID: 0" -H "BK_USER: 0" -X POST -d '{ "host_info":{ "0":{ "bk_host_innerip":"127.0.0.1", "import_from":"excel", "bk_cloud_id":0 } }, "bk_supplier_id":0 }' http://192.168.1.54:8080/api/v3/hosts/add
```
		
### 查询主机信息：
```shell
curl -l -H "Content-type: application/json" -H "HTTP_BLUEKING_SUPPLIER_ID: 0" -H "BK_USER: 0" -X POST -d '{ "page":{ "start":0, "limit":2, "sort":"bk_host_name" }, "pattern":""}' http://192.168.1.54:8080/api/v3/hosts/search
```
返回结果：
```json
{
        "result": true,
        "bk_error_code": 0,
        "bk_error_msg": "success",
        "data": {
                "count": 6,
                "info": [{
                        "biz": {},
                        "host": {
                                "bk_asset_id": "",
                                "bk_bak_operator": null,
                                "bk_cloud_id": [{
                                        "id": "0",
                                        "bk_obj_id": "plat",
                                        "bk_obj_icon": "",
                                        "bk_inst_id": 0,
                                        "bk_obj_name": "",
                                        "bk_inst_name": "default area"
                                }],
                                "bk_comment": "",
                                "bk_cpu": null,
                                "bk_cpu_mhz": null,
                                "bk_cpu_module": "",
                                "bk_disk": null,
                                "bk_host_id": 6,
                                "bk_host_innerip": "127.0.0.1",
                                "bk_host_name": "",
                                "bk_host_outerip": "",
                                "bk_mac": "",
                                "bk_mem": null,
                                "bk_os_bit": "",
                                "bk_os_name": "",
                                "bk_os_type": null,
                                "bk_os_version": "",
                                "bk_outer_mac": "",
                                "bk_service_term": null,
                                "bk_sla": null,
                                "bk_sn": "",
                                "create_time": "2018-05-09T22:27:55.603-04:00",
                                "import_from": "excel",
                                "operator": null
                        },
                        "module": {},
                        "set": {}
                }, {
                        "biz": {},
                        "host": {
                                "bk_asset_id": "",
                                "bk_bak_operator": null,
                                "bk_cloud_id": [{
                                        "id": "0",
                                        "bk_obj_id": "plat",
                                        "bk_obj_icon": "",
                                        "bk_inst_id": 0,
                                        "bk_obj_name": "",
                                        "bk_inst_name": "default area"
                                }],
                                "bk_comment": "",
                                "bk_cpu": null,
                                "bk_cpu_mhz": null,
                                "bk_cpu_module": "",
                                "bk_disk": null,
                                "bk_host_id": 1,
                                "bk_host_innerip": "192.168.1.55",
                                "bk_host_name": "bk-host1",
                                "bk_host_outerip": "",
                                "bk_mac": "",
                                "bk_mem": null,
                                "bk_os_bit": "",
                                "bk_os_name": "",
                                "bk_os_type": null,
                                "bk_os_version": "",
                                "bk_outer_mac": "",
                                "bk_service_term": null,
                                "bk_sla": null,
                                "bk_sn": "",
                                "create_time": "2018-05-08T05:22:09.933-04:00",
                                "import_from": "excel",
                                "operator": null
                        },
                        "module": {},
                        "set": {}
                }]
        }
}
```
	
### 查询业务：
```shell
curl -l -H "Content-type: application/json" -H "HTTP_BLUEKING_SUPPLIER_ID: 0" -H "BK_USER: 0" -X POST -d '{"page": {"start": 0,"limit": 10,"sort": ""},"fields": [],"condition": {}}' http://192.168.1.54:8080/api/v3/biz/search/0/
```
		
返回：
```json
{
        "result": true,
        "bk_error_code": 0,
        "bk_error_msg": "success",
        "data": {
                "count": 2,
                "info": [{
                        "bk_biz_developer": "",
                        "bk_biz_id": 2,
                        "bk_biz_maintainer": "admin",
                        "bk_biz_name": "蓝鲸",
                        "bk_biz_productor": "",
                        "bk_biz_tester": "",
                        "bk_supplier_account": "0",
                        "bk_supplier_id": 0,
                        "create_time": "2018-05-08T04:44:08.743-04:00",
                        "default": 0,
                        "last_time": "2018-05-08T04:44:08.743-04:00",
                        "life_cycle": "测试中",
                        "operator": ""
                }, {
                        "bk_biz_developer": "admin",
                        "bk_biz_id": 3,
                        "bk_biz_maintainer": "admin",
                        "bk_biz_name": "aaaaa",
                        "bk_biz_productor": "admin",
                        "bk_biz_tester": "admin",
                        "bk_supplier_account": "0",
                        "bk_supplier_id": 0,
                        "create_time": "2018-05-09T22:48:48.711-04:00",
                        "default": 0,
                        "last_time": "2018-05-09T22:48:48.711-04:00",
                        "life_cycle": "已上线",
                        "operator": "admin"
                }]
        }
}
```
