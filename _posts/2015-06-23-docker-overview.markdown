---
layout: post
title:  "Docker 101"
date:   2015-06-23 12:06:41
categories: devops rails ruby mac docker
---
写这篇文章时，使用的`Docker`版本是`1.6.1`。
对于`Docker`这样迭代极快的系统，这很重要。

> 文章的目标读者是听说过Docker但并未使用过的用户。
> 希望能够介绍一些使用情况，并鼓励越来越多的用户。

![Docker Overview](https://www.docker.com/sites/default/files/what-is-vm-diagram.png)

[Docker](https://www.docker.com) 可以打包应用，从而作为软件开发的标准单元。
静态的概念是`镜像`(Image)，动态的概念是`容器`(Container)。
是一种轻量级的虚拟化实现。

## Docker家族

Docker 社区极为活跃，变更极大。开始使用之前，最好先看看[官方文档](https://docs.docker.com/)，确认文章描述的还跟得上时代。:)

官方的主要组件有：

* Docker Engine (aka Docker) - 创建和运行 Docker 容器
* Docker Hub - 社区化的 Docker 资源。在这里可以获取和发布配置好的系统镜像。
* Docker Compose - 组合 Dockers 形成统一服务的配置工具。事实上是 [fig](http://www.fig.sh/) 的官方升级。
* Docker Registry - Docker Hub 私有版，便于团队协同。
* Docker Swarm - Docker 主机集群和容器调度。
* Docker Machine - 自动化容器管配。可以应用在本地，远程或云端的 Docker 创建。有点像 Ubuntu 整合的 MaaS，然而轻巧很多。目前还处于 beta 版本。
* Docker Mirror - Docker 提供了工具缓存 Docker Hub 的镜像，便于本地团队开发。

这么多是不是已经晕了？说好的小巧方便，易学易用呢？

这也不完全是 Docker 的错，发展太快了，不要说书籍，连博文都很快落后。
整理下思路，看看学了有没有用。

## Docker 可以做什么

从把玩到实际应用，Docker可以：

### 快速且标准化的建立开发环境

这点类似 Vagrant，各有利弊。Docker 更模块化。Vagrant 概念更加容易理解。
如何开始？这里是 [Mac OSX 下 Rails Docker开发案例](https://robots.thoughtbot.com/rails-on-docker)

将里面所说的 fig 换成 docker-compose 即可，语法都是一样的。

整体说来的结构如下

1. `boot2docker` 做了命令行到vm中 docker 的翻译器，可以透明的在 Mac 下使用 docker
1. docker 根据 `Dockerfile` 从`本地`或`远程` Docker Hub 中下载镜像（预定义的打包系统）并低开销的加载在 Container 中
1. `Dockerfile`中还会定义一些配置命令，用于个性化 Container 中加载的镜像
1. docker run 命令帮助用户在配置好的 container 运行程序

如果是 linux，那么基本从2开始就可以了，不用考虑 boot2docker 是什么。

其中 boot2docker 起到了 vm 的作用（基于 virtualbox），虚拟了一个 linux 系统。

将 docker 命令透明的转化到虚拟机的 docker daemon 中（因为 Docker 本来就是支持远程的）。
实际的实现基本是配置环境变量，因此非常稳定。

ruby 镜像中包括了基本的依赖，就像运行在一个标准的 linux 中，因为是公用的，大家都一样，不用担心编译需要什么包的问题。

docker 在容器中运行后，可以打包成新的镜像分发。其中还引入了版本的概念，和git很像，基本的操作包括 push/pull/commit 等

如果有什么特殊的需要，可以修改并保存在 Dockerfile 中，随 SCM 工具(git)提交，保证团队使用的一致性。

当然可以把数据库服务器应用都装在一个 image 中，但这并不 docker

docker 的哲学中，模块化是弹性的基础。

因此，我们需要 docker-compose.yml 来定义容器间的运行时关系。

docker-compose 会自动将相关的依赖注入各个 docker 容器的环境变量。正确的应用这些特性才能灵活的扩展。

还请详细参考[官方的 Rails Docker 开发参考](https://docs.docker.com/compose/rails/)

### 在生产中弹性部署

Docker 最强大的地方，在于开包即用的生产特性。

利用 Docker Swarm 快速的创建 Docker 集群，瞬间实现弹性计算。

Docker Swarm 需要连接每个 host 上运行的 Docker Daemon，将多个本地，云端，甚至虚拟主机变成一个一体的抽象虚拟机。从而灵活的实现弹性部署。

整件事情也并不复杂，只需要[一个网页的指导就可以了](https://docs.docker.com/swarm/install-w-machine/)

如果能够有效的结合运营监控系统，就可以实现自动化的弹性适性部署，感觉就像拥有了全世界。而这一切，并不需要有个多么高科技的机房。

### Demo 和产品分发

很多企业项目的上线，都特别痛苦。因为有无数的线上配置操作，长长的上线行动表让人窒息。运营工程师各个都要处女座加 AB 型，但仍然状况不断。

Docker 可以让这一切固化在版本控制的配置代码里。

即使面对爆发的流量需要紧急扩容，也可以将成本降低到最低（当然，这需要长期的实践和整体架构的支持）

Docker 也可以很容易的部署在 SaaS 服务的容器中，使用别人维护的基础设施，专注于核心业务逻辑。

也可以有效的利用计算需求时空分配不均的问题，最大限度降低门槛。

## 实际应用Docker(特别是中国)

Docker 最让人惊讶的恐怕就是在生产环境的普及，作为一个2岁的项目。在国内的京东和携程都得到了相当规模的应用。

当然，他们的吐槽也不少。好在很多问题都在快速的解决中。

我们看看入门 Docker 可能会遇到的一些困难和解决方案。

### 网速或者你懂的

Docker 镜像很难少于百兆，毕竟打包了个操作系统。网速会成为极大的问题。Docker 本地是个离散型的镜像库，因此同一镜像只需要下载一次。同时，还提供了 mirror 和 registry 两种工具或组件。

* mirror 可以帮你重定向请求到本地或第三方的加速库，避免从缓慢而你懂的 Docker Hub 下载
* registry 可以在内网存储团队镜像库，快速的分享成果。

入门时，可以应用国内的 DaoCloud 提供的免费加速镜像。注册后在控制台点`加速器`可以看到详细的说明。http://daocloud.io [官方的帮助说明在这里](http://help.daocloud.io/tutorial/DaoCloudMirrorAccelerator.html)

另外实际使用时，软件内部的依赖管理系统如 deb 也会遭遇速度的问题。所以要预先部署一些工具如 artfactory / apt-cacher 等等。

docker 官方就提供了部署 [apt-cacher工具的例子](https://docs.docker.com/examples/apt-cacher-ng/) 看看是不是比以前方便一些。

### 文档和教程

Docker 太快了，搜到的博文都很可能是过期的。建议先从官方文档看起。

另外需要注意的是，Docker 是基于 linux 的，其他平台都需要合适的虚拟机方案。

### 硬件要求

普通的开发电脑就可以。但要体验集群，需要大内存和多核 CPU ，相对于 OpenStack 等的要求，应该说很贫民。对，是贫不是平。

### 团队开发

建议先从配置文件共享做起。做到镜像共享需要更多的熟悉。

> 文/王啸
>
> 可以加微信`lazing` 或访问[http://blog.2byte.us](http://blog.2byte.us) 和我联系。
> 欢迎转载，但请保留著名和出处。


