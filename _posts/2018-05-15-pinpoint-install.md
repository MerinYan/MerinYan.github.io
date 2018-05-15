---
layout: default
title:  "Pinpoint (APM)安装"
date:   2018-05-15 08:54:15 +0800
---  


Pinpoint 是一个Java开源的 APM 应用  
GitHub：	[https://github.com/naver/pinpoint](https://github.com/naver/pinpoint)  

应用主要分为4个部分：
1. Pinpoint Agent（从每个被监控组件采集数据）
2. Pinpoint Collector（收集每个Pinpoint Agent的数据并存入后台数据库）
3. HBASE Storage（性能数据存储）
4. Pinpoint Web UI（用户界面）  

![Branching]({{site.url}}/assets/images/2018-05-15-pinpoint-construce.png)



### 组件下载（具体版本需要参考官网的兼容列表）：
1. hbase 1.2.6  
    [http://archive.apache.org/dist/hbase/1.2.6/hbase-1.2.6-bin.tar.gz](http://archive.apache.org/dist/hbase/1.2.6/hbase-1.2.6-bin.tar.gz)
2. pinpoint-web 1.7.3  
    [https://github.com/naver/pinpoint/releases/download/1.7.3-RC1/pinpoint-web-1.7.3-RC1.war](https://github.com/naver/pinpoint/releases/download/1.7.3-RC1/pinpoint-web-1.7.3-RC1.war)
3. pinpoint-collector 1.7.3  
    [https://github.com/naver/pinpoint/releases/download/1.7.3-RC1/pinpoint-collector-1.7.3-RC1.war](https://github.com/naver/pinpoint/releases/download/1.7.3-RC1/pinpoint-collector-1.7.3-RC1.war)
4. tomcat 9.0.7  
    [http://mirrors.hust.edu.cn/apache/tomcat/tomcat-9/v9.0.7/bin/apache-tomcat-9.0.7.tar.gz](http://mirrors.hust.edu.cn/apache/tomcat/tomcat-9/v9.0.7/bin/apache-tomcat-9.0.7.tar.gz)

### HBASE配置步骤：
1. 解压安装文件  
```shell
tar zxvf hbase-2.0.0-src.tar.gz
mv hbase-2.0.0 /usr/local/
#设置Java home
vi /usr/local/hbase-1.2.6/conf/hbase-env.sh 
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.161-0.b14.el7_4.x86_64/
```
2. 设置HBASE文件存储位置  
```xml
vi /usr/local/hbase-1.2.6/conf/hbase-site.xml  
    <configuration>
      <property>
        <name>hbase.rootdir</name>
        <value>file:///data/hbase</value>
     </property>
    </configuration>
```

3. 启动HBASE
```shell
/usr/local/hbase-1.2.6/bin/start-hbase.sh
```
4. 验证服务
```
[root@k8-m medias]# jps 
29943 HMaster
6775 Jps
```
4. 初始化pinpoint数据表
```shell
/usr/local/hbase-1.2.6/bin/hbase shell https://raw.githubusercontent.com/naver/pinpoint/master/hbase/scripts/hbase-create.hbase
```
5. 查看hbase表内容，看到如下内容表明初始化成功
```shell
[root@k8-m medias]# /usr/local/hbase-1.2.6/bin/hbase shell
2018-05-02 17:06:31,018 WARN  [main] util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
HBase Shell; enter 'help<RETURN>' for list of supported commands.
Type "exit<RETURN>" to leave the HBase Shell
Version 1.2.6, rUnknown, Mon May 29 02:25:32 CDT 2017
hbase(main):001:0>  status 'detailed'
version 1.2.6
0 regionsInTransition
active master:  k8-m:34918 1525233998100
0 backup masters
master coprocessors: []
1 live servers
    k8-m:34887 1525234008805
        requestsPerSecond=0.0, numberOfOnlineRegions=482, usedHeapMB=229, maxHeapMB=3978, numberOfStores=482, numberOfStorefiles=52, storefileUncompressedSizeMB=0, storefileSizeMB=0, memstoreSizeMB=0, storefileIndexSizeMB=0, readRequestsCount=33238, writeRequestsCount=2478, rootIndexSizeKB=11, totalStaticIndexSizeKB=8, totalStaticBloomSizeKB=0, totalCompactingKVs=4329, currentCompactedKVs=4329, compactionProgressPct=1.0, coprocessors=[MultiRowMutationEndpoint]
        "AgentEvent,,1525228077865.0eeae43306d9ddf59d1cd72a9a27cdc3."
            numberOfStores=1, numberOfStorefiles=1, storefileUncompressedSizeMB=0, lastMajorCompactionTimestamp=0, storefileSizeMB=0, memstoreSizeMB=0, storefileIndexSizeMB=0, readRequestsCount=13, writeRequestsCount=24, rootIndexSizeKB=0, totalStaticIndexSizeKB=0, totalStaticBloomSizeKB=0, totalCompactingKVs=0, currentCompactedKVs=0, compactionProgressPct=NaN, completeSequenceId=16, dataLocality=0.0
        "AgentInfo,,1525228062698.efab5c5fae67d0e2c5a686bff73e5298."
```
6. 查看HBASE web
    浏览器打开： http://hbase-ip:16010/master-status
                
### pinpoint collector
1. 解压tomcat
```shell
tar zxvf apache-tomcat-9.0.7.tar.gz 
mv apache-tomcat-9.0.7 /opt/pp-col
```
2. 修改tomcat端口（如果没有端口冲突可以不用修改）
```shell
vi conf/server.xml
```

3. 启动tomcat
```shell
/opt/pp-col/bin/startup.sh
```
                
3. 解压war包
```shell
rm -rf /opt/pp-col/webapps/ROOT/*
unzip /opt/medias/pinpoint-collector-1.7.3-RC1.war -d /opt/pp-col/webapps/ROOT/
```
4. 修改配置文件（默认无需修改）
```
webapps/ROOT/WEB-INF/classes下面的pinpoint-collector.properties, hbase.properties
• pinpoint-collector.properties - contains configurations for the collector. Check the following values with the agent’s configuration options :
        ○ collector.tcpListenPort (agent’s profiler.collector.tcp.port - default: 9994)
        ○ collector.udpStatListenPort (agent’s profiler.collector.stat.port - default: 9995)
        ○ collector.udpSpanListenPort (agent’s profiler.collector.span.port - default: 9996)
• hbase.properties - contains configurations to connect to HBase.
        ○ hbase.client.host (default: localhost)
        ○ hbase.client.port (default: 2181)
```

### pinpoint web
1. 启动tomcat(略)
2. 解压war包
```shell
unzip /opt/medias/pinpoint-web-1.7.3-RC1.war -d webapps/ROOT/
```
3. 修改配置文件（默认无需修改）
```
webapps/ROOT/WEB-INF/classes/下面的pinpoint-web.properties, hbase.properties
hbase.properties - contains configurations to connect to HBase.
        • hbase.client.host (default: localhost)
        • hbase.client.port (default: 2181)
```


### pinpoint agent
![Branching]({{site.url}}/assets/images/2018-05-15-Pinpoint-agent-download.png)

1. Agent 下载：
   打开http://hbase-ip:16010/master-status 页面，在右上角打开设置页面 --> 打开installation页签，填写应用名称和应用ID，点击下载链接即可  

             
2. 下载以后解压，然后修改 pinpoint.config
```
profiler.collector.ip=192.168.1.51
```        
3. 配置中间件  
tomcat
```
#修改bin/catalina.sh，在文件开头加入：
CATALINA_OPTS="$CATALINA_OPTS -javaagent:/Users/mervin/eclipse/pinpoint-agent-1.7.2/pinpoint-bootstrap-1.7.2.jar"
CATALINA_OPTS="$CATALINA_OPTS -Dpinpoint.applicationName=web2"
CATALINA_OPTS="$CATALINA_OPTS -Dpinpoint.agentId=web120180502-1"
```
jetty
```
# 修改start.ini
# Module: jvm
--module=jvm
--exec
-javaagent:/Users/mervin/eclipse/pinpoint-agent-1.7.2/pinpoint-bootstrap-1.7.2.jar
-Dpinpoint.applicationName=web4
-Dpinpoint.agentId=web120180502-3
```
