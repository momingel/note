---
{"dg-publish":true,"permalink":"/linux/linux/","title":"Linux","created":"2024-01-09T11:16:28.223+08:00","updated":"2024-03-07T11:07:45.617+08:00"}
---

# firewalld

## 概述

防火墙是计算机网络中的一种关键安全组件，用于保护系统免受网络攻击。Rocky Linux 提供了 `firewalld` 工具，它是一个动态的防火墙管理器，简化了防火墙规则的配置和管理。

## 安装和启动 firewalld

在 Rocky Linux 中，默认情况下 `firewalld` 应该已经安装。如果没有安装，可以使用以下命令安装：

```bash
sudo dnf install firewalld
```

安装完成后，启动 `firewalld` 并设置开机自启动：

```bash
sudo systemctl start firewalld
sudo systemctl enable firewalld
```

## firewalld 基本操作

### 查看防火墙状态

```bash
sudo firewall-cmd --state
```

### 查看防火墙规则

```bash
sudo firewall-cmd --list-all
```

### 查看可用服务

```bash
sudo firewall-cmd --get-services
```

### 查看可用区域

```bash
sudo firewall-cmd --get-zones
```

## 防火墙配置示例

### 允许某个服务

```bash
sudo firewall-cmd --zone=public --add-service=http --permanent
sudo firewall-cmd --reload
```

### 允许特定端口

```bash
sudo firewall-cmd --zone=public --add-port=8080/tcp --permanent
sudo firewall-cmd --reload
```

### 允许某个 IP 地址访问特定端口

```bash
sudo firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source address="192.168.1.2" port protocol="tcp" port="22" accept' --permanent
sudo firewall-cmd --reload
```

## 高级配置

对于更复杂的防火墙配置，可能需要编辑 `firewalld` 的配置文件或使用更底层的工具，如 `iptables`。请根据具体的安全要求和网络架构谨慎调整防火墙规则。

## 总结

Rocky Linux 的防火墙是系统安全的重要组成部分，通过适当配置和管理，可以有效防止未经授权的访问和网络攻击。管理员应根据实际需求和网络环境，谨慎设置防火墙规则以确保系统的安全性。

* * *

这是一个简要的文档，具体的配置取决于你的系统需求和网络环境。在实际应用中，请根据需要调整防火墙规则，并确保你的配置符合系统的安全标准。