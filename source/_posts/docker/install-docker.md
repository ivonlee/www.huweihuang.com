---
title: "[Docker] 安装Docker"
catalog: true
date: 2018-6-23 17:26:24
type: "tags"
subtitle:
header-img: "http://ozilwgpje.bkt.clouddn.com/scenery/building.jpg?imageslim"
tags:
- Docker
catagories:
- Docker
---

## 1. CentOS 安装Docker

> 建议使用centos7

### 1.1. 安装Docker

#### 1.1.1. 卸载旧版本

旧版本的Docker命名为`docker`或`docker-engine`，如果有安装旧版本，先卸载旧版本

```shell
$ sudo yum remove -y docker \
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

#### 1.1.2. 使用仓库安装

1、安装yum-utils、device-mapper-persistent-data、lvm2

```shell
$ sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```

2、添加软件源

```shell
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

#### 1.1.3. 安装Docker

安装最新版本的Docker CE。

```shell
$ sudo yum install -y docker-ce 
```

#### 1.1.4. 启动Docker

```shell
# 启动Docker
$ sudo systemctl start docker
# 运行容器
$ sudo docker run hello-world
```

### 1.2. 安装指定版本Docker

1、列出可安装版本

```shell
$ yum list docker-ce --showduplicates | sort -r

docker-ce.x86_64            18.03.0.ce-1.el7.centos             docker-ce-stable
```

2、安装指定版本

例如：docker-ce-18.03.0.ce

```shell
$ sudo yum install docker-ce-<VERSION STRING>
```

### 1.3. 升级Docker

依据1.2的方法选择指定版本安装。

### 1.4. 卸载Docker

```shell
# 卸载Docker
$ sudo yum remove docker-ce

# 清理镜像、容器、存储卷等
$ sudo rm -rf /var/lib/docker
```

## 2. Ubuntu 安装Docker

### 2.1. 安装Docker

#### 2.1.1. 卸载旧版本

旧版本的Docker命名为`docker`或`docker-engine`，如果有安装旧版本，先卸载旧版本

```shell
sudo apt-get remove docker docker-engine docker.io
```

#### 2.1.2. 使用仓库安装

1、升级apt

```shell
sudo apt-get update
```

2、允许apt使用https

```shell
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
```

3、添加Docker 官方的GPG密钥

```shell
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

4、添加Docker软件源

```shell
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

#### 2.1.3. 安装Docker

```shell
# update
sudo apt-get update

# install docker
sudo apt-get install docker-ce
```

#### 2.1.4. 启动Docker

```shell
# 设置为开机启动
sudo systemctl enable docker
# 启动docker
sudo systemctl start docker
```

### 2.2. 安装指定版本Docker

1、列出仓库的可安装版本，`apt-cache madison docker-ce`。

```shell
# apt-cache madison docker-ce
 docker-ce | 18.06.0~ce~3-0~ubuntu | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 18.03.1~ce~3-0~ubuntu | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
```

2、指定版本安装

例如：docker-ce=18.03.0~ce-0~ubuntu

```shell
sudo apt-get install docker-ce=<VERSION>
```

### 2.3.  升级Docker

```shell
# 更新源
sudo apt-get update
# 依据上述方法，指定版本安装
```

### 2.4. 卸载Docker

```shell
# 卸载 docker ce
sudo apt-get purge docker-ce

# 清理镜像、容器、存储卷等
sudo rm -rf /var/lib/docker
```



文章参考：

https://docs.docker.com/install/linux/docker-ce/centos/

https://docs.docker.com/install/linux/docker-ce/ubuntu/