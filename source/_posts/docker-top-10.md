---
title: Docker 1.13 新特性top10
date: 2017-01-13 16:34:04
tags:
- docker
categories: 其他

---

> 原文：https://blog.nimbleci.com/2016/11/17/whats-coming-in-docker-1-13/
翻译：laiyuXuan
学习docker过程中看到这篇文章，介绍了docker1.13中的新功能，觉得不错就翻译了过来，若有错漏欢迎拍砖。

Docker 1.13 马上就要发布了，下面是最值得期待的新功能top 10:

* Docker stacks 结束实验阶段，正式发布
从docker-compose文件部署docker stacks
* 密钥管理
* 新加入针对swarm集群的 ‘-attachable’ 参数
* 插件功能结束实验阶段，正式发布
* 通过docker daemon -experimental来开启处于实验阶段的功能
* 在service update中 加入 -force参数
* 在docker service create中加入-port参数
* 在创建镜像时缓存层(layers)
* 其他改进

### Docker stacks 结束实验阶段，正式发布

如果你有关注docker的最新发展，那么你应该知道”dab”（distributed application bundle）文件。他可以从一个docker-compose.yml文件生成，用来定义一个docker swarm集群的一个或者多个services。对于自动化部署来说这是非常便利的，因为不再需要通过docker service create和docker service update命令来更新集群中service的定义。

