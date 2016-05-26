---
layout: post
title: "用 Log.io 在 web 端实时展现应用日志"
tags: Linux, log.io, logging, sersync2, rsync
unsplash_source_number: 5
excerpt: 本文作者：薛勇。他在本文中介绍了如何用 log.io 来搭建一个 Web 日志查看平台，以满足日志实时查看的需求。
---

本文作者：薛勇。他在本文中介绍了如何用 log.io 来搭建一个 Web 日志查看平台，以满足日志实时查看的需求。

# web实时展现应用日志

## 概述

**目的:**

很多时候,开发,测试或生产环境不允许开发直接登录服务器,运维经常都会碰到开发拉日志的需求,或者开发允许直接登录服务器,但应用分布在多台服务器的情况下,需要打开多台服务器的tail窗口,日志查看就会变的很混乱,此文通过以web方式提供给开发一个集中的,实时的日志查看方式

> 关键字:  rsync+sersync2+log.io

**优点:**

> 1. 通过web方式,让开发直接查看类似与tail体验的日志,省去运维介入,提高效率    
> 2. 多个应用日志可集中查看,不用登录多台服务器,打开多个窗口    

**缺点:**

> 1. 只能提供打开页面后的日志实时查看,不能查询历史日志
> 2. 每台应用服务器需要安装sersync2来做实时同步

缺点一临时解决方案:应用按 天/时/大小/ 做日志切割,然后把历史日志压缩,最后把日志的存储目录通过web方式开放出来提供给开发下载

**设计思想:**

> 通过rsync+sersyc2将应用日志实时同步到日志服务器,然后通过log.io实时展现同步过来的日志

**环境准备:**

> 1.服务器三台:

  * 192.168.1.1 (rsync-client,sersync2)
  * 192.168.1.2 (rsync-client,sersync2)
  * 192.168.1.3 (**rsync-server,log.io**)

> 2.操作系统--**centos6.5_x64**
> 3.默认1.1和1.2已经安装好了应用,如tomcat,jboss....,这里以tomcat为例
> 4.默认都已安装yum源的拓展仓库,如果遇到yum无法安装的,请先安装拓展仓库


## 搭建rsync-server
1. 安装rsync-server(192.168.1.3)
> yum install -y rsync xinetd

2. 创建rsync-client同步过来存放日志的文件夹
> mkdir -p /app/appslog/cms
> mkdir -p /app/appslog/job

3. 创建rsync的帐号文件
> touch /etc/rsyncd.pass
>
>```java
>xueyong:xueyong123			//用户名:密码
>```
>
> chmod 600 /etc/rsyncd.pass

4. 创建rsync-server的主配置文件

> touch /etc/rsyncd.conf

```conf
log file = /var/log/rsyncd.log
pidfile = /var/run/rsyncd.pid
lock file = /var/run/rsyncd.lock
secrets file = /etc/rsyncd.pass
[cms]								//同步区块
path =  /app/appslog/cms			//同步存储路径
comment = cms syslog
uid = root
gid = root
port = 873
read only = no
list = no
max connections = 200
timeout = 600
hosts allow = 192.168.1.1			//cms区块只允许192.168.1.1的同步请求
[job]
path =  /app/appslog/job
comment = job syslog
uid = root
gid = root
port = 873
read only = no
list = no
max connections = 200
timeout = 600
hosts allow = 192.168.1.2			//job区块区块只允许192.168.1.2的同步请求
```

5. 修改xinetd的配置文件

> vim /etc/xinetd.d/rsync

```bash
service rsync
{
        disable = no				//yes替换为no
        flags           = IPv6
        socket_type     = stream
        wait            = no
        user            = root
        server          = /usr/bin/rsync
        server_args     = --daemon
        log_on_failure  += USERID
}
```

6. 启动xinetd,由于rsync是用守护启动,启动xinetd遍可自动启动rsync

> service xinetd start

7. 查看rsync是否启动成功

> netstat -anutp |grep 873
```bash
tcp        0      0 0.0.0.0:873                 0.0.0.0:*                   LISTEN      17267/xinetd
```

可看到rsync的873端口已打开,代表已启动成功。

搭建sersync2
-------------
**安装sersync2(192.168.1.1和192.168.1.2)**

1. 下载解压sersync2(网上比较难找,我已经上传到自己的服务器上)

