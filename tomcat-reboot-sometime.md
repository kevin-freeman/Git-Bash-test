## 	Tomcat  定时重启

##### 开启监控

```shell
#多开一个窗口

#模糊匹配项目的端口号，比如80，实际是8080，...
[root@t1 ~]# watch -n1 -c  "netstat -nutlp|grep 80 "
```

##### 1.找到tomcat项目

```shell
[root@t1 tomcat]# ps -ef|grep tomcat|grep file
root      2895     1  3 12:18 pts/0    00:00:02 /usr/lib/jvm/java/bin/java -Djava.util.logging.config.file=/app/tomcat/conf/logging.properties -
...


#找到项目位置为：/app/tomcat
```

##### 2.创建重启脚本

```shell
#存放位置，项目应用的总目录
# bak-more-tomcat-restart.sh 多个tomcat应用时采用，与“more.."等效，代码更简洁
# more-tomcat-restart.sh     多个tomcat应用时采用，与上等效，但更规范
# one-tomcat-restart.sh      单个tomcat应用时采用

[root@t1 tomcat]# ll
total 24
-rwxr-xr-x 1 root   root    419 Feb 27 02:05 bak-more-tomcat-restart.sh
-rwxr-xr-x 1 tomcat tomcat 2790 Feb 27  2020 more-tomcat-restart.sh
-rwxr-xr-x 1 tomcat tomcat  797 Feb 27  2020 one-tomcat-restart.sh
drwxr-xr-x 9 tomcat tomcat 4096 Feb 27  2020 tomcat01
drwxr-xr-x 9 tomcat tomcat 4096 Feb 27  2020 tomcat02
drwxr-xr-x 9 tomcat tomcat 4096 Feb 27  2020 tomcat03

```

##### 3.创建定时任务

```shell
#确认系统服务ok
[root@t1 tomcat]# systemctl start  crond 
[root@t1 tomcat]# systemctl enable  crond 
[root@t1 tomcat]# systemctl status  crond |grep Active 


#创建定时任务，使用root身份执行
[root@t1 tomcat]# crontab -e 
#查看确认
[root@t1 tomcat]# crontab -l
#多个tomcat项目时打开下面一行注释做重启
#00 02 * * * sh /app/tomcat/more-tomcat-restart.sh  

#只有一个tomcat项目时
00 02 * * * sh /app/tomcat/one-tomcat-restart.sh
#####################################################################################

#创建定时任务，指定应用用户tomcat身份执行,假定tomcat项目的应用用户
[root@t1 tomcat]# crontab -e -u tomcat
[root@t1 tomcat]# crontab -l -u tomcat
#多个tomcat项目时打开下面一行注释做重启
#00 02 * * * sh /app/tomcat/more-tomcat-restart.sh  

#只有一个tomcat项目时
00 02 * * * sh /app/tomcat/one-tomcat-restart.sh

#重启系统服务，刷新列表
[root@t1 tomcat]# systemctl restart crond.service 

```

###### 脚本内容

more-tomcat-restart.sh

```shell
#!/bin/sh
echo "################################## 第一个tomcat ####################################"
echo 

TOMCAT01_PATH=/app/tomcat/tomcat01/bin
echo "TOMCAT_PATH is $TOMCAT01_PATH"
echo 

PID=`ps aux | grep /app/tomcat/tomcat01 | grep java | awk '{print $2}'`

if [ -n "$PID" ]; then
        echo "Will kill tomcat: $PID"
        sh "$TOMCAT01_PATH/shutdown.sh"
        sleep 6
else   
	echo "No Tomcat Process $PID"
	echo 
	echo 
fi


PID2=`ps aux | grep /app/tomcat/tomcat01 | grep java | awk '{print $2}'`

if [ -n "$PID2" ]; then
        kill -9 $PID2
        echo "Try to kill $PID2"
else 
	echo "No Tomcat Process $PID2"
	echo 
fi

sh "$TOMCAT01_PATH/startup.sh"
sleep 3
echo 


PID=`ps aux | grep /app/tomcat/tomcat01 | grep java | awk '{print $2}'`
if [ -n "$PID" ]; then
        echo "Restart tomcat successfully!"
	echo 
else
        echo "Fail to startup tomcat"
	echo 
        exit 1
fi


echo "################################## 第二个tomcat ####################################"
echo

TOMCAT02_PATH=/app/tomcat/tomcat02/bin
echo "TOMCAT_PATH is $TOMCAT02_PATH"
echo 

PID1=`ps aux | grep /app/tomcat/tomcat02 | grep java | awk '{print $2}'`

if [ -n "$PID1" ]; then
        echo "Will kill tomcat: $PID1"
        sh "$TOMCAT02_PATH/shutdown.sh"
        sleep 6
else
        echo "No Tomcat Process $PID1"
        echo 
fi


PID3=`ps aux | grep /app/tomcat/tomcat02 | grep java | awk '{print $2}'`

if [ -n "$PID3" ]; then
        kill -9 $PID3
        echo "Try to kill $PID3"
else
        echo "No Tomcat Process $PID3"
        echo 
fi

sh "$TOMCAT02_PATH/startup.sh"
sleep 3
echo 


PID1=`ps aux | grep /app/tomcat/tomcat02 | grep java | awk '{print $2}'`
if [ -n "$PID1" ]; then
        echo "Restart tomcat successfully!"
        echo 
else
        echo "Fail to startup tomcat"
        echo 
        exit 1
fi


echo "################################## 第三个tomcat ####################################"
echo


TOMCAT03_PATH=/app/tomcat/tomcat03/bin
echo "TOMCAT_PATH is $TOMCAT03_PATH"
echo 

PID5=`ps aux | grep /app/tomcat/tomcat03 | grep java | awk '{print $2}'`

if [ -n "$PID5" ]; then
        echo "Will kill tomcat: $PID5"
        sh "$TOMCAT03_PATH/shutdown.sh"
        sleep 6
else
        echo "No Tomcat Process $PID5"
        echo 
        echo 
fi


PID4=`ps aux | grep /app/tomcat/tomcat03 | grep java | awk '{print $2}'`

if [ -n "$PID4" ]; then
        kill -9 $PID4
        echo "Try to kill $PID4"
else
        echo "No Tomcat Process $PID4"
        echo 
fi

sh "$TOMCAT03_PATH/startup.sh"
sleep 3
echo 


PID5=`ps aux | grep /app/tomcat/tomcat03 | grep java | awk '{print $2}'`
if [ -n "$PID5" ]; then
        echo "Restart tomcat successfully!"
        echo 
else
        echo "Fail to startup tomcat"
        echo 
        exit 1
fi
```

