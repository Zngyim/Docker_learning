# Docker_learning

***以下docker命令均要在sudo下执行***

***可以将docker转变为non-root user，详见[官方教程](https://docs.docker.com/engine/install/linux-postinstall/)***


[爬爬虾教程](https://www.bilibili.com/video/BV1THKyzBER6/?spm_id_from=333.1007.top_right_bar_window_history.content.click&vd_source=322f1eb47e35d454d075998e82c3b3ce)

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
4. $:$后跟着的是tag，也可以理解为版本，在hub.docker.com中可以查到
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

1. 接下来先不考虑端口映射和目录映射
```
创建容器 
docker create nginx
也就是运用nignx这个镜像来创建容器

启动容器
docker start nginx
docker start -d nginx------增加-d参数即为分离模式，容器在后台执行，不会打印日志 

查看容器状态
docker ps------可以查看正在运行状态的容器
docker ps -a-----代表all，即可以查看所有容器

查看容器的详细信息
docker inspect <ID/name>-----可以查看容器的详细信息，包括挂载卷、端口映射信息

停止容器
docker stop <ID/name>

删除容器
docker rm <ID/name>
docker rm -f <ID/name>------强制删除容器，正在运行的容器
```

2. **端口映射**
```
端口映射
> 一般来说，创建的所有容器会默认加入到一个子网里，这个子网与宿主机的网络是隔离的，也就是说宿主机的端口和容器的端口并不是一一对应的关系，因此如果要让二者的端口一一对应，那么就要进行端口映射

如
-p
sudo docker run nginx -p 80:80
：前是宿主机端口，后是容器端口

也有一种简单的情况
--net=host
容器直接共享宿主机网络

当然这会有风险

这部分知识涉及到计算机网络，一般来说并不是常用，且直接共享宿主机网络暂时不知道会出现什么问题，因此不予理会
```

3. **挂载卷**


> 在删除容器时i，容器的数据也会被删除，但是我们如果进行了挂载卷，就相当于把容器的某个文件夹和宿主机的文件夹进行了绑定，则该文件夹的内容会同步到宿主机的该文件夹中。
```
绑定挂载（与原有的宿主机文件夹进行绑定）

如
-v /website/html:/usr/share/nginx/html
冒号前为宿主机的目录，冒号后为容器内的目录

需要注意的是，在第一次进行绑定的时候，会将容器内的对应目录与宿主机的对应目录进行同步。也就是说如果宿主机对应目录是空目录，而容器的对应目录是非空的，则在第一次绑定之后容器的对应目录会变为空的。
```

```
命名卷挂载

docker volume create <挂载卷名字>------可以创建一个挂载卷
docker volume inspect <volume_name>-----查看挂载卷的具体信息，包括目录位置

-v <volume_name>:/usr/share/nginx/html------此时可以使用挂载卷名字进行挂载

命名卷第一次挂载时，会与容器内的对应目录进行同步，也就是不会把容器内对应目录进行清空，这也是命名卷的特别功能

```

==其实我并不知道有哪些文件夹要进行挂载，这方面知识后续再说==

### DockerFile

> DockerFile文件为你在别的镜像的基础上创建自己的镜像提供了途径，如果将你创建的镜像上传到docker的仓库中，那么别人就可以直接下载你制作的镜像进行使用

```

FROM osrf/ros:melodic-desktop-full #表示镜像是基于哪个原有的镜像创建的
 
#GPU的支持配置 
ENV NVIDIA_VISIBLE_DEVICES \
    ${NVIDIA_VISIBLE_DEVICES:-all}
 
ENV NVIDIA_DRIVER_CAPABILITIES \
    ${NVIDIA_DRIVER_CAPABILITIES:+$NVIDIA_DRIVER_CAPABILITIES,}graphics

#安装一系列的必要依赖 
RUN apt-get update && \
    apt-get install -y \
    build-essential \
    libgl1-mesa-dev \
    libglew-dev \
    libsdl2-dev \
    libsdl2-image-dev \
    libglm-dev \
    libfreetype6-dev \
    libglfw3-dev \
    libglfw3 \
    libglu1-mesa-dev \
    freeglut3-dev \

```

`docker build -t <image_name> .`-t后为镜像名，'.'表示在当前文件夹下创建
```
docker login #登陆docker
docker push <image_name>

这里的image_name必须完整，比如zngyim/luminavi:tag

由于一些原因未能上传成功，但这不是当前工作重点，于是不再深入研究
```










