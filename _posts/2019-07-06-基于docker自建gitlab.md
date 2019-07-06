---
author: zyqzyq
date: 2019-07-06 10:10:32+00:00
layout: post
title: 基于docker自建gitlab
key: 20190706
tags:
- gitlab
---

# gitlab_install
  记录gitlab安装过程
# 安装docker

官网地址：[安装docker-ce centos教程](https://docs.docker.com/install/linux/docker-ce/centos/ )

### 卸载旧版本

```
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```

## 安装Docker CE

### 添加存储库

```
$ sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```

```
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

*备注*：如果失败看要修改镜像地址为https://mirrors.ustc.edu.cn/docker-ce/linux/centos/docker-ce.repo

```
$ sudo yum-config-manager --enable docker-ce-edge
$ sudo yum-config-manager --enable docker-ce-test
```

### 安装docker ce

```
$ sudo yum install docker-ce
$ sudo systemctl start docker
# 确认docker是否安装成功
$ sudo docker run hello-world
```

# 安装gitlab centos docker版

官网地址：[gitlab docker安装](https://docs.gitlab.com/omnibus/docker/README.html)

```
sudo docker run --detach \
	--hostname gitlab.example.com \
	--publish 443:443 --publish 80:80 --publish 2222:22 \
	--name gitlab \
	--restart always \
	--volume /srv/gitlab/config:/etc/gitlab \
	--volume /srv/gitlab/logs:/var/log/gitlab \
	--volume /srv/gitlab/data:/var/opt/gitlab \
	gitlab/gitlab-ce:latest

```

*备注*：默认镜像下载较慢，可更换

编辑 /etc/docker/deamon.json,也可自行选择其他加速地址。




## 备份与恢复

官网地址：[gitlab备份恢复](https://docs.gitlab.com/ce/raketasks/backup_restore.html)

```
# 备份
sudo docker exec -t gitlab gitlab-rake gitlab:backup:create
# 恢复
sudo docker exec -it gitlab gitlab-rake gitlab:backup:restore
```

*备注*：gitlab需与备份版本相同（当前版本latest 11.6.3）
