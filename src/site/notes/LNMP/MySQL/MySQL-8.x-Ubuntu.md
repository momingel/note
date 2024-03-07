---
{"dg-publish":true,"permalink":"/lnmp/my-sql/my-sql-8-x-ubuntu/","title":"MySQL-8.x-Ubuntu","created":"2024-01-29T11:39:09.120+08:00","updated":"2024-03-07T11:09:51.220+08:00"}
---

# 安装依赖

``` shell
apt-get install -y ncurses-devel libaio-devel gcc gcc-c++ glibc cmake3 autoconf openssl openssl-devel
```

``` shell
apt-get -y install centos-release-scl
```

``` shell
apt-get -y install devtoolset-11-gcc devtoolset-11-gcc-c++ devtoolset-11-binutils
```

# 安装

进入解压后的目录

``` shell
cd mysql-8.3.0
```

cmake配置路径

``` shell
cmake3 \
-DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
-DMYSQL_DATADIR=/data/mysql/data \
-DSYSCONFDIR=/etc \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci \
-DEXTRA_CHARSETS=all \
-DMYSQL_UNIX_ADDR=/tmp/mysql.sock \
-DWITH_MYISAM_STORAGE_ENGINE=1 \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_ARCHIVE_STORAGE_ENGINE=1 \
-DWITH_PARTITION_STORAGE_ENGINE=1 \
-DFORCE_INSOURCE_BUILD=1 \
-DWITH_SYSTEMD=1 \
-DWITH_BOOST=./boost
```

- DCMAKE_INSTALL_PREFIX：mysql安装目录
- DMYSQL_DATADIR：mysql数据文件目录
- DSYSCONFDIR：mysql配置文件目录
- DDEFAULT_CHARSET：默认字符集
- DDEFAULT_COLLATION：默认排序规则/校验字符集
- DEXTRA_CHARSETS：额外字符集支持
- DMYSQL_UNIX_ADDR：mysql.sock文件目录
- DWITH_MYISAM_STORAGE_ENGINE：myisam引擎支持
- DWITH_INNOBASE_STORAGE_ENGINE：innobase引擎支持
- DWITH_ARCHIVE_STORAGE_ENGINE：archive引擎支持
- DWITH_PARTITION_STORAGE_ENGINE：partition引擎支持
- DWITH_SYSTEMD：systemd支持
- DWITH_BOOST：boost扩展库目录

make编译

``` shell
make -j 4
```

- j：表示使用几个CPU核心加速

make安装

``` shell
make install
```