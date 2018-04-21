title: 在linux下安装redis
date: 2016-01-13 15:02:06
tags:
- linux
- redis
categories:
- linux
---

# 安装前提需求
拥有可安装软件权限的帐号，一般为root，系统已安装gcc相关编译软件

# 相关包说明
redis-3.0.6.tar.gz：redis安装源码包

# 安装详细步骤：
将源码文件上传到/opt/soft_bak目录下

### 解压redis安装包
```
	tar -zxvf /opt/soft_bak/redis-3.0.6.tar.gz -C /opt/soft_bak
```
### 创建redis安装目录/opt/redis
```
	mkdir /opt/redis
```
### 解压源码，并进入解压后的目录/opt/soft_bak/redis-3.0.6，执行编译安装
```
	tar -zxvf redis-3.0.6.tar.gz
	make PREFIX=/opt/redis install
```
### 安装完成后，复制源码目录里面的启动工具文件到/etc/init.d目录，并修改名字为redis
```
	cp /opt/soft_bak/redis-3.0.6/utils/redis_init_scriptcp /etc/init.d/redis
```
### 创建redis配置所在目录、日志文件所在目录、持久化所在目录
```
	mkdir /opt/redis
	mkdir /opt/redis/log	--存放日志的目录
	mkdir /opt/redis/dump	--存放数据持久化文件的目录
	mkdir /opt/redis/run	--存放pid标识文件
```
### 复制源码目录里面的redis配置文件到/opt/redis目录下
```
	cp /opt/soft_bak/redis-3.0.6/redis.conf /opt/redis/
```
### 修改/etc/init.d/redis中的相关内容，调整为本机配置
```
	REDISPORT=6379    --端口默认为6379
	EXEC=/opt/redis/bin/redis-server    --redis服务端执行文件所在位置
	CLIEXEC=/opt/redis/bin/redis-cli    --redis客户端执行文件所在位置
	
	PIDFILE=/opt/redis/run/redis.pid    --redis进程文件所在位置
	CONF="/opt/redis/redis.conf"    --redis配置文件所在位置
```
### 修改redis配置文件/opt/redis/redis.conf
```
	daemonize yes    --设置服务默认为后台启动
	pidfile /var/run/redis.pid    --设置进程文件所在位置
	port 6379    --设置端口
	logfile /opt/redis/log/stdout.log   --设置日志文件位置
	dbfilename dump.rdb    --设置持久化文件名
	dir /opt/redis/dump    --设置持久化文件所在目录
	slaveof 主服务ip 端口   --配置主从的话，从机需要多配置一项指向主机的主从配置
```
### 启动redis
```
	service redis start    --观察/opt/redis/log是否生成日志文件
```
### 测试redis服务是否正常
```
	telnet 服务ip 6379    --是否能正常链接
	set xxx 123456    --是否能正常写数据
```
### 停止redis
```
	service redis stop    --观察/opt/redis/dump是否生成持久化文件
```
