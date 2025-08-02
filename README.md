# Docker_learning

## core concepts
1. 容器
2. 镜像，创造容器，任何人都可以创造镜像并且放到下述的仓库中
3. 仓库Registry，存放各种镜像 [官方仓库](hub.docker.com)

> Docker不同于虚拟机，虚拟机是在原有的操作系统下运行一个全新的操作系统，有一个完整的新操作系统的交互界面，而Docker不一样，Docker的容器并没有一个单独的交互界面，而是使用原系统的交互界面，但通过某些操作可以是原来的系统内核展现出你想要的系统内核的特性，例如你可以在ubuntu22.04系统中展现ubuntu18.04系统的特性（用户态环境，包括软件包，配置文件等）。但是你进入了Docker容器，所以你的网络端口或者文件地址和宿主机的并不一样，因此会需要一系列的映射。

## 操作流程概述

获得镜像----利用镜像创建容器----运行容器----停止运行----删除容器----删除镜像	

## 核心操作
### 镜像

1. **拉取镜像** ，从某些仓库中拉取一个镜像，为后续运行作准备，`sudo docker pull <images>`(与git类似)

> 一般就是从官方仓库或其他仓库中下载某个别人已经制作好的镜像文件（当然也可以自己配置，后续会提到）


```
例如
docker pull docker.io/library/nginx:latest
1. docker.io即为docker的官方仓库，如果是从官方仓库下载镜像可以将这部分省略
2. library为docker官方的namespace，可以理解为上传该镜像的用户名，如果下载的镜像是官方镜像也可以省略
3. nginx即为镜像名称
4. $:$后跟着的是版本号，省略则为获取最新版本
省略后即为docker pull nginx

Question:
docker pull docker.n8n.io/n8nio/n8n  代表什么？
```

也可以拉取特定CPU架构的docker镜像，`-platform=xxxxxx`

例如`docker pull --platform=linux/arm64 nginx`

省略则会自动根据当前系统选择最为适合的CPU架构

2. **查看镜像**，`docker images`

可以查看已经下载的镜像名称以及ID

3. **删除镜像**，`docker rmi <image_name>/<image_ID>`（*image_name和image_ID基本上在各种情况下都互相等价，感兴趣可自行验证，以下只用image_name代替*）


### 容器

> 正如开头所说，使用Docker容器时要进行端口映射和目录映射，接下来会介绍这两个概念

1. **（创建并）运行容器**，`sudo docker run -d nginx`，将不在控制台打印进程日志`-d`


3. `sudo docker ps`，查看进程状态
4. **容器与宿主机的端口映射**，
```
-p
sudo docker run nginx -p 80:80
：前是宿主机端口，后是容器端口
--net=host
容器直接共享宿主机网络
```

5. **目录映射**，相当于容器的某个目录挂载到了宿主机的某个目录上 