> wget http://180.168.127.10:14177/sersync2.5_64bit_binary_stable_final.tar.gz
> tar zxvf sersync2.5_64bit_binary_stable_final.tar.gz

2. 将解压后的文件移动到自定义的目录

> mv GNU-Linux-x86 /usr/local/sersync
> cd /usr/local/sersync

3. 修改配置文件

> cp confxml.xml confxml.xml.bak
> mv confxml.xml cms.xml

4. 创建rsync的密码文件

>echo "xueyong123" >/usr/local/sersync/passwd.txt
>chmod 600 /usr/local/sersync/passwd.txt

5. 修改sersync2的主配置文件

> vim /usr/local/sersync/cms.xml

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<head version="2.5">
    <host hostip="localhost" port="8008"></host>
    <debug start="false"/>
    <fileSystem xfs="false"/>
    <filter start="true"> <!--开启过滤器,用来排除指定类型的文件-->
        <exclude expression="(.*)\.log"></exclude> <!--排除.log后缀的文件-->
        <exclude expression="(.*)\.txt"></exclude> <!--排除.txt后缀的文件-->
    </filter>
    <inotify>
        <delete start="false"/> <!--不监视文件的删除状态-->
        <createFolder start="true"/>
        <createFile start="true"/>
        <closeWrite start="true"/>
        <moveFrom start="true"/>
        <moveTo start="true"/>
        <attrib start="true"/>
        <modify start="true"/>
    </inotify>
    <sersync>
        <localpath watch="/app/tomcat1/logs"> <!--本地监视的同步目录-->
            <remote ip="192.168.1.3" name="cms"/> <!--同步到远端的服务器IP,及同步目录标识-->
        </localpath>
        <rsync>
            <commonParams params="-artuz"/> <!--rsync同步参数-->
            <auth start="true" users="xueyong" passwordfile="/usr/local/sersync/passwd.txt"/> <!--开启用户验证及验证参数-->
            <userDefinedPort start="false" port="874"/><!-- port=874 -->
            <timeout start="false" time="100"/><!-- timeout=100 -->
            <ssh start="false"/>
        </rsync>
		<!--以下都是默认的配置文件,无需改动-->
        <failLog path="/tmp/rsync_fail_log.sh" timeToExecute="60"/><!--default every 60mins execute once-->
        <crontab start="false" schedule="600"><!--600mins-->
            <crontabfilter start="false">
                <exclude expression="*.php"></exclude>
                <exclude expression="info/*"></exclude>
            </crontabfilter>
        </crontab>
        <plugin start="false" name="command"/>
    </sersync>
    <plugin name="command">
        <param prefix="/bin/sh" suffix="" ignoreError="true"/>  <!--prefix /opt/tongbu/mmm.sh suffix-->
        <filter start="false">
            <include expression="(.*)\.php"/>
            <include expression="(.*)\.sh"/>
        </filter>
    </plugin>
