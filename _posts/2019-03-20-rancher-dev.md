---
layout: post
title:  "持续集成Docker + Gitlab + Rancher（nfs + jenkins） + k8s"
date:   2019-03-20 18:56:00 +0800
categories: 技术
tag: rancher
---


## 一、结构图

## 二、容器化工具Docker
### 简介
### 作用
### 安装
``` shell

## 使用官方的安装脚本安装，启动&加入开机启动docker服务
curl -fsSL get.docker.com -o get-docker.sh && sudo sh get-docker.sh --mirror Aliyun && sudo systemctl start docker && sudo systemctl enable docker

```

### 配置

## 三、代码私有仓库Gitlab
### 简介
### 作用
### 安装

``` shell
## docker中安装&启动gitlab，gitlab产生的数据存到本地/srv/gitlab
docker run --detach --hostname 192.168.1.40 --publish 443:443 --publish 80:80 --publish 23:22 --name gitlab --restart always --volume /srv/gitlab/config:/etc/gitlab --volume /srv/gitlab/logs:/var/log/gitlab --volume /srv/gitlab/data:/var/opt/gitlab gitlab/gitlab-ce
```
Gitlab首次启动需要一段时间，安静等待...
### 配置
- 浏览器访问：[Gitlab地址](http://192.168.1.40)
- 输入密码，要求：最少8位

### 使用

## 四、容器管理Rancher
### 简介
### 安装

#### 非集群方式安装 
##### 环境

| 服务器 | [ip][ip]/[主机名][hostname] | 配置 |
| --- | --- | --- |
| Centos7.5 | 192.168.1.40/server | 2cpu、4g内存 |
| Centos7.5 | 192.168.1.41/work1 | 2cpu、4g内存 |
| Centos7.5 | 192.168.1.42/work2 | 2cpu、4g内存 |

##### 安装
``` shell
## 192.168.1.40安装&启动Rancher
sudo docker run -d --restart=unless-stopped -p 81:80 -p 444:443 -v /opt/rancher:/var/lib/rancher rancher/rancher
```

### 添加私有k8s集群

### 配置持久化存储NFS

### Docker私有仓库

### 部署负载均衡

### 持续集成CI/CD自动部署负载均衡

#### 制作包含项目依赖jar的maven镜像
1. 下载&运行&进入maven镜像：
`docker run -it maven:3-jdk-8-alpine /bin/bash`

2. 把项目的pom.xml复制到镜像：
`vi pom.xml`

3. 使用mvn install安装依赖的jar到镜像：
`mvn install`

4. 在docker所在主机重新保存镜像：
``` shell
docker commit -p $(docker ps -q --filter name=maven) maven:v1
```

5. 镜像上传到Docker私有仓库
``` shell
## docker私有仓库非https，客户端使用（pull，push）需要设置，不然会报错
echo '{ "insecure-registries":["192.168.1.40:28286"] }' > /etc/docker/daemon.json
systemctl restart docker

## 打标签
docker tag maven:v1 192.168.1.40:28286/maven:v1

## 上传镜像
docker push 192.168.1.40:28286/maven:v1
```

#### 制作包含项目依赖的node镜像




---
[ip]: 设置ip：
`vi /etc/sysconfig/network-scripts/ifcfg-ens33`
内容基本如下
```
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=f14288df-7ec0-4e62-b8ab-93d80f3eef17
DEVICE=ens33
ONBOOT=yes
BOOTPROTO=static
IPADDR=192.168.1.40
DNS1=114.114.114.114
NETMASK=225.225.225.0
GATEWAY=192.168.1.1
```

[hostname]: 设置hostname：
```
## 192.168.1.40执行：
hostnamectl set-hostname server
service network restart

## 192.168.1.41执行：
hostnamectl set-hostname work1
service network restart

## 192.168.1.42执行：
hostnamectl set-hostname work2
service network restart
```
