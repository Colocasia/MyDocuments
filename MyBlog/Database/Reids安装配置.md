
[TOC]

# **Redis 安装和配置** #

好记性不如烂笔头啊，还是记录一下!

----------

## **<font color=#191970 size=5>1.下载 Redis</font> ** ##

下载 Redis

``` shell
cd /home/downloads
wget http://download.redis.io/releases/redis-4.0.9.tar.gz
```


## **<font color=#191970 size=5>2.在 CentOS 7 中安装 Redis</font> ** ##

解压 Redis。

``` shell
tar -zxvf redis-4.0.9.tar.gz
cd redis-4.0.9
```

编译和安装 Redis。

``` shell
make
make install
```

make install安装完成后，会在/usr/local/bin目录下生成下面几个可执行文件，它们的作用分别是：

| 命令 | 用途 | 
| :- | :- | 
redis-server | Redis服务器端启动程序
redis-cli | Redis客户端操作工具。也可以用telnet根据其纯文本协议来操作
redis-benchmark | Redis性能测试工具 
redis-check-aof | 数据修复工具 
redis-check-dump | 检查导出工具 



## **<font color=#191970 size=5>3.在 CentOS 7 中配置 Redis</font> ** ##

复制配置文件到/etc/目录：

``` shell
cp redis.conf /etc/
```

为了让Redis后台运行，一般还需要修改redis.conf文件：

``` shell
vi /etc/redis.conf
```

修改daemonize配置项为yes，使Redis进程在后台运行：

``` shell
daemonize yes
```

## **<font color=#191970 size=5>4.在 CentOS 7 中启动 Redis</font> ** ##

配置完成后，启动Redis：

``` shell
cd /usr/local/bin
./redis-server /etc/redis.conf
```

检查启动情况：

``` shell
ps -ef | grep redis
```

看到类似下面的一行，表示启动成功：

``` shell
root     18443     1  0 13:05 ?        00:00:00 ./redis-server *:6379 
```

添加开机启动项：
让Redis开机运行可以将其添加到rc.local文件，也可将添加为系统服务service。
本文使用rc.local的方式，添加service请参考：Redis 配置为 Service 系统服务 。

为了能让Redis在服务器重启后自动启动，需要将启动命令写入开机启动项：
``` shell
echo "/usr/local/bin/redis-server /etc/redis.conf" >>/etc/rc.local
```

## **<font color=#191970 size=5>5.Redis常用配置</font> ** ##

在 前面的操作中，我们用到了使Redis进程在后台运行的参数，下面介绍其它一些常用的Redis启动参数：

| 配置 | 用途 | 
| :- | :- | 
daemonize | 是否以后台daemon方式运行
pidfile | pid文件位置
port | 监听的端口号
timeout | 请求超时时间
loglevel | log信息级别
logfile | log文件位置
databases | 开启数据库的数量
save * * | 保存快照的频率，第一个*表示多长时间，第三个*表示执行多少次写操作。在一定时间内执行一定数量的写操作时，自动保存快照。可设置多个条件。
rdbcompression | 是否使用压缩
dbfilename | 数据快照文件名（只是文件名）
dir | 数据快照的保存目录（仅目录）
appendonly | 是否开启appendonlylog，开启的话每次写操作会记一条log，这会提高数据抗风险能力，但影响效率。
appendfsync | appendonlylog如何同步到磁盘。三个选项，分别是每次写都强制调用fsync、每秒启用一次fsync、不调用fsync等待系统自己同步
