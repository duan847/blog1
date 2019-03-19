---
layout: post
title:  "k8s集群-使用rancher快速搭建"
date:   2019-3-15 13:09:01 +0800
categories: document
tag: k8s;容器;
---

* content
{:toc}


```
## 关闭防火墙
systemctl stop firewalld.service & systemctl disable firewalld.service
## 安装docker，使用官方的安装脚本安装，启动&加入开机启动docker服务
curl -fsSL get.docker.com -o get-docker.sh && sudo sh get-docker.sh --mirror Aliyun && sudo systemctl start docker && sudo systemctl enable docker
echo '{
"registry-mirrors":["https//registry.docker-cn.com"]
}' > /etc/docker/daemon.json
## 设置hostname，永久生效，每一台服务器的hostname应该设置为不一样如server1、work1、work2..
hostnamectl set-hostname work2

## 主节点docker安装&启动rancher
sudo docker run -d --restart=unless-stopped -p 81:80 -p 444:443 -v /opt/rancher:/var/lib/rancher rancher/rancher
## 浏览器访问https://192.168.1.40:444，网页打开后设置admin的密码

## 如果出现找不到镜像的错误：Error: No such image: rancher/hyperkube:v1.11.6-rancher1。可能是由于网速较慢导致，耐性等待。也可以选择手动执行：
docker run -d rancher/hyperkube:v1.11.6-rancher1

## 如果docker容器启动失败，重启电脑reboot

```
