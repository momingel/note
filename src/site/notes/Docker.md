---
{"dg-publish":true,"permalink":"/docker/","title":"Docker","tags":["gardenEntry"]}
---

# 安装

## 安装依赖包

``` shell
yum install -y yum-utils device-mapper-persistent-data lvm2
```

## 添加yum源

``` shell
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

## 安装docker

``` shell
yum install -y docker-ce
```

## 启动docker

``` shell
systemctl start docker
```

## 设置开机自启

``` shell
systemctl enable docker
```

## 配置docker镜像加速

``` shell
cat << EOF > /etc/docker/daemon.json
{
  "registry-mirrors": [
    "https://3ec1fcfd.mirror.aliyuncs.com",
    "https://reg-mirror.qiniu.com",
    "http://hub-mirror.c.163.com",
    "https://docker.mirrors.ustc.edu.cn"
  ]
}
EOF
```

``` shell
systemctl restart docker
```