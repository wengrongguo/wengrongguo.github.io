---
title: Linux下安装Nginx
date: 2017-09-08 22:01:48
tags:
- linux
- nginx
categories:
- linux
---
# 准备工作
### 系统条件
>系统本身要已安装gcc-c++才能进行源码编译安装，如没有，基于生产环境的要求，可参照如下方式进行安装

挂载光盘
```
mount -t iso9660 -o loop rhel-server-6.5-x86_64-dvd.iso system_dvd
```
配置本地yum仓库源(/etc/yum.repos.d/local.repo)
```
[local]
name=local  - Source Bate
baseurl=file:///opt/soft_data/system_dvd/
enabled=1
gpgcheck=0
```
刷新本地仓库源
```
yum clean & yum update
```
安装gcc-c++
```
yum -y install gcc-c++
```
### 准备安装包
* nginx：nginx源码包
* zlib：用于支持传输时压缩解压缩
* openssl：用于支持加解密
* pcre：用于支持正则表达写法

# 安装工作
### 预配置
将几个源码包放到同一个目录，并进行解压缩，进入解压缩后的nginx源码目录进行预配置（--prefix指定安装目录）
```
./configure --prefix=/opt/nginx --with-pcre=../pcre-8.37 --with-zlib=../zlib-1.2.8 --with-openssl=../openssl-1.0.2d --with-http_stub_status_module --with-http_ssl_module
```

### 编译安装
预配置成功后，还是在当前目录位置进行编译安装
```
make && make install
```

### 验证
进入安装后目录下的子目录sbin
```
cd /opt/nginx/sbin
```
启动nginx，查看是否启动成功，并通过浏览器进行访问
```
./nginx
```
# 后续工作
### 日志分割
nginx的访问日志不会自动分割，需要定时去切割，并进行压缩
分割脚本
```
#!/bin/bash
logs_path="/usr/local/nginx/logs"

pid_path="/usr/local/nginx/logs/nginx.pid"

mv ${logs_path}/access.log ${logs_path}/access_$(date -d "yesterday" +"%Y%m%d").log

kill -USR1 `cat ${pid_path}`
```
压缩脚本
```
#!/bin/sh

if [ -n "$1" ];then
        if [ ${#1} = 8 ]; then
                dealDate=$1 #accept the datestr yyyyMMdd
                sourceFiles=access.log
        elif [ ${#1} = 6 ]; then
                dealDate=$1 #accept the datestr yyyyMM
                sourceFiles=access_$dealDate*.log
        else
                #take the first argument as day offset
                dealDate=`date --date="1 days ago" +%Y%m%d`
                sourceFiles=access_$dealDate.log
        fi
else
        #the archive date, default 1 days ago
        dealDate=`date --date="1 days ago" +%Y%m%d`
        sourceFiles=access_$dealDate.log
fi
cd /usr/local/nginx/logs
tar -cjf access_$dealDate.tar.bz2 $sourceFiles
rm $sourceFiles
```
创建定时任务
```
crontab -e
```
添加以下内容
```
0  0 * * * /opt/nginx/nginx_log.sh
0 2 * * * /opt/nginx/ziphttplog.sh
```
保存后退出，查看是否添加成功
```
crontab -l
```
