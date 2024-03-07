---
{"dg-publish":true,"permalink":"/lnmp/lnmp/","title":"LNMP","created":"2024-01-22T16:44:49.994+08:00","updated":"2024-03-07T11:31:05.144+08:00"}
---

# Linux

## 安装工具1

> Centos

``` shell
yum -y install vim iputils net-tools
```

> Ubuntu

``` shell
apt-get -y install vim iputils net-tools
```

## 配置静态IP

> Centos

`/etc/sysconfig/network-scripts/ifcfg-***`
```
TYPE=Ethernet
BOOTPROTO=static
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
IPV6_FAILURE_FATAL=no
NAME=eno16780032
UUID=180dc319-10ba-4c05-9fbc-586cb09bc5c1
DEVICE=eno16780032
ONBOOT=yes
IPADDR=192.168.110.80
GATEWAY=192.168.110.1
NETMASK=255.255.255.0
DNS1=8.8.8.8
```

> Ubuntu

`/etc/netplan/**-installer-config.yaml`
```
network:
	ethernets:
		enp0s5:
			dhcp4: no
			addresses:
			    - 10.211.55.80/24
			routes:
				- to: default
				  via: 10.211.55.1
			nameservers:
				addresses:
					- 10.211.55.1
	version: 2
	renderer: networkd
```

# nginx

## 安装依赖

> Centos

``` shell
yum install -y gcc-c++ pcre pcre-devel zlib zlib-devel openssl openssl-devel
```

> Ubuntu

``` shell
apt-get install -y gcc-c++ pcre pcre-devel zlib zlib-devel openssl openssl-devel
```

## 下载

https://nginx.org/en/download.html

## 解压

``` shell
tar -xzvf nginx-1.25.3.tar.gz
```

## 安装

进入解压后的目录

``` shell
cd ./nginx-1.25.3
```

然后执行

> 检测配置

``` shell
./configure
```

> make编译

``` shell
make
```

> make安装

``` shell
make install
```

## 命令

> 启动

``` shell
/etc/local/nginx/sbin/nginx
```

> 重载配置

``` shell
/etc/local/nginx/sbin/nginx -s reload
```

> 关闭

``` shell
/etc/local/nginx/sbin/nginx -s stop
```

# MySQL

## 下载

下载：
https://dev.mysql.com/downloads/mysql/

> 5.x版本：
![Pasted image 20240123131718.png](/img/user/images/Pasted%20image%2020240123131718.png)

> 8.x版本：
![Pasted image 20240123131832.png](/img/user/images/Pasted%20image%2020240123131832.png)

## 创建用户组和用户

``` shell
groupadd mysql
useradd mysql -g mysql -M -s /sbin/nologin
```

## 解压

``` shell
tar -xzvf mysql-boost-8.3.0.tar.gz
```

## 安装

[[LNMP/MySQL/MySQL-5.x-Centos\|MySQL-5.x-Centos]]

[[LNMP/MySQL/MySQL-8.x-Centos\|MySQL-8.x-Centos]]

[[LNMP/MySQL/MySQL-8.x-Ubuntu\|MySQL-8.x-Ubuntu]]

## 初始化

> 修改密码

默认密码位于`mysql.log`文件

``` shell
mysql -uroot -p
```

``` mysql
set password = password('xxx');
flush privileges;
```

> 修改访问权限

``` shell
mysql -uroot -p
```

``` mysql
update user set host = '%' where user = 'root';
flush privileges;
```

# PHP

## 更新yum源

> Centos

`配置epel源`
``` shell
yum install -y epel-release
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
```

`配置remi源`
``` shell
rpm -ivh http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
```

## 安装

``` shell
yum -y install --enablerepo=remi --enablerepo=remi-php56 php php-fpm php-mysqlnd
```

## 命令

> 启动

``` shell
systemctl start php-fpm
```

> 关闭

``` shell
systemctl stop php-fpm
```

> 重启

``` shell
systemctl restart php-fpm
```

> 查看状态

``` shell
systemctl status php-fpm
```