</head>
```

**如果同一台服务器里有不同的目录需要同步,参考cms.xml再重新生成一个新的配置文件**

6. 同步前先做一次全量同步

```Shell
rsync -avzu --password-file=/usr/local/sersync/passwd.txt --include "*.out" --include "*.gz" --exclude "*" /app/tomcat1/logs/ xueyong@192.168.1.3::cms
```

7. 启动rsync2

> /usr/local/sersync/sersync2 -d -o /usr/local/sersync/cms.xml

8. 测试rsync2是否正常工作

>在1.1上的/app/tomcat1/logs目录里创建一个aaa.tar.gz文件,然后在1.3上的/app/appslog/cms目录里看看是否已同步过去,如果有则说明rsync2已正常工作.
>在1.1上的/app/tomcat1/logs目录里删除aaa.tar.gz文件,然后在1.3上/app/appslog/cms目录里看看是否aaa.tar.gz还存在,如果存在,则说明rsync2的同步策略有效


##设置日志切割

**配置logratate (192.168.1.1和192.168.1.2)**

1. 把logrotate改成每天零点轮询

> vim /etc/anacrontab

```Shell
SHELL=/bin/sh
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
**RANDOM_DELAY=0			 //修改为0
START_HOURS_RANGE=0-0		//修改为0-0
1       0       cron.daily              nice run-parts /etc/cron.daily		//第二个修改为0
7       25      cron.weekly             nice run-parts /etc/cron.weekly
@monthly 45     cron.monthly            nice run-parts /etc/cron.monthly
```

2. 创建tomcat的logrotate轮询文件

> touch /etc/logrotate.d/tomcat1

```Shell
/app/tomcat1/logs/*.out {
        rotate 4				//日志转存保留前4份，多余的会被删除
        copytruncate			//打开中的日志转储
        compress				//启用压缩,文件以.gz后缀
        nodelaycompress			//转储并压缩
        missingok				//如果日志文件不存在，继续处理下一个文件而不产生报错信息
        daily					//每天轮询
        dateext					//归档旧日志文件时，文件名添加YYYYMMDD形式日期
        sharedscripts
        postrotate				//转储结束后执行的脚本,需配合sharedscripts,endscript用
             /bin/find /app/tomcat1/logs  -mtime +4 -exec rm -f {} \;		//保留logs目录下最近四天的日志文件
        endscript
}
```

3. 手动执行下logrotate  (如果不执行,可能第二天你会发现配置没生效)

> /usr/sbin/logrotate -f /etc/logrotate.conf

## 搭建log.io

**安装log.io(192.168.1.3)**

1. 安装nodejs和npm (log.io是用nodejs开发的,依赖项)

> yum install -y nodejs npm

2. 使用npm安装log.io

> npm install -g log.io --prefix=/app/logio

3. 修改log.io的harvester配置文件  (在家目录下)

> vim ~/.log.io/harvester.conf

```Shell
exports.config = {
  nodeName: "logview_server",				//定义node名字
  logStreams: {
    cms: [									//定义node的节点名,对应前面同步的应用名
      "/app/appslog/cms/catalina.out",		//定义待查看的日志路径,对应前面同步过来的应用日志
    ],
     job: [
      "/app/appslog/job/catalina.out",
    ]
  },
  server: {
    host: '0.0.0.0',
    port: 28777
  }
}
```

4. 启动log.io

> cd /app/logio/bin
> nohup ./log.io-server start &
> nohup ./log.io-harvester start &

5. 查看log.io

> 在浏览器里访问 http://192.168.1.3:28778

6. 停止log.io的方法

```Shell
ps -ef |grep log.io |grep -v "grep"
```

找到两个进程ID,然后kill

## 使用nginx反代(可选)

**安装nginx (192.168.1.3)**

1. 下载安装nginx

> wget http://180.168.127.10:14177/nginx-1.8.0.tar.gz
> yum install -y pcre-devel.x86_64 openssl-devel.x86_64
> tar zxvf nginx-1.8.0.tar.gz
> cd nginx-1.8.0
> ./configure --prefix=/app/nginx --with-http_ssl_module --with-http_stub_status_module --with-pcre
> make && make install

2. 新增log.io在nginx的配置文件

> mkdir -p /app/nginx/conf/hosts
> touch /app/nginx/conf/hosts/log.conf

```bash
map $http_upgrade $connection_upgrade {
    default upgrade;
    '' close;
}
server {
        listen          80;
        server_name     log.xueyong.com;
        access_log      logs/log.access.log  main;
        access_log      off;
        error_log       logs/log.error.log   notice;
        location / {
                proxy_pass http://127.0.0.1:28778/;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "Upgrade";
        }
        location /down/ {			//由于log.io只能实时展示日志,无法提供查询,所以这里开放down这个二级目录供开发来下载历史日志
                autoindex on;
                autoindex_exact_size off;
                autoindex_localtime on;
                root html;
        }
        error_page      500 502 503 504 /50x.html;
        location = /50x.html {
                root html;
        }
}
```

> vim /app/nginx/conf/nginx

```Shell
include hosts/*.conf;			//在http的区块里新增一个include
```

3. 把同步目录软链接到down里

> cd /app/nginx/html/
> ln -s down /app/appslog/

4. 启动nginx

> /app/nginx/sbin/nginx start

5. nginx日志切割 (使用logrotate)

> touch /etc/logrotate.d/nginx

```Shell
/app/nginx/logs/*.log {
        rotate 4
        copytruncate
        compress
        nodelaycompress
        missingok
        daily
        dateext
        sharedscripts
        postrotate
             /bin/find /app/nginx/logs  -mtime +30 -exec rm -f {} \;		//nginx没做日志同步,本地保留30天的日志
        endscript
}
```

6. 查看log.io

> 开发通过http://log.xueyong.com 可以访问到log.io,查看应用的实时日志
> 开发通过http://log.xueyong.com/down/  可以访问到日志目录,去下载历史日志