one-tomcat-restart.sh

```shell
#!/bin/sh
TOMCAT_PATH=/app/tomcat/tomcat01/bin
echo "TOMCAT_PATH is $TOMCAT_PATH"
echo 


PID=`ps aux | grep /app/tomcat/tomcat01 | grep java | awk '{print $2}'`

if [ -n "$PID" ]; then
        echo "Will kill tomcat: $PID"
        sh "$TOMCAT_PATH/shutdown.sh"
        sleep 6
else   
	echo "No Tomcat Process $PID"
	echo 
	echo 
fi


PID2=`ps aux | grep /app/tomcat/tomcat01 | grep java | awk '{print $2}'`

if [ -n "$PID2" ]; then
        kill -9 $PID2
        echo "Try to kill $PID2"
else 
	echo "No Tomcat Process $PID2"
	echo 
fi

sh "$TOMCAT_PATH/startup.sh"
sleep 3
echo 


PID=`ps aux | grep /app/tomcat/tomcat01 | grep java | awk '{print $2}'`
if [ -n "$PID" ]; then
        echo "Restart tomcat successfully!"
	echo 
else
        echo "Fail to startup tomcat"
	echo 
        exit 1
fi
```

bak-more-tomcat-restart.sh

```shell
#!/bin/bash
#获取XXX项目进程ID
tomcatpid=`ps -ef | grep /app/tomcat | grep -v grep | awk '{print $2}'`
echo "tomcat项目进程ID为：$tomcatpid"
#杀进程
echo "kill tomcat PID..."
for id in $tomcatpid
do
kill -9 $id
done
echo "$tomcatpid已杀死..."
echo "重启tomcat..."

#依照实际路径修改
/app/tomcat/tomcat01/bin/startup.sh
/app/tomcat/tomcat02/bin/startup.sh
/app/tomcat/tomcat03/bin/startup.sh
```

##### 4.检查端口和进程

```shell
#调出进程号
[root@t1 tomcat]# jps
#调出开放端口号，比较二者是否一致
[root@t1 tomcat]# netstat -nutlp|grep java
```

##### 5.查看执行记录

```shell
[root@t1 tomcat]# vim /var/spool/mail/tomcat
#执行用户
From tomcat@t1.localdomain  Thu Feb 27 02:00:04 2020
Return-Path: <tomcat@t1.localdomain>
X-Original-To: tomcat
Delivered-To: tomcat@t1.localdomain
Received: by t1.localdomain (Postfix, from userid 1000)
	id D7FE4C0608; Thu, 27 Feb 2020 02:00:04 +0800 (CST)
From: "(Cron Daemon)" <tomcat@t1.localdomain>
To: tomcat@t1.localdomain

#执行脚本
Subject: Cron <tomcat@t1> sh /app/tomcat/one-tomcat-restart.sh

Content-Type: text/plain; charset=UTF-8
Auto-Submitted: auto-generated
Precedence: bulk
X-Cron-Env: <XDG_SESSION_ID=129>
X-Cron-Env: <XDG_RUNTIME_DIR=/run/user/1000>
X-Cron-Env: <LANG=en_US.UTF-8>
X-Cron-Env: <SHELL=/bin/sh>
X-Cron-Env: <HOME=/home/tomcat>
X-Cron-Env: <PATH=/usr/bin:/bin>
X-Cron-Env: <LOGNAME=tomcat>
X-Cron-Env: <USER=tomcat>
Message-Id: <20200226180004.D7FE4C0608@t1.localdomain>

#执行时间
Date: Thu, 27 Feb 2020 02:00:01 +0800 (CST)

#输出结果
TOMCAT_PATH is /app/tomcat/tomcat01/bin

No Tomcat Process 


No Tomcat Process 

Tomcat started.

#执行结果
Restart tomcat successfully!
```

##### 提示

```shell
# 该方案的tomcat应用详情，可依据具体实际情况进行调整
家目录： /app/tomcat
子目录1：/app/tomcat/tomcat01
子目录2：/app/tomcat/tomcat02
子目录3：/app/tomcat/tomcat03

所用端口：8005，8006，8007；8080，8081，8082

定时任务所采用脚本：one-tomcat-restart.sh

应用的管理用户：tomcat
定时任务的启动用户：tomcat
```

##### 效果

![image-20200227175524593](../../AppData/Roaming/Typora/typora-user-images/image-20200227175524593.png)