**延伸阅读**[Docker Stacks and Why We Need Them](https://blog.nimbleci.com/2016/09/14/docker-stacks-and-why-we-need-them/)

随着docker 1.13的发布，docker stacks 现已结束实验阶段，所有人都可以用上这个功能。

### 从docker-compose文件部署docker stacks

以往若需要使用docker stack deploy命令，我们必须先生成一个.dab文件。在docker 1.13中，不再需要把docker-compose.yml文件转换成一个.dab文件，直接使用docker-compose.yml即可。下面是具体用法：

```powershell
$ docker stack deploy \
    --compose-file ./docker-compose.yml \
    mystack
```
这样就可以在swarm集群中通过docker-compose.yml中的定义创建/更新services。


# 密钥管理

密钥管理让我们可以把密钥保存到swarm集群中，并给相应的services分配使用特定密钥的权限。

四个新加入的指令：

* docker secret create
* docker secret insepect
* docker secret ls
* docker secret rm 

举个栗子，首先把一个密钥加入到swarm集群中：

```powershell
$ echo "my-sensitive-password" | docker secret create login-password

```

然后启动一个带密钥的service

```powershell
$ docker service create \
    --name myapp \
    --secret login-password \
    ubuntu
```

这个密钥会被挂载到容器内的``/run/secrets/login-password``路径下

```powershell
$ docker exec -it myapp cat /run/secrets/login-password
my-sensitive-password
```

现在我们可以使用这个功能来给运行中的services提供密钥，但是仅限docker service模式下。通过docker run来启动的容器暂时不支持该功能，不过已经在开发中了。同样在开发中的还有在创建时加入密钥的功能。

更多详情可以查看添加了这个功能的[PR](https://github.com/docker/docker/pull/27794)


### 新加入针对swarm集群的 ‘-attachable’ 参数

新加入到docker network create中的--attachable参数可以把通过docker run启动的单个容器附属到swarm集群的网络中，从而与集群中的services进行通信。在以往这是没办法做到的，这对于需要和swarm集群中的services进行通信的单一任务来说非常便利，比如备份、手动debug。

下面是具体的用法：

```powershell
$ docker network create \
    --driver overlay \
    --attachable \
    my-attachable-network

```

接下来可以以下使用``docker run``指令连接到这个网络：

```powershell
$ docker run --rm -it \
    --net my-attachable-network \
    ping google.com
```

引入这个功能的PR可以参考[这里](https://github.com/docker/docker/pull/25962)

### 插件功能结束实验阶段，正式发布

Docker的插件功能之前一直处于实验阶段，在docker 1.13 版本中终于结束实验阶段，所有用户都可以使用这个功能了。

插件功能使得其他应用可以嵌入docker，增强docker的功能。当当前存在的插件如：Flocker， Weave

这里可以找到更多的插件。

### 通过``docker daemon -experimental``来开启处于实验阶段的功能

现在稳定的docker版本中会同时包含了尚处于实验阶段的功能。以往若要使用这些功能需要另外下载一个特定的版本，现在我们只需要在启动docker daemon的时候加上--experimental参数即可。

引入这个功能的PR可以参考[这里](https://github.com/docker/docker/pull/27223)


### 在``service update``中 加入`` -force``参数

如果你在非开发环境使用 docker 1.12 版本的swarm 集群，可能会有一些问题，比如当有一个新的节点加入到集群中，是没有办法重新分配swarm集群中的services的。

新加入的``--force``参数会强制重新拉取镜像以及重启容器。只要设置了``--update-parallelism 1``和``--update-deplay 5s``，那么每个容器会逐个重启，并且每个容器之间会有5s的间隔，保证无缝更新（Rolling updates）。

Note：以往使用``docker service update --image myimage:latest``命令时会有一个bug，无法拉取到最新的tag的镜像。这个bug已经被修复了，所以只要运行``service update``命令即可让镜像更新到最新的版本（无需加上``--force``参数）

引入这个功能的PR可以参考[这里](https://github.com/docker/docker/pull/27596)


### 在``docker service create``中加入``-port``参数

在``docker service create``中新加入的``--port``参数是用来取代``--publish``参数的。
旧的用法如下：

```powershell
$ docker service create --publish 8080:80 my-service

```

如上所示，通过这样可以把swarm集群中每个节点里外部可以访问的8080端口映射到容器内的80端口。

``--port``参数遵循了“csv”格式规范（参考``--mount``参数）。这样使得可以在一个``--port``参数中进行多项设定，而不是分别使用多个参数。下面我们来看一下用``--port``参数进行设定的一个栗子：

```powershell
$ docker service create \
    --name my-service \
    --port mode=ingress,target=80,published=8080,protocol=tcp

```

这种把多项设定放在一个参数内进行的语法显得更加灵活以及更加清晰。

### 在创建镜像时缓存层(layers)
对于时常在不同的服务器上构建镜像的情况，这是一个持续集成上重要的更新。 目前为止并没有一个比较简便的方法可以在多个服务器上共享镜像的构造层（build layers），导致每一次构建镜像时都需要把所有层都构建一遍。在近期版本的docker中，处于安全问题的考虑，一次简单的拉取是无法拉取到缓存的。因为我们无法确保我们所拉取回来的镜像就是我们所期望的那个镜像，有可能已经被篡改了，具体的issuce可以参考这里（注：在早期版本的docker中，构建镜像时会出现“using cache…”，当时docker默认会使用缓存来进行镜像的构建。在最近的版本中（1.10+），这个功能被禁用了，出于以上安全性问题的考虑。在本次更新中会加入``--cache-from``参数来让使用者选择是否基于缓存构建镜像。）

具体用法：

```powershell
docker pull myimage:v1.0
docker build --cache-from myimage:v1.0 -t myimage:v1.1 .

```

引入这个功能的PR可以参考[这里](https://github.com/docker/docker/pull/26839)

### 其他改进
1.13版本修复了成吨的bug以及带来了大量的改进，下面是一些上文没有提及但是也较为重要的地方：

* Dockerfiles中的MAITAINER 指令已经被弃用了，现在可以使用labels来记录maintainer。
* 新加入了docker service logs命令来查看service中的日志（实验阶段）
* 在docker service update中加入--rollback参数来回滚到上一个版本。
* 新版本的docker inspect命令可以可以用来查看各种各样的东西
* 使用docker build --squash来合并镜像层
* 使用docker system命令来清除不再使用的容器


### 安装
通过以下命令可以使用最新的RC版本的Docker 1.13

```powershell
$ curl -fsSL https://test.docker.com/ | sh

```
（注：或直接在Docker github官网下载）

### 总结

总而言之，docker 1.13 带来了大量的新功能和bugfixes。随着``service update``命令的完善以及stacks功能的推出，在生产环境部署swarm集群也更加简便了。

