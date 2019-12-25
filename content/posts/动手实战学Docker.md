---
title: "动手实战学Docker"
date: 2019-12-24T17:07:17+08:00
draft: false
toc: true
images:
tags: ["tutorial"]
categories: ["实验楼"]
---



# Docker 概念及基本用法

## 1. 课程说明

课程为纯动手实验教程，为了能说清楚实验中的一些操作会加入理论内容。理论内容我们不会写太多，已经有太多好文章了，会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

实验环境中可以联网，不受实验楼网络限制。

## 2. 学习方法

实验楼的 Docker 课程包含 14 个实验，每个实验都提供详细的步骤和截图。适用于有一定 Linux 系统基础，想快速上手 Docker 的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容。

如果实验开始部分有推荐阅读的材料，请务必先阅读后再继续实验，理论知识是实践必要的基础。

## 3. 本节内容简介

本实验中我们初步接触 Docker 的概念和基本用法。需要依次完成下面几项任务：

1. Docker 基本概念
2. 安装 Docker
3. Docker 运行 Hello World

## 4. 推荐阅读

本节实验推荐先阅读下述内容：

- 4.1 [深入浅出 Docker（一）：Docker 核心技术预览](http://www.infoq.com/cn/articles/docker-core-technology-preview)

这篇文章介绍了 Docker 产生的技术发展历程，Docker 中的核心技术以及相关的子项目，非常好的入门资料。

- 4.2 [Understand the architecture](https://docs.docker.com/engine/understanding-docker/)

这篇 Docker 官方的文章详细介绍了 Docker 的运行机制和必要的组件。不涉及到很底层的技术，可以做为对 Docker 的一个初步了解。

## 5. Docker 概念

### 5.1 容器技术

`Linux` 容器技术很早就有了，比较有名的是被集成到主流 `Linux` 内核中的 `LXC` 项目。容器通过对操作系统的资源访问进行限制，构建成独立的资源池，让应用运行在一个相对隔离的空间里，同时容器间也可以进行通信。

容器技术对比虚拟机技术，容器比虚拟化更轻量级，对资源的消耗小很多。容器操作也更快捷，启动和停止都要比虚拟机快。但容器需要与主机共享操作系统内核，不能像虚拟机那样运行独立的内核。

`Docker` 是一个基于 `LXC` 技术构建的容器引擎，基于 `GO` 语言开发，遵循 `Apache2.0` 协议开源。`Docker` 的发展得益于为使用者提供了更好的容器操作接口。包括一系列的容器，镜像，网络等管理工具，可以让用户简单的创建和使用容器。

`Docker` 支持将应用打包进一个可以移植的容器中，重新定义了应用开发，测试，部署上线的过程，核心理念就是 `Build once, Run anywhere`。它带来了快速高效的开发生命周期，构建了 `Build, Ship and Run` 流程，统一了整个开发、测试和部署的环境和流程。

`Docker` 容器技术的典型应用场景是开发运维上提供持续集成和持续部署的服务。

下面我们开始介绍 `Docker` 中的几个基本概念。

### 5.2 镜像

`Docker` 的镜像概念类似于虚拟机里的镜像，是一个只读的模板，一个独立的文件系统，包括运行容器所需的数据，可以用来创建新的容器。

镜像可以基于 `DockerFile` 构建，`DockerFile` 是一个描述文件，里面包含若干条命令，每条命令都会对基础文件系统创建新的层次结构。

用户可以通过编写 `DockerFile` 创建新的镜像，也可以直接从类似 `GitHub` 的 `Docker Hub` 上下载镜像使用。

### 5.3 容器

`Docker` 容器是由 `Docker` 镜像创建的运行实例。`Docker` 容器类似虚拟机，可以支持的操作包括启动，停止，删除等。每个容器间是相互隔离的，但隔离的效果比不上虚拟机。容器中会运行特定的应用，包含特定应用的代码及所需的依赖文件。

在 `Docker` 容器中，每个容器之间的隔离使用 `Linux` 的 `CGroups` （`Control Groups`）和 `Namespace` 技术实现的。CGroups（控制组）提供了资源控制： CPU、内存、磁盘等资源的访问限制。Namespaces （命名空间）提供了系统资源的隔离：进程、网络、文件系统等。

### 5.4 仓库

如果你使用过 `Git` 和 `GitHub` 就很容易理解 `Docker` 的仓库概念。`Docker` 仓库相当于一个 `GitHub` 上的代码库。

`Docker` 仓库是用来包含镜像的位置，`Docker` 提供一个注册服务器（`Registry`）来保存多个仓库，每个仓库又可以包含多个具备不同 `tag` 的镜像。`Docker` 运行中使用的默认仓库是 `Docker Hub` 公共仓库。

仓库支持的操作类似 `Git`，创建了新的镜像后，我们可以 `push` 提交到仓库，也可以从指定仓库 `pull` 拉取镜像到本地。

## 6. 安装

`Docker` 有两个版本，`Community Edition(CE)` 和 `Enterprise Edition(EE)`。即社区版和企业版本。我们将介绍 Ubuntu 中社区版的安装过程。

> 在实验环境中，已经安装了 docker-ce。

### 6.1 设置存储库

首先更新 `apt` 软件包数据库，以确保软件包列表是最新的：

```bash
$ sudo apt-get update
```

> 执行 `sudo` 时可能会出现 “sudo: 无法解析主机: xxxxxx” 这样的提示，这是因为云主机的 `hostname` 不是 `localhost`，而在 `/etc/hosts` 中定义了 `127.0.0.1 localhost`，这个提示可以忽略。如果不想要终端出现这样的提示，可以执行以下命令 `sudo hostname localhost`。

安装一些软件包，以允许 `apt` 通过 `HTTPS` 使用存储库：

```bash
$ sudo apt-get -y install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
```

> 这里的 `\` 是换行的意思，当我们输入的命令太长，不方便看时，可以用 `\` 将命令分行输入。另外 `$` 是提示符，输入命令的时候不需要输入这个符号。

这里我们使用阿里云提供的源，先添加相应的密钥：

```bash
$ curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
```

再添加相应源的信息：

```bash
$ sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
```

### 6.2 安装 docker-engine

查看此时 `docker` 的版本信息:

```bash
# 更新 apt 索引库
$ sudo apt-get update

# 查看可用的版本
$ sudo apt-cache madison docker-engine
```

更新版本的 `docker` 软件包为 `docker-ce`。我们环境中已经有 docker 了，可以先移除再进行安装。

```bash
$ sudo apt-get remove docker docker-engine docker.io
```

最后我们执行安装命令，我们这里是安装的 17.05 版本的：

```bash
$ sudo apt-get install docker-engine=17.05.0~ce-0~ubuntu-trusty 
```

> 如果要安装最新版，可使用 `sudo apt-get install docker-ce`。

在安装成功后，`Docker` 的守护进程自动启动，不需要手动启动服务。

此时，我们可以查看其版本信息，使用如下命令：

```bash
$ docker version
```

当前实验环境中的 `docker` 版本信息如下图所示：

<img src="https://doc.shiyanlou.com/document-uid377240labid4104timestamp1515563110757.png">

在上图中，该命令能够正确执行，但是如果是**自己搭建的 `docker` 环境**，可能会提示我们没有相应的权限连接到 `Docker` 守护进行绑定的 `Unix` 套接字。这是因为，默认情况下，该套接字归属于 `root` 用户，对于其它用户只能通过 `sudo` 来进行访问。

如果要让 `shiyanlou` 用户可以直接执行 `docker` 命令而不必在每次执行时都输入 `sudo` 来获得权限，我们可以将要执行 `docker` 命令的用户添加到用户组 `docker` 中。该用户组会在安装后自动创建，我们只需执行添加用户到 `docker` 用户组的操作（在实验楼的在线实验环境中已完成该设置项）：

```bash
$ sudo gpasswd -a shiyanlou docker

```

添加用户到一个用户组中的方式有很多，例如我们还可以使用如下命令：

```bash
$ sudo usermod -aG docker shiyanlou

```

在添加成功后，我们还需要重新开始一个 `shell` 修改才能生效。这时可以尝试打开一个新的终端或者使用如下命令：

```bash
$ sudo su shiyanlou

```

### 6.3 启动 Docker 服务

对于 `Docker` 的镜像仓库来说，国内访问速度较慢，我们添加一个阿里云提供的 `Docker` 镜像加速器。

首先，我们需要编辑 `/etc/docker/daemon.json` 文件：

```bash
$ sudo vi /etc/docker/daemon.json

```

然后加入如下内容：

```bash
{
  "registry-mirrors": ["https://n6syp70m.mirror.aliyuncs.com"]
}

```

修改之后，需要重启 `docker` 服务，让修改生效。使用如下命令：

```bash
$ sudo service docker restart

```

## 7. Hello world

在安装之后，我们可以通过运行一个 `hello-world` 的镜像来验证 `Docker CE` 是否被正确的安装，使用如下命令：

```bash
$ docker container run hello-world

```

该命令会下载一个名为 `hello-world` 的镜像并运行于一个容器中。当这个容器运行时，会输出一些信息并退出:

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1515563587185.png)

如上图中标注出的提示信息所示，提示我们安装正确。

## 8. 总结

1. Docker 基本概念
2. 安装 Docker
3. Docker 运行 Hello World

本节实验主要讲解了 docker 的相关概念以及如何安装 docker。



# 容器管理

## 1. 课程说明

课程为纯动手实验教程，为了能说清楚实验中的一些操作会加入理论内容。理论内容我们不会写太多，已经有太多好文章了，会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

实验环境中可以联网，不受实验楼网络限制。

## 2. 学习方法

实验楼的 Docker 课程包含 14 个实验，每个实验都提供详细的步骤和截图，适用于有一定 Linux 系统基础，想快速上手 Docker 的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容。

如果实验开始部分有推荐阅读的材料，请务必先阅读后再继续实验，理论知识是实践必要的基础。

## 3. 本节内容简介

容器是 Docker 的一个基本概念，每个容器中都运行一个应用并为该应用提供完整的运行环境。本实验将详细学习 Docker 容器的创建，运行管理操作。需要依次完成下面几项任务：

1. 容器命令基础
2. 创建容器
3. 容器的启动与停止
4. 容器中进程的暂停与恢复
5. 查看容器列表
6. 连接到容器中
7. 查看元数据
8. 显示进程信息
9. 查看文件修改
10. 容器中执行命令
11. 删除容器

对于 `Docker` 的镜像仓库来说，国内访问速度较慢，我们添加一个阿里云提供的 `Docker` 镜像加速器。

首先，我们需要编辑 `/etc/docker/daemon.json` 文件：

```bash
$ sudo vi /etc/docker/daemon.json

```

然后加入如下内容：

```bash
{
  "registry-mirrors": ["https://n6syp70m.mirror.aliyuncs.com"]
}

```

修改之后，需要重启 `docker` 服务，让修改生效。使用如下命令：

```bash
$ sudo service docker restart

```

## 4. docker 命令

### 4.1 查看系统信息

除了查看版本信息之外，在 `docker` 的命令组中还有一个较为常用的命令，查看系统的一些相关信息：

```bash
$ docker system info

# 或者使用命令

$ docker info

```

运行截图如下所示：

![图片描述](https://doc.shiyanlou.com/courses/uid977658-20190426-1556273816272)

### 4.2 help

我们可以直接通过 `help` 或者使用 `man` 手册的方式查看相关命令的详细说明，例如我们直接使用如下命令:

```bash
$ docker --help

```

我们可以看到运行结果如下图所示。如果之前有学习过 docker 相关知识的同学，可能会发现一些不一样的地方。即下图中标出的 `Management commands` 和 `Commands`。在 1.13 版本之前，`docker` 并没有 `Mangement commands`。

![图片描述](https://doc.shiyanlou.com/courses/uid977658-20190426-1556273844702)

### 4.3 Management Commands

在 `Docker 1.12 CLI` 中大约有四十个左右的顶级命令，这些命令没有经过任何组织，显得十分混乱，对于新手来说，学习它们并不轻松。

而在 `Docker 1.13` 中将命令进行分组，就得到如上图中所示的 `Management Commands`。例如经常使用的容器的一些相关命令：

```bash
# 创建一个新的容器，下面分别为 Commands 和 Management Commands，作用相同
$ docker create
$ docker container create

# 显示容器列表
$ docker ps
$ docker container ls

# 在一个新的容器中运行一个命令
$ docker run
$ docker container run

...

```

如上所示，对于新的命令而言相比于旧命令明显更具有可读性。并且在实验环境中的 `docker` 版本以及最新版本中两者都是有效的命令，所以在这里我们将一些常用的命令，及其对应的 `Management Commands` 命令都列举出来，方便大家在后续的学习过程中可以进行参考。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517897142154.png)

## 4.4 命令选项

命令的选项有以下几种：长选项、短选项、复合选项、无选项。

我们以 `docker container ls` （即 `docker ps`，它的作用是查看所有）

可以先输入以下命令，获得提示信息：

```bash
$ docker container ls --help

```

命令执行后的结果如下：

![图片描述](https://doc.shiyanlou.com/courses/uid977658-20190426-1556273860565)

可以看到图中的 `-a` 和 `--all` 选项，他们的作用都是显示所有容器（包括未运行的容器）。短选项是以一个 `-` 开头，其后紧跟上一个字母或数字，比如 `-a`。长选项是以两个 `-` 开头，其后紧跟一个单词，比如 `--all`。

```bash
# 使用短选项
$ docker container ls -a

# 使用长选项
$ docker container ls --all

# 它们两者的功能是一样的，只是不同的写法

```

那什么是复合选项呢？当我们要使用多个短选项时，比如使用 `-a` 和 `-q`（从图中可以看到，这个选项只会显示容器的 ID），原本命令应该是 `docker container ls -a -q`，这时我们可以简写为 `docker container ls -aq`，这里的选项 `-aq` 就是复合选项，它是 `-a` 和 `-q` 的复合。

对于 `docker container ls` 这个命令，我们还可以不使用选项（没有选项时，输出正在运行的容器）。

关于 `docker container ls` 在本实验后面的内容中还会详细介绍，我们只需要记住它的简单功能就可以了。

## 5. 容器生命周期管理

### 5.1 创建容器（1）

首先，我们回顾在上一节使用到的 `docker container run hello-world` 命令，该命令的格式为：

```bash
# Management Commands
$ docker container run [OPTIONS] IMAGE [COMMAND [ARGS...]]

# 旧命令格式如下：
$ docker run [OPTIONS] IMAGE [COMMAND [ARGS...]]

```

上述两个命令的作用相同，`docker container run` 命令会在指定的镜像 `IMAGE` 上创建一个可写的容器（因为镜像是只读的），然后开始运行指定的命令 `[COMMAND [ARGS...]]`。

一些常用的配置项为：

- `-i` 或 `--interactive`， 交互模式
- `-t` 或 `--tty`， 分配一个 `pseudo-TTY`，即伪终端
- `--rm` 在容器退出后自动移除
- `-p` 将容器的端口映射到主机
- `-v` 或 `--volume`， 指定数据卷

> 这些配置项对于上述的两个命令（`Management Commands` 和旧命令）都是有效的，在后面的内容不会再特殊说明。

> 关于该命令的详细参数较多，并且大多数参数在很多命令中的意义是相同的，将在后面的内容中使用到时进行相应的介绍。

我们指定 `busybox` 镜像，然后运行命令 `echo "hello shiyanlou"` 命令，如下所示：

```bash
$ docker container run \
    busybox echo "hello shiyanlou"

```

![图片描述](https://doc.shiyanlou.com/courses/uid977658-20190426-1556273902835)

在上图中，我们可以看到该命令执行的过程：

1. 对于指定镜像而言，首先会从本地查找，找不到时将会从镜像仓库中下载该镜像
2. 镜像下载完成后，通过镜像启动容器，并运行 `echo "hello shiyanlou"` 命令，输出运行结果之后退出。

在执行命令之后，容器就会退出，如果我们需要一个保持运行的容器，最简单的方法就是给这个容器一个可以保持运行的命令或者应用，比如 `bash`，例如我们在 `ubunutu` 容器中运行 `/bin/bash` 命令：

```bash
$ docker container run \
    -i -t \
    ubuntu /bin/bash

```

对于交互式的进程而言（例如这里的 bash），必须将 `-i` 和 `-t` 参数一起使用，才能为容器进程分配一个伪终端，通常我们会直接使用 `-it`。

![图片描述](https://doc.shiyanlou.com/courses/uid977658-20190426-1556273889598)

如上所示，我们已经进入到分配的终端中了，这时如果我们需要退出 `bash`，可以使用以下两种方式，它们的效果完全不同：

1. 直接使用 `exit` 命令，这时候 `bash` 程序终止，容器进入到停止状态。
2. 使用组合键退出，容器仍然保持运行的状态，可以再次连接到这个 `bash` 中，组合键是 `ctrl + p` 和 `ctrl +q`。即先同时按下 `ctrl` 和 `p` 键，再同时按 `ctrl` 和 `q` 键，就可以让容器在后台运行。

对于刚刚创建创建的容器，我们输入 `exit` 退出容器，再使用 `docker container ls -a` 查看容器的状态，结果如下图所示：

![图片描述](https://doc.shiyanlou.com/courses/uid977658-20190426-1556273951479)

然后我们新建一个容器，使用第二种方式退出，创建的方式和刚刚相同。使用 `docker container ls -a` 命令查看容器的状态，可以看到该容器仍然处于运行中：

![图片描述](https://doc.shiyanlou.com/courses/uid977658-20190426-1556273937853)

实际使用中，我们不必创建容器之后使用组合键 `ctrl + p` 和 `ctrl +q` 来让容器进入后台运行，通常以 `-d` 参数指定容器以后台模式运行：

```bash
# 以后台模式创建并运行一个容器
$ docker container run \
    -i -t -d \
    ubuntu /bin/bash

```

使用 `docker container ls -a` 命令查看容器的状态，可以看到这个容器已经在后台运行。

![图片描述](https://doc.shiyanlou.com/courses/uid977658-20190426-1556273962212)

### 5.2 创建容器（2）

严格意义上来讲，`docker run` 命令的作用并不是创建一个容器，而是在一个新的容器中运行一个命令。而用于创建一个新容器的命令为

```bash
# Management Commands
$ docker container create [OPTIONS] IMAGE [COMMAND] [ARG...]

# 旧的命令格式如下：
$ docker create [OPTIONS] IMAGE [COMMAND] [ARG...]

```

该命令会在指定的镜像 `IMAGE` 上创建一个可写容器层，并 **准备** 运行指定的命令。需要着重强调的是，这里是准备运行，并不是立即运行。即该命令只创建容器，并不会运行容器。

一些常见的配置项如下所示：

- `--name` 指定一个容器名称，未指定时，会随机产生一个名字
- `--hostname` 设置容器的主机名
- `--mac-address` 设置 `MAC` 地址
- `--ulimit` 设置 Ulimit 选项

> 关于上述提到的 `ulimit`，我们可以通过其对容器运行时的一些资源进行限制。`ulimit` 是一种 `linux` 系统的内建功能，一些简单的描述，可以参考 https://www.ibm.com/developerworks/cn/linux/l-cn-ulimit/ ，而对于在下面我们将要设置的部分值的含义，可以参考 https://access.redhat.com/solutions/61334 。

除此之外，关于创建容器，我们还可以设置有关存储和网络的详细内容，将会在下一节的内容中进行介绍。

如下示例，我们指定容器的名字为 `shiyanlou01`，主机名为 `shiyanlou01`，设置相应的 `MAC` 地址，并通过 `ulimit` 设置最大进程数（`1024:2048` 分别代表软硬资源限制，详细内容可以参考上面的链接），使用 `ubuntu` 的镜像，并运行 `bash`：

```bash
$ docker container create \
    --name shiyanlou01 \
    --hostname shiyanlou01 \
    --mac-address 00:01:02:03:04:05 \
    --ulimit nproc=1024:2048 \
    -it ubuntu /bin/bash

```

![图片描述](https://doc.shiyanlou.com/courses/uid977658-20190426-1556273978908)

此时，容器创建成功后，会打印该容器的 `ID`，这里需要简单说明一下，在 `docker` 中，容器的标识有三种比较常见的标识方式：

- `UUID` 长标识符，例如 `1f6789f885029dbdd4a6426d7b950996a5bcc1ccec9f8185240313aa1badeaff`
- `UUID` 短标识符，从长标识符开始，只要不与其它标识符冲突，可以从头开始，任意选用位数，例如针对上面的长标识符，可以使用 `1f`，`1f678` 等等
- `Name` 最后一种方式即是使用容器的名字

在容器创建成功后，我们可以查看其运行状态，使用如下命令：

```bash
# 此时该容器并未运行，需要使用 -a 参数
$ docker container ls -a

```

![图片描述](https://doc.shiyanlou.com/courses/uid977658-20190426-1556273971432)

新创建的容器的状态 (`STATUS`) 为 `Created`，并且其容器名被设置为对应的值，而之前没有指定名字的容器都是随机生成的名字。

### 5.3 容器的启动与停止

#### 启动容器操作

容器的启动命令为：

```bash
# Management Commands
$ docker container start [OPTIONS] CONTAINER [CONTAINER...]

# 旧的命令格式如下：
$ docker start [OPTIONS] CONTAINER [CONTAINER...]

```

对于上面我们创建的容器 `shiyanlou01` 而言，此时处于 `Created` 状态，需要使用如下命令启动它：

```bash
$ docker container start shiyanlou01

```

![图片描述](https://doc.shiyanlou.com/courses/uid977658-20190426-1556273986407)

此时，运行一个容器我们分成了两个步骤，即创建和启动，使用的命令如下：

```bash
# 创建
$ docker container create \
    --name shiyanlou01 \
    --hostname shiyanlou01 \
    --mac-address 00:01:02:03:04:05 \
    --ulimit nproc=1024:2048 \
    -it ubuntu /bin/bash

# 启动
$ docker container start shiyanlou01

```

上述的两个命令如果我们使用 `docker container run` 只需要一步即可，即此时 `run` 命令同时完成了 `create` 及 `start` 操作：

```bash
$ docker container run \
    --name shiyanlou01 \
    --hostname shiyanlou01 \
    --mac-address 00:01:02:03:04:05 \
    --ulimit nproc=1024:2048 \
    -it ubuntu /bin/bash

```

> 除此之外，上面的 `run` 命令还完成一些其它的操作，例如没有镜像时会 `pull` 镜像，使用 `-it` 参数时完成了 `attach` 操作（后面会学习该操作），使用 `--rm` 参数在容器退出后还会完成 `container rm` 操作。

> `run` 命令是一个综合性的命令，如果能够熟练的使用它可以简化很多步骤，但是其使用方式较为复杂

#### 停止容器操作

停止容器可以使用如下命令：

```bash
# Management Commands
$ docker container stop CONTAINER [CONTAINER...]

# 旧的命令格式如下：
$ docker stop CONTAINER [CONTAINER...]

```

刚刚我们启动了一个名为 `shiyanlou01` 的容器，并且进入了交互式界面，这里我们先同时按下 `ctrl` 和 `p` 键，再同时按 `ctrl` 和 `q` 键，让这个容器进入到后台运行。

此时我们使用 `docker container ls -a` 命令查看容器的状态，从下图可以看到，容器正在运行。输入 `docker container stop shiyanlou01`，docker 返回了容器的 UUID，再次使用 `docker container ls -a` 命令查看容器的状态，发现容器停止运行了。

![图片描述](https://doc.shiyanlou.com/courses/uid977658-20190426-1556273993199)

#### 重启容器操作

重启容器可以使用如下命令：

```bash
# Management Commands
$ docker container restart CONTAINER [CONTAINER...]

# 旧的命令格式如下：
$ docker restart CONTAINER [CONTAINER...]

```

这里我们重启刚刚停止的容器 `shiyanlou01`，然后再使用 `docker container ls -a` 命令查看容器，从下图中可以看到，容器又处于运行状态了。

![图片描述](https://doc.shiyanlou.com/courses/uid977658-20190426-1556273999632)

### 5.4 进程的暂停与恢复

#### 暂停进程操作

暂停容器中进程的命令格式如下：

```bash
# Management Commands
$ docker container pause CONTAINER [CONTAINER...]

# 旧的命令格式如下：
$ docker pause [OPTIONS] CONTAINER [CONTAINER...]

```

这里还是使用 `shiyanlou01` 这个容器，执行下述命令。

```bash
$ docker container pause shiyanlou01
$ docker container ls -a

```

如下图所示，容器被暂停后，此时处于 `Paused` 状态。

![图片描述](https://doc.shiyanlou.com/courses/uid977658-20190426-1556274007755)

#### 恢复进程操作

恢复容器中进程的命令格式如下：

```bash
# Management Commands
$ docker container unpause CONTAINER [CONTAINER...]

# 旧的命令格式如下：
$ docker unpause [OPTIONS] CONTAINER [CONTAINER...]

```

使用 `shiyanlou01` 这个容器，执行下述命令。

```bash
# 恢复容器中的进程
$ docker container unpause shiyanlou01

# 查看容器列表
$ docker container ls -a

```

如下图所示，容器恢复后，此时处于运行状态。

![图片描述](https://doc.shiyanlou.com/courses/uid977658-20190426-1556274015270)

## 6. 其他容器操作

### 6.1 查看容器列表

查看容器列表可以使用如下命令：

```bash
# Management Commands
$ docker container ls [OPTIONS]

# 旧的命令格式如下：
$ docker ps [OPTIONS]

```

在使用命令时，我们可以使用一些可选的配置项 `[OPTIONS]`。

- `-a` 显示所有的容器
- `-q` 仅显示 `ID`
- `-s` 显示总的文件大小

默认情况下，直接使用该命令仅显示正在运行的容器，如下所示：

```bash
$ docker container ls

```

![图片描述](https://doc.shiyanlou.com/courses/uid977658-20190426-1556274022838)

我们可以使用 `-a` 参数，来显示所有的容器，并加上 `-s` 选项，显示大小，命令如下：

```bash
$ docker container ls -a -s

```

![图片描述](https://doc.shiyanlou.com/courses/uid977658-20190426-1556274029366)

### 6.2 连接到正在运行中的容器

上述操作我们启动的容器运行于后台，所以，我们需要使用 `attach` 操作将本地标准输入输出流连接到一个运行中的容器，命令格式为：

```bash
# Management Commands
$ docker container attach [OPTIONS] CONTAINER

# 旧的命令格式如下：
$ docker attach [OPTIONS] CONTAINER

```

如下示例，我们启动容器，并使用连接命令：

```bash
$ docker container start shiyanlou01

$ docker container attach shiyanlou01

```

连接到容器后，查看相应的主机名（输入 `hostname` 命令）和 `Mac` 地址（输入 `ifconfig` 命令），可以判断我们连接到了刚刚创建的容器。

![图片描述](https://doc.shiyanlou.com/courses/uid977658-20190426-1556274036913)

### 6.3 查看容器的元数据

查看容器的详细信息（即元数据）可以使用如下命令：

```bash
# Management Commands
$ docker container inspect [OPTIONS] CONTAINER [CONTAINER...]

# 旧的命令格式如下：
$ docker inspect [OPTIONS] CONTAINER [CONTAINER...]

```

例如我们查看刚刚创建的容器的详细信息就可以使用以下命令：

```bash
# 使用容器名
$ docker container inspect shiyanlou01

# 使用 ID ，因生成的 ID 不同，需要修改为相应的 ID
$ docker container inspect 1f6789

$ docker container inspect 1f6

```

例如，我们查看刚刚创建的名为 `shiyanlou01` 的容器的 `MAC` 地址，就可以使用如下命令：

```bash
$ docker container inspect shiyanlou01 | grep "MacAddress"

```

![图片描述](https://doc.shiyanlou.com/courses/uid977658-20190426-1556274043690)

### 6.4 容器的日志管理

获取容器的输出信息可以使用如下命令：

```bash
# Management Commands
$ docker container logs [OPTIONS] CONTAINER

# 旧的命令格式如下：
$ docker logs [OPTIONS] CONTAINER

```

常用的配置项有：

- `-t` 或 `--timestamps` 显示时间戳
- `-f` 实时输出，类似于 `tail -f`

这里我们重新运行一个容器，让它在后台执行一个不断输出的脚本，命令如下：

```bash
$ docker container run \
    --name shiyanlou02 \
    -i -t -d \
    ubuntu /bin/sh -c "while true; do echo hello shiyanlou; sleep 2; done"

```

> "while true; do echo hello world; sleep 2; done" 是一个脚本，它的功能是每 2 秒输出一次“ hello shiyanlou ”，此处不讲解语法构成，感兴趣的同学可以自己了解相关知识。

如下所示，我们查看刚刚创建的容器的日志，使用如下命令：

```bash
$ docker container logs -tf shiyanlou02

```

我们可以使用组合键 `Ctrl` + `c` 来结束日志跟踪，其结果如下图所示。

![图片描述](https://doc.shiyanlou.com/courses/uid977658-20190426-1556274050853)

### 6.5 显示容器中的进程信息

除了获取日志之外，还可以显示运行中的容器的进程信息，命令格式如下：

```bash
# Management Commands
$ docker container top CONTAINER

# 旧的命令格式如下：
$ docker top CONTAINER

```

例如查看刚刚创建的容器的进程信息：

```bash
$ docker container top shiyanlou02

```

![图片描述](https://doc.shiyanlou.com/courses/uid977658-20190426-1556274069660)

> 需要注意的是，该命令对于并未运行的容器是无效的

### 6.6 查看文件修改

查看相对于镜像的文件系统来说，容器中做了哪些改变，可以使用如下命令：

```bash
# Management Commands
$ docker container diff CONTAINER

# 旧的命令格式如下：
$ docker diff CONTAINER

```

我们先在 `shiyanlou01` 中创建一个文件，执行以下命令：

```bash
# 重启容器
$ docker container restart shiyanlou01

# 连接到容器中
$ docker container attach shiyanlou01

```

进入容器中后，创建一个文件，并退出：

```bash
# 创建一个文件
$ touch ~/a.txt

```

现在我们在 `shiyanlou01` 容器中创建一个文件，就可以使用 `docker container diff shiyanlou01` 命令查看到相应的修改：

![图片描述](https://doc.shiyanlou.com/courses/uid977658-20190426-1556274075675)

### 6.7 容器中执行命令

除了使用 `docker container run` 执行命令之外，我们还可以在一个运行中的容器中执行命令，使用如下格式：

```bash
# Management Commands
$ docker container exec [OPTIONS] CONTAINER COMMAND [ARG...]

# 旧的命令格式如下：

```

例如，我们在 `shiyanlou01` 容器中执行 `echo "test_exec"` 命令，就可以使用如下命令：

```bash
# 重启容器
$ docker container restart shiyanlou01

# 执行命令
$ docker container exec shiyanlou01 echo "test_exec"

```

![图片描述](https://doc.shiyanlou.com/courses/uid977658-20190426-1556274406068)

### 6.8 删除容器

删除容器的命令：

```bash
# Management Commands
$ docker container rm [OPTIONS] CONTAINER [CONTAINER...]

# 旧的命令格式如下：
$ docker rm [OPTIONS] CONTAINER [CONTAINER...]

```

> 需要注意的是，在删除容器后，在容器中进行的操作并不会持久化到镜像中

如果想删除之前创建的所有容器，可以使用以下命令：

```bash
$ docker container rm -f $(docker container ls -aq)

```

`docker container ls -aq` 会输出所有容器的 UUID ，rm 命令可以根据 UUID 去删除容器。这里用来选项 `-f` 是因为还有在运行中的容器，所以需要强制删除。`ls` 列出的 UUID 传递给 `rm` 进行删除。

![图片描述](https://doc.shiyanlou.com/courses/uid977658-20190426-1556274087512)

## 7. 总结

本节实验中我们学习了以下内容：

1. 容器命令基础
2. 创建容器
3. 容器的启动与停止
4. 容器中进程的暂停与恢复
5. 查看容器列表
6. 连接到容器中
7. 查看元数据
8. 显示进程信息
9. 查看文件修改
10. 容器中执行命令
11. 删除容器

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。



# 镜像管理

## 1. 课程说明

课程为纯动手实验教程，为了能说清楚实验中的一些操作会加入理论内容。理论内容我们不会写太多，已经有太多好文章了，会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

实验环境中可以联网，不受实验楼网络限制。

## 2. 学习方法

实验楼的 Docker 课程包含 14 个实验，每个实验都提供详细的步骤和截图，适用于有一定 Linux 系统基础，想快速上手 Docker 的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容。

如果实验开始部分有推荐阅读的材料，请务必先阅读后再继续实验，理论知识是实践必要的基础。

## 3. 本节内容简介

第一节实验中我们已经接触了一些镜像的概念，简单的说镜像就是一个容器的只读模板，用来创建容器。当运行容器时需要指定镜像，如果本地没有该镜像，则会从 Docker Registry 下载。默认查找的是 Docker Hub。Docker 的镜像是增量的修改，每次创建新的镜像都会在老的镜像上面构建一个增量的`层`，使用到的技术是`Another Union File System(AUFS)`，感兴趣的同学可以学习文档 [InfoQ:剖析 Docker 文件系统：Aufs 与 Devicemapper](https://blog.csdn.net/zhaihaifei/article/details/51263232)。

本节中，我们需要依次完成下面几项任务：

1. 查看镜像列表
2. 查看镜像详细信息
3. 搜索镜像
4. 拉取镜像
5. 构建镜像
6. 删除镜像

对于 `Docker` 的镜像仓库来说，国内访问速度较慢，我们添加一个阿里云提供的 `Docker` 镜像加速器。

首先，我们需要编辑 `/etc/docker/daemon.json` 文件：

```bash
$ sudo vi /etc/docker/daemon.json

```

然后加入如下内容：

```bash
{
  "registry-mirrors": ["https://n6syp70m.mirror.aliyuncs.com"]
}

```

修改之后，需要重启 `docker` 服务，让修改生效。使用如下命令：

```bash
$ sudo service docker restart

```

## 4. 镜像

如果能够熟练的使用容器管理命令，那么这里关于镜像的部分操作，类比 `Management Commands` 的特性，可以很容易的学习相应的命令。

镜像存储中的核心概念仓库（Repository）是镜像存储的位置。Docker 注册服务器（Registry）是仓库存储的位置。每个仓库包含不同的镜像。

比如一个镜像名称 `ubuntu:14.04`，冒号前面的 `ubuntu` 是仓库名，后面的 `14.04` 是 TAG，不同的 TAG 可以对应相同的镜像，TAG 通常设置为镜像的版本号。在前面的实验中，我们的镜像都没有加 TAG，这时 Docker 会使用默认的 TAG：`latest`。

`Docker Hub` 是 `Docker` 官方提供的公共仓库，提供大量的常用镜像，由于国内网络原因经常连接 `Docker Hub` 会比较慢。

并且 `Docker` 的镜像是分层存储，每一个镜像都是由很多层组成的。而一些镜像会共享一些相同的层。对于实验环境中的 `docker` 来说，其使用的存储驱动是 `aufs`，如下图所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516930473464.png)

> 图片中显示的是 aufs，但是对于如果是在自己的 Linux 环境下安装的 docker，其版本是高于 17.05，显示的有可能是 overlay2，其基本原理和 aufs 类似。

`aufs` 是一种联合文件系统(`UnionFS`)，理解其原理对于我们理解 Docker 镜像有很大的帮助，有兴趣的同学可以尝试学习 [Linux 文件系统之 aufs](https://segmentfault.com/a/1190000008489207)

### 4.1 查看镜像列表

我们查看镜像可以使用如下命令：

```bash
# Management Commands
$ docker image ls

# 旧的命令格式如下：
$ docker images

```

也可以查看指定仓库的镜像，例如。查看 `ubuntu` 仓库的镜像：

```bash
$ docker image ls ubuntu

```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516697042412.png)

### 4.2 查看镜像的详细信息

查看镜像的详细信息使用如下命令：

```bash
# Management Commands
$ docker image inspect ubuntu

# 旧的命令格式如下：
$ docker inspect ubuntu

```

> 注意：`docker inspect` 命令可以用来查看容器的信息，也可以用来查看镜像的信息

### 4.3 搜索镜像

```bash
$ docker search ubuntu

```

这个命令会列出所有包含 ubuntu 关键字的镜像，结果如下图所示：

![图片描述](https://doc.shiyanlou.com/courses/uid977658-20190428-1556414655829)

### 4.4 拉取镜像

上面的内容中描述了仓库和注册表的内容，这里，我们学习从注册表中获得镜像或者仓库的命令，使用如下命令：

```bash
# Management Commands
$ docker image pull [OPTIONS] NAME[:TAG|@DIGEST]

# 旧的命令格式如下：
$ docker pull [OPTIONS] NAME[:TAG|@DIGEST]

```

比较常用的配置参数为 `-a`，代表下载仓库中的所有镜像，即下载整个存储库。

如下所示，我们下载 `ubuntu:14.04` 镜像，使用如下命令：

```bash
$ docker image pull ubuntu:14.04

```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516697216997.png)

对于 `pull` 下来的镜像来说，其默认的保存路径为 `/var/lib/docker`。因为这里的存储驱动为 `aufs`，所以具体路径为 `/var/lib/docker/aufs`。

除了通过标签来拉取具体的镜像， 我们还可以通过摘要来拉取不同标签的镜像。从刚刚下载的结果，我们有一行 Digest 信息，这就是摘要信息。

![图片描述](https://doc.shiyanlou.com/courses/uid977658-20190428-1556415367379)

接下来我们简单演示一下通过摘要拉取镜像：首先删除刚刚下载的镜像 `docker image rm ubuntu:14.04` （关于删除的命令会在本小节后面的部分详细讲述），此处以获取到的 Digest 为例，具体 Digest 参数应参照实际的值。

![图片描述](https://doc.shiyanlou.com/courses/uid977658-20190428-1556415702937)

### 4.5 构建镜像

#### commit

此时，对于我们 `pull` 的新镜像 `ubuntu:14.04` 来说，如果我们需要对其进行更新，可以创建一个容器，在容器中进行修改，然后将修改提交到一个新的镜像中。

提交修改使用如下命令：

```bash
# Management Commands
$ docker container commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]

# 旧的命令格式如下：
$ docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]

```

该命令的解释为从一个容器的修改中创建一个新的镜像。例如，我们运行一个容器，然后在其中创建一个文件，最后使用 `commit` 命令：

```bash
# 使用 run 创建运行一个新命令
$ docker container run \
    --name shiyanlou001 \
    -it busybox /bin/sh

# 在运行的容器中创建两个文件，test1 和 test2
touch test1 test2

# 使用 ctrl + p  及  ctrl+q 键退出

# 使用提交命令，提交容器 shiyanlou001 的修改到镜像 busybox:test 中
$ docker container commit shiyanlou001 busybox:test

# 查看通过提交创建的镜像
$ docker image ls busybox

```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516697682918.png)

> 通过上述操作我们创建了一个新的镜像，但是本方法不推荐用在生产系统中，未来会很难维护镜像。最好的创建镜像的方法是 `Dockerfile`，修改镜像的方法是修改 `Dockerfile`，然后重新从 `Dockerfile` 中构建新的镜像。

#### BUILD

`docker` 可以从一个 `Dockerfile` 文件中自动读取指令构建一个新的镜像。 `Dockerfile` 是一个包含用户构建镜像命令的文本文件。在 创建该文件后，我们可以使用如下命令构建镜像：

```bash
docker image build [OPTIONS] PATH | URL

```

> 构建镜像时，该过程的第一件事是将 `Dockerfile` 文件所在目录下的所有内容递归的发送到守护进程。所以在大多数情况下，最好是创建一个新的目录，在其中保存 `Dockerfile`，并在其中添加构建 `Dockerfile` 所需的文件。

对于一个 `Dockerfile` 文件内容来说，基本语法格式如下所示：

```bash
# Comment
INSTRUCTION arguments

```

使用 `#` 号作为注释，指令（`INSTRUCTION`）不区分大小写，但是为了可读性，一般将其大写。而 `Dockerfile` 的指令一般包含下面几个部分：

1. 基础镜像：以哪个镜像为基础进行制作，使用 `FROM` 指令来指定基础镜像，一个 `Dockerfile` 必须以 `FROM` 指令启动。
2. 维护者信息：可以指定该 `Dockerfile` 编写人的姓名及邮箱，使用 `MAINTAINER` 指令。
3. 镜像操作命令：对基础镜像要进行的改造命令，比如安装新的软件，进行哪些特殊配置等，常见的是 `RUN` 命令。
4. 容器启动命令：基于该镜像的容器启动时需要执行哪些命令，常见的是 `CMD` 命令或 `ENTRYPOINT`

例如一个最基本的 `Dockerfile`：

```
# 指定基础镜像
FROM ubuntu:14.04

# 维护者信息
MAINTAINER shiyanlou/shiyanlou001@simplecloud.cn

# 镜像操作命令
RUN \
    apt-get -yqq update && \
    apt-get install -yqq apache2

# 容器启动命令
CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]

```

通过阅读上述内容中我们熟悉的一些 `linux` 指令，可以很容易的得出该命令创建了一个 `apache` 的镜像。包含了最基本的四项信息。

其中 `FROM` 指定基础镜像。`RUN` 命令默认使用 `/bin/sh`，并使用 `root` 权限执行。`CMD` 命令也是默认在 `/bin/sh` 中执行，但是只能有一条 `CMD` 指令，如果有多条则只有最后一条会被执行。

下面我们创建一个空目录，并在其中编辑 `Dockerfile` 文件，并基于此构建一个新的镜像，使用如下操作：

```bash
# 首先创建目录并切换目录
$ mkdir /home/shiyanlou/test1 && cd /home/shiyanlou/test1

# 编辑 Dockerfile 文件，默认文件名为 `Dockerfile`，也可以使用其它值，使用其它值需要在构建时通过 `-f` 参数指定，这里我们使用默认值。并在其中添加上述示例的内容
$ vim Dockerfile

# 使用 build 命令，`-t` 参数指定新的镜像
$ docker image build -t shiyanlou:1.0 .

```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516700296627.png)

在执行构建命令后，需要花费一些时间来完成构建。在运行结束后，最后查看新创建的镜像：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516700483506.png)

在构建完成后，我们可以使用该镜像启动一个容器来运行 `apache` 服务，运行如下命令：

```bash
# 使用 -p 参数将本机的 8000 端口映射到容器中的 80 端口上。
$ docker container run \
    -d -p 8000:80 \
    --restart=always \
    --name shiyanlou002 shiyanlou:1.0

```

> 其中 `--restart=always` 选项保证容器始终不关闭，在本实验中如果不加此参数无法运行时，可以加上这个参数保证容器一直在运行。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516700663701.png)

此时，容器启动成功后，并且配置了端口映射，我们就可以通过本机的 `8000` 端口访问容器 `shiyanlou002` 中的 `apache` 服务了。我们打开浏览器，输入 `localhost:8000`，显示结果如下图：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516700754471.png)

> 如果 `localhost:8000` 无法访问，那么可以输入 `127.0.0.1:8000`。

> 更多有关于 Dockerfile 文件格式的信息可以参考官方文档 https://docs.docker.com/engine/reference/builder/

### 4.6 删除镜像

我们删除 `ubuntu:latest` 镜像就可以使用如下命令：

```bash
# Management Commands
$ docker image rm ubuntu:latest

# 旧的命令格式如下：
$ docker rmi ubuntu:latest

```

需要注意的是，如果该镜像正在被一个容器所使用，需要将容器删除才能成功的删除镜像。

## 5. 总结

本节实验中我们学习了以下内容：

1. 查看镜像列表
2. 查看镜像详细信息
3. 搜索镜像
4. 拉取镜像
5. 构建镜像
6. 删除镜像

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。



# 存储管理

## 1. 课程说明

课程为纯动手实验教程，为了能说清楚实验中的一些操作会加入理论内容。理论内容我们不会写太多，已经有太多好文章了，会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

实验环境中可以联网，不受实验楼网络限制。

## 2. 学习方法

实验楼的 Docker 课程包含 14 个实验，每个实验都提供详细的步骤和截图，适用于有一定 Linux 系统基础，想快速上手 Docker 的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容。

如果实验开始部分有推荐阅读的材料，请务必先阅读后再继续实验，理论知识是实践必要的基础。

## 3. 本节内容简介

在本节内容中，我们将讨论 `Docker` 中管理数据的几种方式。

本节中，我们需要依次完成下面几项任务：

1. 使用 volumes
2. 使用 bind mounts
3. 使用 tmpfs
4. 数据卷容器
5. 数据卷的备份与恢复

对于 `Docker` 的镜像仓库来说，国内访问速度较慢，我们添加一个阿里云提供的 `Docker` 镜像加速器。

首先，我们需要编辑 `/etc/docker/daemon.json` 文件：

```bash
$ sudo vi /etc/docker/daemon.json

```

然后加入如下内容：

```bash
{
  "registry-mirrors": ["https://n6syp70m.mirror.aliyuncs.com"]
}

```

修改之后，需要重启 `docker` 服务，让修改生效。使用如下命令：

```bash
$ sudo service docker restart

```

## 4. 存储

### 4.1 概述

通过之前的学习，我们学习了有关于容器和镜像的一些知识。对于数据来说，我们可以将其保存在容器中，但是会存在一些缺点：

- 当容器不再运行时，我们无法使用数据，并且容器被删除时，数据并不会被保存。
- 数据保存在容器中的可写层中，我们无法轻松的将数据移动到其他地方。

针对上述的缺点而言，有些数据信息，例如我们的数据库文件，我们不应该将其保存在镜像或者容器的可写层中。Docker 提供三种不同的方式将数据从 Docker 主机挂载到容器中，分别为卷（`volumes`），绑定挂载（`bind mounts`），临时文件系统（`tmpfs`）。很多时候，`volumes` 总是正确的选择。

- `volumes`， 卷存储在 Docker 管理的主机文件系统的一部分中（`/var/lib/docker/volumes/`） 中。完全由 `Docker` 管理
- `bind mounts`， 绑定挂载，可以将主机上的文件或目录挂载到容器中
- `tmpfs`， 仅存储在主机系统的内存中，而不会写入主机的文件系统

无论使用上述的哪一种方式，数据在容器内看上去都是一样的。它被认为容器文件系统中的目录或单个文件。

### 4.2 卷列表

对于三种不同的存储数据的方式来说，卷是唯一完全由 `Docker` 管理的。它更容易备份或迁移，并且我们可以使用 `Docker CLI` 命令来管理卷。

列出本地可用的卷列表可以使用如下命令：

```bash
$ docker volume ls

```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516701361387.png)

由于此时我们并未创建有相应的卷，所以显示为空。

#### 创建卷

创建卷我们可以直接使用如下命令：

```bash
$ docker volume create

```

上述命令会创建一个数据卷，并且会随机生成一个名称。创建之后我们可以查看卷列表：

```bash
$ docker volume ls

```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516701446377.png)

这种由系统随机生成名称的创建卷的方式被称为**匿名卷**，直接使用该卷需要指定卷名，即自动生成的 `ID`，所以创建卷时一般手动指定其 `name`，例如我们创建一个名为 `volume1` 的卷。

```bash
$ docker volume create volume1

```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516701557988.png)

#### 用卷启动一个容器

创建卷之后，我们可以用卷来启动一个容器，这里首先需要学习 `docker container run` 命令的两个参数：

- `-v` 或 `--volume`
  - 由三个由冒号（:）分隔的字段组成，`[HOST-DIR:]CONTAINER-DIR[:OPTIONS]`。
  - `HOST-DIR` 代表主机上的目录或数据卷的名字。省略该部分时，会自动创建一个匿名卷。如果是指定主机上的目录，需要使用绝对路径。
  - `CONTAINER-DIR` 代表将要挂载到容器中的目录或文件，即表现为容器中的某个目录或文件
  - `OPTIONS` 代表配置，例如设置为只读权限(`ro`)，此卷仅能被该容器使用（`Z`），或者可以被多个容器使用 （`z`）。多个配置项由逗号分隔。
  - 例如，我们使用 `-v volume1:/volume1:ro,z`。代表的是意思是将卷 `volume1` 挂载到容器中的 `/volume1` 目录。`ro,z` 代表该卷被设置为只读（`ro`），并且可以多个容器使用该卷（`z`）
- `--mount`
  - 由多个键值对组成，键值对之间由逗号分隔。例如： `type=volume,source=volume1,destination=/volume1,ro=true`。
  - `type`，指定类型，可以指定为 `bind`，`volume`，`tmpfs`。
  - `source`，当类型为 `volume` 时，指定卷名称，匿名卷时省略该字段。当类型为 `bind`，指定路径。可以使用缩写 `src`。
  - `destination`，挂载到容器中的路径。可以使用缩写 `dst` 或 `target`。
  - `ro` 为配置项，多个配置项直接由逗号分隔一般使用 `true` 或 `false`。

针对上述创建的卷 `volume1`，用其来运行一个容器就可以使用如下命令：

```bash
$ docker container run \
    -it \
    --name shiyanlou001 \
    -v volume1:/volume1 \
    --rm ubuntu /bin/bash

```

或者我们也可以使用 `--mount`，其语法格式如下：

```bash
$ docker container run \
    -it --name shiyanlou002 \
    --mount type=volume,src=volume1,target=/volume1 \
    --rm ubuntu /bin/bash

```

> 从命令中，可以很明显的得出，`--mount` 的可读性更好。所以，推荐大家使用 `--mount`。

> 在 `docker container run` 中我们使用了参数 `--rm`，它的作用在容器退出时删除容器。这里我们创建的镜像只是希望它短期运行，其用户数据并无保留的必要，因而可以在容器启动时设置 --rm 选项，这样在容器退出时就能够自动清理容器内部的文件系统。但自己使用时要注意：后台运行的容器无法使用此选项，即 `-d` 与 `--rm` 无法同时使用。

上述操作，我们分别运行了两个容器，并分别挂载了一个卷，还可多次使用该参数挂载多个卷或目录。并且对于这两个容器来说，由于我们使用的是同一个卷，所以他们将共享该数据卷，但是对于多个容器共享数据卷时，需要注意并发性。大家可以分别连接到两个容器中，操作数据，验证其是同步的，这里就不再详细演示了。

### 4.3 bind-mounts

对于数据卷来说，其优点在于方便管理。而对于绑定挂载（`bind-mounts`）来说，通过将主机上的目录绑定到容器中，容器就可以操作和修改主机上该目录的内容。这既是其优点也是其缺点。

例如，我们将 `/home/shiyanlou` 目录挂载到容器中的 `/home/shiyanlou` 目录下，使用的命令如下：

```bash
$ docker container run \
    -it \
    -v /home/shiyanlou:/home/shiyanlou \
    --name shiyanlou003 \
    --rm ubuntu /bin/bash

```

而如果使用的是 `--mount`，相应的语句如下：

```bash
$ docker container run \
    -it \
    --mount type=bind,src=/home/shiyanlou,target=/home/shiyanlou \
    --name shiyanlou004 \
    --rm ubuntu /bin/bash

```

> 如果绑定挂载时指定的容器目录是非空的，则该目录中的内容将会被覆盖。并且如果主机上的目录不存在，会自动创建该目录。

上述两个操作针对的是目录，而对于挂载文件来说，可能会出现一些特殊情况，涉及到绑定挂载和使用卷的区别。下面我们重现这一操作：

(1) 首先在当前目录，即 `/home/shiyanlou` 目录下，创建一个 `test.txt` 文件。并向其中写入文本内容 "test1"：

```bash
$ echo "test1" > test.txt

```

(2) 接着创建一个容器 `shiyanlou005`，将 `test.txt` 文件挂载到容器中的 `/test.txt` 文件，并查看容器中 `/test.txt` 文件的内容：

```bash
$ docker container run \
    -it \
    -v /home/shiyanlou/test.txt:/test.txt \
    --name shiyanlou005 ubuntu /bin/bash

```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516934638946.png)

(3) 这时新打开一个终端，通过 `echo` 命令向 `/home/shiyanlou/test.txt` 文件追加内容 "test2"，并在容器中查看 `/test.txt` 文件的内容:

```bash
$ echo "test2" >> test.txt

```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516934866508.png)

(4) 这时无论是在容器中还是主机上都能查看到该文件的内容。接下来在主机上查看 `test.txt` 的 `inode` 号，并使用 `vim` 编辑该文件，添加 "test3"，并查看该文件的内容：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516935268507.png)

如上图所示，在主机上使用 `vim` 编辑后，通过 `vim` 做出的修改不能在容器中查看到。这是因为 `vim` 编辑保存文件的时候，会将文件内容写入到一个新的文件中，保存好后，删除掉原来的文件，并将新文件重命名，从而完成保存的操作。但是我们标识文件是通过 `inode`，这在第一周的内容中有讲解到，因此 Docker 绑定的主机文件，依旧是 `vim` 编辑之前的 `inode`，即旧文件。所以容器中看到的，依然是旧的内容。

对于数据卷来说，由 `docker` 完全管理，而绑定挂载，则需要我们自己去维护。我们需要自己手动去处理这些问题，这些问题并不仅仅是上面演示的内容，还可能有用户权限，`SELINUX` 等问题。

### 4.4 tmpfs

`tmpfs` 只存储在主机的内存中。当容器停止时，相应的数据就会被移除。

```bash
$ docker run \
    -it \
    --mount type=tmpfs,target=/test \
    --name shiyanlou008 \
    --rm ubuntu bash

```

## 5. 数据卷容器

如果容器之间需要共享一些持续更新的数据，最简单的方式就是使用用户数据卷容器。其他容器通过挂载这个容器实现数据共享，这个挂载数据卷的容器就叫做数据卷容器。数据卷容器就是一种普通容器，它专门提供数据卷供其它容器挂载使用。

### 5.1 创建数据卷容器

首先，我们创建一个数据卷和数据卷容器，执行的命令如下：

```bash
# 创建一个名为 vdata 的数据卷
$ docker volume create vdata

# 创建一个挂载了 vdata 的容器，这个容器就是数据卷容器
$ docker container run \
    -it \
    -v vdata:/vdata 
    --name ShiyanlouVolume ubuntu /bin/bash

# 在 /vdata 目录下创建一个文本文件
$ echo "I am ShiyanlouVolume" > /vdata/f.txt

```

![图片描述](https://doc.shiyanlou.com/courses/uid977658-20190428-1556430124640)

接下来我们分别打开新的终端输入以下命令，创建两个容器，在执行 `docker container run` 时，我们添加参数 `--volumes-from` 继承数据卷容器 ShiyanlouVolume 挂载的数据卷。进入容器后分别创建文件

```bash
# 创建容器 test1
$ docker container run \
    -it \
    --volumes-from ShiyanlouVolume \
    --name test1 ubuntu /bin/bash

# 查看 vdata 目录是否存在
$ ls -dl /vdata/

# 创建一个文件，并写入内容
$ echo "I am test1" > /vdata/test1.txt

```

![图片描述](https://doc.shiyanlou.com/courses/uid977658-20190428-1556430247290)

同样的，我们执行上述操作，创建容器 test2。

```bash
# 创建容器 test2
$ docker container run \
    -it \
    --volumes-from ShiyanlouVolume \
    --name test2 ubuntu /bin/bash

# 查看 vdata 目录是否存在
$ ls -dl /vdata/

# 创建一个文件，并写入内容
$ echo "I am test2" > /vdata/test2.txt

```

我们进入到 ShiyanlouVolume 容器所在的终端，在挂载的数据中查看文件的内容：

```bash
# 查看数据卷中的内容
$ ls -al /vdata/

```

![图片描述](https://doc.shiyanlou.com/courses/uid977658-20190428-1556430402953)

从上图的结果中我们可以看到，数据卷在三个容器之间是共享的。

### 5.2 数据备份

数据存在于数据卷中，如果我们想要备份它，可以采用创建备份容器的方式。

```bash
$ docker container run \
   --volumes-from ShiyanlouVolume \
   -v /home/shiyanlou/backup:/backup \
   ubuntu tar cvf /backup/backup.tar /vdata/

```

`--volumes-from ShiyanlouVolume` 使得备份容器继承容器 ShiyanlouVolume 的数据卷。

`-v /home/shiyanlou/backup:/backup` 把 /home/shiyanlou/backup 目录采用绑定挂载的方式，挂载到容器的 /backup 目录上。

`tar cvf /backup/backup.tar /vdata` 容器中执行了这么一条压缩归档命令，将 /vdata 中的全部数据打包到了 /backup/backup.tar，而刚刚的数据绑定，使得整个压缩包存在于主机中，从而达到了数据备份的效果。

![图片描述](https://doc.shiyanlou.com/courses/uid977658-20190428-1556431013272)

如上图所示，数据卷中的所有数据都被打包到了 /home/shiyanlou/backup 目录中。

### 5.3 数据恢复

与数据备份相同的方式，我们可以使用如下命令创建恢复容器，来还原数据卷中的数据。

```bash
$ docker container run \
   --volumes-from ShiyanlouVolume \
   -v /home/shiyanlou/backup:/backup \
   ubuntu tar xvf /backup/backup.tar -C /

```

> 这里解压的路径为 `/` 即 `/vdata` 的上一级目录。

由于与数据备份非常相似，这里不再给出结果分析。

## 6. 总结

本节实验主要使用三种不同的方式将数据从 Docker 主机挂载到容器中，分别为卷（`volumes`），绑定挂载（`bind mounts`），临时文件系统（`tmpfs`）。还介绍了数据卷容器、数据卷的备份与恢复。

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。



# 网络管理

## 1. 课程说明

课程为纯动手实验教程，为了能说清楚实验中的一些操作会加入理论内容。理论内容我们不会写太多，已经有太多好文章了，会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

实验环境中可以联网，不受实验楼网络限制。

## 2. 学习方法

实验楼的 Docker 课程包含 14 个实验，每个实验都提供详细的步骤和截图，适用于有一定 Linux 系统基础，想快速上手 Docker 的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容。

如果实验开始部分有推荐阅读的材料，请务必先阅读后再继续实验，理论知识是实践必要的基础。

## 3. 本节内容简介

本节中，我们需要依次完成下面几项任务：

1. docker 容器端口映射
2. 自定义网络实现容器互联
3. host 和 none 网络的使用

对于 `Docker` 的镜像仓库来说，国内访问速度较慢，我们添加一个阿里云提供的 `Docker` 镜像加速器。

首先，我们需要编辑 `/etc/docker/daemon.json` 文件：

```bash
$ sudo vi /etc/docker/daemon.json

```

然后加入如下内容：

```bash
{
  "registry-mirrors": ["https://n6syp70m.mirror.aliyuncs.com"]
}

```

修改之后，需要重启 `docker` 服务，让修改生效。使用如下命令：

```bash
$ sudo service docker restart

```

## 4. 网络

> 在开始下面的内容之前，为了不出现命名上的冲突，也为了显示更为直观并且方便演示示例，首先需要将前面创建或启动的容器全部删除。可以使用下面两条命令达到这一效果：

```bash
# 暂停所有运行中的容器
$ docker container ls -q | xargs docker container stop

# 删除所有的容器
$ docker container ls -aq | xargs docker container rm

```

![图片描述](https://doc.shiyanlou.com/courses/uid977658-20190428-1556432590557)

在我们安装 `Docker` 后，会自动创建三个网络。我们可以使用下面的命令来查看这些网络：

```bash
$ docker network ls

```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516712155589.png)

如上图所示，三种默认的网络，分别为 `bridge`，`host`，`none`。

### 4.1 bridge

`bridge`，即桥接网络，在安装 `docker` 后会创建一个桥接网络，该桥接网络的名称为 `docker0`。我们可以通过下面两条命令去查看该值。

```bash
# 查看 bridge 网络的详细信息，并通过 grep 获取名称项
$ docker network inspect bridge | grep name

# 使用 ifconfig 查看 docker0 网络
$ ifconfig

```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516775691321.png)

在上图中，我们可以查看到对应的值。默认情况下，我们创建一个新的容器都会自动连接到 `bridge` 网络。使用 `docker network inspect bridge` 查看网桥网络的详细信息，结果如下所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516778041875.png)

可以看到 `docker0` 的默认网段是 `192.168.0.0/20`。

我们可以尝试创建一个容器，该容器会自动连接到 `bridge` 网络，例如我们创建一个名为 `shiyanlou001` 的容器：

```bash
$ docker container run \
    --name shiyanlou001 \
    -itd ubuntu /bin/bash

# 上述命令中默认使用 --network bridge ，即指定 bridge 网络
# 与下面的命令等同

$ docker container run \
    --name shiyanlou001 \
    --network bridge \
    -itd ubuntu /bin/bash

```

创建后，再次查看 `bridge` 的信息：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516778620559.png)

这时可以查看到相应的容器的网络信息，该容器在连接到 `bridge` 网络后，会从子网的地址池中获得一个 IP 地址，即上图中的 `192.168.0.2`。

使用 `docker container attach shiyanlou001` 命令，也可查看相应的地址信息：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516778879129.png)

> 如果提示没有找到 `ifconifg` 命令，可以通过如下命令安装：
>
> ```bash
> $ sudo apt update
> $ sudo apt install net-tools
> 
> ```

并且对于连接到默认的 `bridge` 之间的容器可以通过 IP 地址互相通信。例如我们启动一个 `shiyanlou002` 的容器，它可以与 `shiyanlou001` 通过 IP 地址进行通信。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516877067391.png)

> 如果提示没有找到 `ping` 命令，可使用如下命令安装：
>
> ```bash
> $ sudo apt update 
> $ sudo apt install iputils-ping
> 
> ```
>
> 其具体的实现原理可以参考链接 [Linux 上的基础网络设备](https://www.ibm.com/developerworks/cn/linux/1310_xiawc_networkdevice/index.html)，以及涉及到[网桥的工作原理](https://www.jianshu.com/p/9070f4bfeddf)

上述的操作我们通过 `ping` 命令演示了 `IP` 相关的内容。但是对于应用程序来讲，如果需要在外部进行访问，我们还会涉及到端口的使用，而 `Docker` 对于 `bridge` 网络使用端口的方式为设置端口映射，通过 `iptables` 实现。

下面我们通过 `iptables` 来为大家演示 docker 实现端口映射的方式，主要针对 `nat` 表和 `filter` 表：

(1) 首先删除掉上面创建的两个容器。这里不再给出具体的命令

(2) 这时，我们查看 `nat` 表的转发规则，使用如下命令：

```bash
$ sudo iptables -t nat -nvL

```

(3) 由于此时并未创建 docker 容器，nat 表中没有什么特殊的规则。接下来，我们使用实验 03 - Docker 镜像管理中构建的 `shiyanlou:1.0` 镜像创建一个容器 `shiyanlou001`，并将本机的端口 `10001` 映射到容器中的 `80` 端口上，在浏览器中可以通过 `localhost:10001` 访问容器 `shiyanlou001` 的 `apache` 服务，命令如下：

```bash
$ docker run -d -p 10001:80 --name shiyanlou001 shiyanlou:1.0

```

> 其中 `docker container run` 命令的 `-p` 参数是通过端口映射的方式，将容器的端口发布到主机的端口上。其使用格式为 `-p ip:hostPort:containerPort`。并且还可以指定范围，例如 `-p 10001-10100:1-100`，代表将容器 `1-100` 的端口映射到主机上的 `10001-10100`端口上，两者一一对应。

构建镜像 `shiyanlou:1.0` 的 DockerFile 如下，具体的构建过程请参考实验 03 的文档：

```dockerfile
# 指定基础镜像
FROM ubuntu:14.04

# 维护者信息
MAINTAINER shiyanlou/shiyanlou001@simplecloud.cn

# 镜像操作命令
RUN \
    apt-get -yqq update && \
    apt-get install -yqq apache2

# 容器启动命令
CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]

```

(4) 创建成功后，我们可以在浏览器中输入 `localhost:10001` 访问到容器 `shiyanlou001` 的 `apache` 服务，并查看此时 `iptables` 中 `nat` 表和 `filter` 表的规则，其中分别新增了一条比较重要的内容，如下图所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517189337316.png)

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517189420438.png)

(5) 接下来，再次使用镜像 `shiyanlou:1.0` 来启动一个容器 `shiyanlou002`，这次我们不指定端口映射，通过手动修改 `nat` 表的方式来模拟实现：

```bash
$ docker run -d --name shiyanlou002 shiyanlou:1.0

```

(6) 获取容器 `shiyanlou002` 的 ip 地址，如果按步骤操作此 ip 为 `192.168.0.3`。此时我们想通过主机的 `10002` 端口访问容器 `shiyanlou002` 的 `80` 端口，就可以添加一条规则：

```bash
# 添加一条规则，大致解释为将从非 docker0 接口上，目的端口为 10002 的 tcp 报文，修改其目的地址为 192.168.0.3:80

$ sudo iptables -t nat -A DOCKER ! -i docker0 -p tcp --dport 10002 -j DNAT --to-destination 192.168.0.3:80

```

(7) 添加成功后我们在主机发出的本地公网或内网 ip 加端口号 10002 的请求会被定位到 `192.168.0.3:80` 上，但是在将请求转发到 `docker0` 网桥上时，对于默认的 `filter` 表中的 `FORWARD` 链的规则是 `DROP`，因此我们还需要在 `filter` 表中设置相应的规则：

```bash
$ sudo iptables -t filter -A FORWARD ! -i docker0 -o docker0 -p tcp -d 192.168.0.3 -j ACCEPT --dport 80

# 或者你也可以选择将其加到由 docker 定义的 DOCKER 链中，上面的命令和下面的命令选择其中的一个即可

$ sudo iptables -t filter -A DOCKER ! -i docker0 -o docker0 -p tcp -d 192.168.0.3 -j ACCEPT --dport 80

```

(8) 此时我们就能够通过 `localhost:10002` 访问容器 `shiyanlou002` 中的 `apache` 服务了。 即通过 `iptables` 的方式实现了容器 `shiyanlou002` 上 `80` 端口到主机 `10002` 端口的映射。

(9) 最后，为了不影响后面实验的进行，这里我们删除掉手动添加的规则，并删除容器。

删除手动添加的规则可使用如下方法：

```bash
#查看 nat 规则
$ sudo iptables -t nat -nvL --line-numbers
#比如删除 DOCKER 链的第 2 条规则
$ sudo iptables -t nat -D DOCKER 2

#查看 filter 规则
$ sudo iptables -nvL --line-numbers
#比如删除 DOCKER 链第 1 条规则
$ sudo iptables -D DOCKER 1

```

因为每个环境都不同，具体删除第几条规则，应查看规则后再删除。

### 4.2 自定义网络

对于默认的 `bridge` 网络来说，使用端口可以通过端口映射的方式来实现，并且在上面的内容中我们也演示了容器之间通过 `IP` 地址互相进行通信。但是对于默认的 `bridge` 网络来说，每次重启容器，容器的 `IP` 地址都是会发生变化的，因为对于默认的 `bridge` 网络来说，并不能在启动容器的时候指定 ip 地址，在启动单个容器时并不容易看到这一区别。

#### 旧版的容器互联

容器间都是通过在 `/etc/hosts` 文件中添加相应的解析，通过容器名，别名，服务名等来识别需要通信的容器。

这里，我们启动两个容器，来演示旧的容器互联：

(1) 首先启动一个名为 `shiyanlou001` 的容器，使用镜像 `busybox`：

```bash
$ docker run -it --rm --name shiyanlou001 busybox /bin/sh

```

(2) 这时打开一个新的终端，启动一个名为 `shiyanlou002` 的容器，并使用 `--link` 参数与容器 `shiyanlou001` 互联。

```bash
$ docker run -it --rm --name shiyanlou002 --link shiyanlou001 busybox /bin/sh

```

> docker run 命令的 `--link` 参数的格式为 `--link :alias`。格式中的 `name` 为容器名，`alias` 为别名。即可以通过 `alias` 访问到该容器。

如下图所示，左侧为 `shiyanlou001`，右侧为 `shiyanlou002`：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517193383367.png)

(3) 如果此时 `shiyanlou001` 容器退出，这时我们启动一个 `shiyanlou003`，再次启动一个 `shiyanlou001`：

```bash
$ docker run -itd --name shiyanlou003 --rm busybox /bin/sh

$ docker run -it --name shiyanlou001 --rm busybox /bin/sh

```

按照顺序分配的原则，此时 `shiyanlou003` 的 IP 地址为 `192.168.0.2`，容器 `shiyanlou001` 的 IP 地址为 `192.168.0.4`。并且此时容器 `shiyanlou002` 中 `/etc/hosts` 文件的解析依旧不变，所以不能获取到正确的解析：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517194194401.png)

如上所示，旧的容器 `shiyanlou002` 通过 `--link` 连接到 `shiyanlou001`。而在 `shiyanlou001` 重启后，由于 IP 地址的变化，此时 `shiyanlou002` 并不能正确的访问到 `shiyanlou001`。

除了使用 `--link` 链接的方式来达到容器间互联的效果，在 `docker` 中，容器间的通信更应该使用的是自定义网络。

#### 自定义网络

docker 在安装时会默认创建一个桥接网络，除了使用默认网络之外，我们还可以创建自己的 `bridge` 或 `overlay` 网络。

如下所示，我们创建一个名为 `network1` 的桥接网络，简单命令如下：

```bash
$ docker network create network1

$ docker network ls

```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517195281317.png)

创建成功后，可以使用 `ifconfig` 或者 `ip addr show` 命令查看该桥接网络的网络接口信息，如下所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517195572337.png)

而对于该网络的详细信息可以通过 `docker network inspect network1` 命令来查看，如下图所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517195691681.png)

其相应的网络接口名称和子网都是由 docker 随机生成，当然，我们也可以手动指定：

```bash
# 首先删除掉刚刚创建的 network1 
$ docker network rm network1

# 再次创建 network1，指定子网
$ docker network create -d bridge --subnet=192.168.16.0/24 --gateway=192.168.16.1 network1

```

此时，我们可以运行一个容器 `shiyanlou001`，指定其网络为 `network1`，使用 `--network network1`：

```bash
$ docker run -it --name shiyanlou001 --network network1 --rm busybox /bin/sh

```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517196736819.png)

使用 `exit` 退出该容器使其自动删除，这时我们再次创建该容器，但是不指定其 `--network`：

```bash
$ docker run -it --name shiyanlou001 --rm busybox /bin/sh

```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517196983288.png)

此时，该容器连接到默认的 `bridge` 网络，这时，可以新打开一个终端，在其中运行如下命令，将 `shiyanlou001` 连接到 `network1` 网络中：

```bash
# 在新打开的终端中运行，将容器 shiyanlou001 连接到 network1 网络中
$ docker network connect network1 shiyanlou001

# 这时再次在容器 `shiyanlou001` 中使用 `ifconfig` 命令

```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517197247336.png)

如上图中所示，出现了一个 `eth1` 接口，此时，`eth0` 连接到默认的 `bridge` 网络，`eth1` 连接到 `network1` 网络。

对于自定义的网络来说，docker 嵌入的 `DNS` 服务支持连接到该网络的容器名的解析。这意味着连接到同一个网络的容器都可以通过容器名去 `ping` 另一个容器。

如下所示，启动两个容器，连接到 `network1`：

```bash
$ docker run -itd --name shiyanlou_1 --network network1 --rm busybox /bin/sh

$ docker run -it --name shiyanlou_2 --network network1 --rm busybox /bin/sh

```

启动之后，由于上述的两个容器都是连接到 `network1` 网络，所以可以通过容器名 `ping` 通：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517198037471.png)

除此之外，在用户自定义的网络中，是可以通过 `--ip` 指定 IP 地址的，而在默认的 `bridge` 网络不能指定 IP 地址：

```bash
# 连接到 network1 网络，运行成功
$ docker run -it --network network1 --ip 192.168.16.100 --rm busybox /bin/sh

# 连接到默认的 bridge 网络，下面的命令运行失败
$ docker run -it --rm busybox --ip 192.168.0.100 --rm busybox /bin/sh

```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517198404575.png)

### 4.3 host 和 none

`host` 网络，容器可以直接访问主机上的网络。

例如，我们启动一个容器，指定网络为 `host`：

```bash
$ docker run -it --network host --rm busybox /bin/sh

```

如下所示，该容器可以直接访问主机上的网络：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517214736450.png)

`none` 网络，容器中不提供其它网络接口。none 网络的容器创建之后还可以自己 connect 一个网络，比如使用 `docker network connet bridge 容器名` 可以将这个容器添加到 bridge 网络中。

```bash
$ docker run -it --nerwork none --rm busybox /bin/sh

```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517214875513.png)

## 5. 总结

本节实验中我们学习了以下内容：

1. docker 容器端口映射
2. 自定义网络实现容器互联
3. host 和 none 网络的使用

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。



# 编写 Dockerfile

## 1. 课程说明

课程为纯动手实验教程，为了能说清楚实验中的一些操作会加入理论内容。理论内容我们不会写太多，已经有太多好文章了，会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

实验环境中可以联网，不受实验楼网络限制。

## 2. 学习方法

实验楼的 Docker 课程包含 14 个实验，每个实验都提供详细的步骤和截图，适用于有一定 Linux 系统基础，想快速上手 Docker 的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容。

如果实验开始部分有推荐阅读的材料，请务必先阅读后再继续实验，理论知识是实践必要的基础。

## 3. 本节内容简介

在前面的实验中我们多次用到的 Dockerfile，在本实验里我们将通过完成一个实例来学习 Dockerfile 的编写。

本节中，我们需要依次完成下面几项任务：

1. Dockerfile 基本语法
2. Dockerfile 创建镜像流程

对于 `Docker` 的镜像仓库来说，国内访问速度较慢，我们添加一个阿里云提供的 `Docker` 镜像加速器。

首先，我们需要编辑 `/etc/docker/daemon.json` 文件：

```bash
$ sudo vi /etc/docker/daemon.json

```

然后加入如下内容：

```bash
{
  "registry-mirrors": ["https://n6syp70m.mirror.aliyuncs.com"]
}

```

修改之后，需要重启 `docker` 服务，让修改生效。使用如下命令：

```bash
$ sudo service docker restart

```

## 4. Dockerfile

Dockerfile 是一个文本文件，其中包含我们为了构建 Docker 镜像而手动执行的所有命令。Docker 可以从 Dockerfile 中读取指令来自动构建镜像。我们可以使用 `docker build` 命令来创建一个自动构建。

### 4.1 上下文

在 Docker 容器及镜像管理一节中我们有提到构建镜像的一些知识。

构建镜像时，该过程的第一件事是将 `Dockerfile` 文件所在目录下的所有内容递归的发送到守护进程。所以在大多数情况下，最好是创建一个新的目录，在其中保存 `Dockerfile`，并在其中添加构建 `Dockerfile` 所需的文件。而 Dockerfile 文件所在的路径也被称为上下文（context）。

首先创建一个目录，以便开始后面的实验过程：

```bash
$ mkdir dir1 && cd dir1

```

下面我们简单介绍 Dockerfile 中常用的指令。

### 4.2 FROM

使用 FROM 指令指定一个基础镜像，后续指令将在此镜像的基础上运行：

```
FROM ubuntu:14.04

```

### 4.3 USER

在 Dockerfile 中可以指定一个用户，后续的 `RUN`，`CMD` 以及 `ENTRYPOINT` 指令都会使用该用户去执行，但是该用户必须提前存在。

```
USER shiyanlou

```

### 4.4 WORKDIR

除了指定用户之外，还可以使用 `WORKDIR` 指定工作目录，对于 `RUN`，`CMD`，`COPY`，`ADD` 指令将会在指定的工作目录中去执行。也可以理解为命令执行时的当前目录。

```
WORKDIR /

```

### 4.5 RUN，CMD，ENTRYPOINT

RUN 指令用于执行命令，该指令有两种形式：

- `RUN `，使用 shell 去执行指定的命令 `command`，一般默认的 `shell` 为 `/bin/sh -c`。
- `RUN ["executable", "param1", "param2", ...]`，使用可执行的文件或程序 `executable`，给予相应的参数 `param`。

例如我们执行更新命令：

```
RUN apt-get update

```

CMD 的使用方式跟 RUN 类似，不过在一个 Dockerfile 文件中只能有一个 CMD 指令，如果有多个 CMD 指令，则只有最后一个会生效。该指令为我们运行容器时提供默认的命令，例如：

```
CMD echo "hello shiyanlou"

```

在构建镜像时使用了上面的 `CMD` 指令，则可以直接使用 `docker run image`，该命令等同于 `docker run image echo "hello shiyanlou"`。即作为默认执行容器时默认使用的命令，也可在 `docker run` 中指定需要运行的命令来覆盖默认的 `CMD` 指令。

除此之外，该指令还有一种特殊的用法，在 Dockerfile 中，如果使用了 ENTRYPOINT 指令，则 CMD 指令的值会作为 ENTRYPOINT 指令的参数：

```
CMD ["param1", "param2"]

```

ENTRYPOINT 指令会覆盖 CMD 指令作为容器运行时的默认指令，并且不会在 `docker run` 时被覆盖，如下示例：

```
FROM ubuntu:latest
ENTRYPOINT ["ls", "-a"]
CMD ["-l"]

```

上述构建的镜像，在我们使用 `docker run ` 时等同于 `docker run  ls -a -l` 命令。使用 `docker run  -i -s` 命令等同于 `docker run  ls -a -i -s` 指令。即 CMD 指令的值会被当作 ENTRYPOINT 指令的参数附加到 ENTRYPOINT 指令的后面，并且如果 `docker run` 中指定了参数，会覆盖 `CMD` 中给出的参数。

### 4.6 COPY 和 ADD

COPY 和 ADD 都用于将文件，目录等复制到镜像中。使用方式如下：

```
ADD <src>... <dest>
ADD ["<SRC>",... "<dest>"]

COPY <src>... <dest>
COPY ["<src>",... "<dest>"]

```

`` 可以指定多个，但是其路径不能超出上下文的路径，即必须在跟 Dockerfile 同级或子目录中。

`` 不需要预先存在，不存在路径时会自动创建，如果没有使用绝对路径，则 `` 为相对于工作目录的相对路径。

COPY 和 ADD 的不同之处在于，ADD 可以添加远程路径的文件，并且 `` 为可识别的压缩格式，如 gzip 或 tar 归档文件等，ADD 会自动将其解压缩为目录。

### 4.7 ENV

ENV 指令用于设置环境变量：

```
ENV <key> <value>
ENV <key>=<value> <key>=<value>...

```

### 4.8 VOLUME

VOLUME 指令将会创建指定的挂载目录，在容器运行时，将创建相应的匿名卷：

```
VOLUME /data1 /data2

```

上述指令将会在容器运行时，创建两个匿名卷，并挂载到容器中的 /data1 和 /data2 目录上。

### 4.9 EXPOSE

EXPOSE 指定在容器运行时监听指定的网络端口，它与 `docker run` 命令的 `-p` 参数不一样，并不实际映射端口，只是将该端口暴露出来，允许外部或其它的容器进行访问。

要将容器端口暴露出来，需要在 `dcoker run` 命令中使用 `-p` 或者 `--publish` 参数。如果采用 `-P` 随机映射端口的方式，Docker 会将在 DockerFile 中声明的所有 EXPOSE 的端口随机映射。

```
EXPOSE port

```

## 5. 从 Dockerfile 创建镜像

了解了上面一些常用于构建 Dockerfile 的指令之后，可以通过这些指令来构建一个镜像，如下所示，搭建一个 ssh 服务:

```
# 指定基础镜像
FROM ubuntu:14.04

# 安装软件
RUN apt-get update && apt-get install -y openssh-server && mkdir /var/run/sshd

# 添加用户 shiyanlou 及设定密码
RUN useradd -g root -G sudo shiyanlou && echo "shiyanlou:123456" | chpasswd shiyanlou

EXPOSE 22

CMD ["/usr/sbin/sshd", "-D"]

```

首先，我们在之前创建的一个空目录 `dir1` 中编辑 `Dockerfile` 文件，并将上面的内容复制到该文件中，相关的命令如下所示：

```
# 创建目录
$ mkdir dir1 && cd dir1

# 编辑 Dockerfile，将上面的内容写入
$ vim Dockerfile

# 最后执行构建命令
$ docker build -t sshd:test .

```

在上面的命令执行完成之后，该镜像就构建成功了，直接使用该镜像启动一个容器就可以运行一个 ssh 的服务，如下所示：

```
$ docker run -itd -p 10001:22 sshd:test

```

这时就可以通过公网的 IP 地址，以及端口 10001，并且使用用户 `shiyanlou`，密码 `123456`，远程通过 `ssh` 连接到该容器中了。

这里我们使用回环地址来进行测试，即自己请求自己的 ssh 连接。

首先安装 openssh 客户端，对应的命令为 `apt-get install openssh-client`。然后连接本机的 ssh-server，使用的命令为 `ssh -p 10001 shiyanlou@127.0.0.1`，这里的 `-p 10001` 即使用端口 10001，也就是我们刚刚映射的端口。

实验的结果如下图，可以看到，我们成功连接到了容器。

![图片描述](https://doc.shiyanlou.com/courses/uid977658-20190428-1556437424946)

## 6. 总结

本节实验中我们学习了以下内容：

1. Dockerfile 基本语法
2. Dockerfile 创建镜像流程

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。



# 使用 Docker 运行 MongoDB 和 Redis

## 1. 课程说明

课程为纯动手实验教程，为了能说清楚实验中的一些操作会加入理论内容。理论内容我们不会写太多，已经有太多好文章了，会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

实验环境中可以联网，不受实验楼网络限制。

## 2. 学习方法

实验楼的 Docker 课程包含 14 个实验，每个实验都提供详细的步骤和截图，适用于有一定 Linux 系统基础，想快速上手 Docker 的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容。

如果实验开始部分有推荐阅读的材料，请务必先阅读后再继续实验，理论知识是实践必要的基础。

## 3. 本节内容简介

在本实验里我们将通过完成 MongoDB 和 Redis 两个容器来学习 Dockerfile 及 Docker 的运行机制。

本节中，我们需要依次完成下面几项任务：

1. MongoDB 的安装及配置
2. Redis 的安装及配置
3. Dockerfile 的编写
4. 从 Dockerfile 构建镜像

本次实验的需求是完成 Dockerfile，通过 Dockerfile 创建 MongoDB 或 Redis 应用。Dockerhub 上已经提供了官方的 MongoDB 和 Redis 镜像，本实验仅仅用于学习 Dockerfile 及 Docker 机制。

> MongoDB 是一个基于分布式文件存储的数据库。由 C++ 语言编写。旨在为 WEB 应用提供可扩展的高性能数据存储解决方案。MongoDB 是一个介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的。特点是高性能、易部署、易使用，存储数据非常方便。 -来自百度百科

> Redis 是一个开源的使用 ANSI C 语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value 数据库，并提供多种语言的 API。 -来自百度百科

对于 `Docker` 的镜像仓库来说，国内访问速度较慢，我们添加一个阿里云提供的 `Docker` 镜像加速器。

首先，我们需要编辑 `/etc/docker/daemon.json` 文件：

```bash
$ sudo vi /etc/docker/daemon.json

```

然后加入如下内容：

```bash
{
  "registry-mirrors": ["https://n6syp70m.mirror.aliyuncs.com"]
}

```

修改之后，需要重启 `docker` 服务，让修改生效。使用如下命令：

```bash
$ sudo service docker restart

```

## 4. 实验准备

### 4.1 实验分析

在本实验中，我们除了安装所需的核心服务外，还安装一个 ssh 服务提供便捷的管理。

为了提高 `docker build` 速度，我们直接使用阿里云的 Ubuntu 源。因此要在 Dockerfile 开始位置增加下面一句命令：

```
RUN echo "deb http://mirrors.cloud.aliyuncs.com/ubuntu/ trusty main universe" > /etc/apt/sources.list

```

### 4.2 创建 Dockerfile 文件

首先，需要创建一个目录来存放 Dockerfile 文件，目录名称可以任意，在目录里创建 Dockerfile 文件：

```bash
$ cd /home/shiyanlou
$ mkdir shiyanloumongodb shiyanlouredis
$ touch shiyanloumongodb/Dockerfile shiyanlouredis/Dockerfile

```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid1709timestamp1457511228797.png)

使用 vim/gedit 编辑 Dockerfile 文件，根据我们的需求输入内容。

## 5. Dockerfile 基本框架

### 5.1 基本框架

按照上一节学习的内容，我们先完成 Dockerfile 基本框架。

依次输入下面的基本框架内容：

```dockerfile
# Version 0.1

# 基础镜像
FROM ubuntu:14.04

# 维护者信息
MAINTAINER shiyanlou@shiyanlou.com

# 镜像操作命令
RUN echo "deb http://mirrors.cloud.aliyuncs.com/ubuntu/ trusty main universe" > /etc/apt/sources.list
RUN apt-get update && apt-get install -yqq supervisor && apt-get clean

# 容器启动命令
CMD ["supervisord"]

```

上面的 Dockerfile 创建了一个简单的镜像，并使用 `Supervisord` 启动服务。

### 5.2 安装 SSH 服务

首先安装所需要的软件包：

```dockerfile
RUN apt-get install -yqq openssh-server openssh-client

```

创建运行目录：

```dockerfile
RUN mkdir -p /var/run/sshd

```

设置 root 密码及允许 root 通过 ssh 登录：

```dockerfile
RUN echo 'root:shiyanlou' | chpasswd
RUN sed -i 's/PermitRootLogin without-password/PermitRootLogin yes/' /etc/ssh/sshd_config

```

## 6. 完成 MongoDB Dockerfile

在上述基本的架构下，我们根据需求可以增加新的内容到 Dockerfile 中，完成 MongoDB Dockerfile。

进入到 shiyanloumongodb 的目录编辑 Dockerfile：

```bash
$ cd /home/shiyanlou/shiyanloumongodb/
$ vim Dockerfile

```

### 6.1 安装最新的 MongoDB

在 Ubuntu 最新版本下安装 MongoDB 非常简单，参考 [MongoDB 安装文档](https://docs.mongodb.org/manual/tutorial/install-mongodb-on-ubuntu/) 。有两种方法：

方法一是添加 mongodb 的源，执行 `apt-get install mongodb-org` 就可以安装下面的所有软件包：

1. mongodb-org-server：mongod 服务和配置文件
2. mongodb-org-mongos：mongos 服务
3. mongodb-org-shell：mongo shell 工具
4. mongodb-org-tools：mongodump，mongoexport 等工具

方法二是下载二进制包，然后解压出来就可以。

由于 MongoDB 的官网连接网速问题，我们使用第二种方案，并把最新的 MongoDB 的包放到阿里云上。

MongoDB 的下载链接如下：

```
http://labfile.oss-cn-hangzhou-internal.aliyuncs.com/courses/498/mongodb-linux-x86_64-ubuntu1404-3.2.3.tgz

```

> 下载链接较长，建议保存到工具栏中的剪切板，在云主机中复制即可。

我们完善 Dockerfile，使用 ADD 命令添加压缩包到镜像：

```dockerfile
RUN mkdir -p /opt
ADD http://labfile.oss-cn-hangzhou-internal.aliyuncs.com/courses/498/mongodb-linux-x86_64-ubuntu1404-3.2.3.tgz /opt/mongodb.tar.gz
RUN cd /opt && tar zxvf mongodb.tar.gz && rm -rf mongodb.tar.gz
RUN mv /opt/mongodb-linux-x86_64-ubuntu1404-3.2.3 /opt/mongodb

```

创建 MongoDB 的数据存储目录：

```dockerfile
RUN mkdir -p /data/db

```

将 MongoDB 的执行路径添加到环境变量里：

```dockerfile
ENV PATH=/opt/mongodb/bin:$PATH

```

MongoDB 和 SSH 对外的端口：

```dockerfile
EXPOSE 27017 22

```

### 6.2 编写`Supervisord`配置文件

添加 `Supervisord` 配置文件来启动 mongodb 和 ssh，创建文件`/home/shiyanlou/shiyanloumongodb/supervisord.conf`，添加以下内容：

```
[supervisord]
nodaemon=true

[program:mongodb]
command=/opt/mongodb/bin/mongod

[program:ssh]
command=/usr/sbin/sshd -D

```

Dockerfile 中增加向镜像内拷贝该文件的命令：

```dockerfile
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

```

### 6.3 完整的 Dockerfile

```dockerfile
# Version 0.1

# 基础镜像
FROM ubuntu:14.04

# 维护者信息
MAINTAINER shiyanlou@shiyanlou.com

# 镜像操作命令
RUN echo "deb http://mirrors.cloud.aliyuncs.com/ubuntu/ trusty main universe" > /etc/apt/sources.list
RUN apt-get -yqq update && apt-get install -yqq supervisor
RUN apt-get install -yqq openssh-server openssh-client

RUN mkdir /var/run/sshd
RUN echo 'root:shiyanlou' | chpasswd
RUN sed -i 's/PermitRootLogin without-password/PermitRootLogin yes/' /etc/ssh/sshd_config

RUN mkdir -p /opt
ADD http://labfile.oss-cn-hangzhou-internal.aliyuncs.com/courses/498/mongodb-linux-x86_64-ubuntu1404-3.2.3.tgz /opt/mongodb.tar.gz
RUN cd /opt && tar zxvf mongodb.tar.gz && rm -rf mongodb.tar.gz
RUN mv /opt/mongodb-linux-x86_64-ubuntu1404-3.2.3 /opt/mongodb

RUN mkdir -p /data/db

ENV PATH=/opt/mongodb/bin:$PATH

COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

EXPOSE 27017 22

# 容器启动命令
CMD ["supervisord"]

```

## 7. 完成 Redis Dockerfile

在上述基本的架构下，我们根据需求可以增加新的内容到 Dockerfile 中，完成 Redis Dockerfile。

进入到 shiyanlouredis 的目录编辑 Dockerfile：

```bash
$ cd /home/shiyanlou/shiyanlouredis/
$ vim Dockerfile

```

### 7.1 安装 Redis

由于 MongoDB 中我们已经学习了如何通过二进制压缩包安装最新版本 MongoDB 的过程，在此安装 Redis 我们直接使用 Ubuntu 源中默认的 Redis 版本。

安装方法非常简单：

```dockerfile
RUN apt-get install redis-server

```

添加对外的端口号：

```dockerfile
EXPOSE 6379 22

```

### 7.2 编写`Supervisord`配置文件

添加`Supervisord`配置文件来启动 redis-server 和 ssh，创建文件`/home/shiyanlou/shiyanlouredis/supervisord.conf`，添加以下内容：

```
[supervisord]
nodaemon=true

[program:redis]
command=/usr/bin/redis-server

[program:ssh]
command=/usr/sbin/sshd -D

```

Dockerfile 中增加向镜像内拷贝该文件的命令：

```dockerfile
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

```

### 7.3 完整的 Dockerfile

```dockerfile
# Version 0.1

# 基础镜像
FROM ubuntu:14.04

# 维护者信息
MAINTAINER shiyanlou@shiyanlou.com

# 镜像操作命令
RUN echo "deb http://mirrors.cloud.aliyuncs.com/ubuntu/ trusty main universe" > /etc/apt/sources.list
RUN apt-get -yqq update && apt-get install -yqq supervisor redis-server
RUN apt-get install -yqq openssh-server openssh-client

RUN mkdir /var/run/sshd
RUN echo 'root:shiyanlou' | chpasswd
RUN sed -i 's/PermitRootLogin without-password/PermitRootLogin yes/' /etc/ssh/sshd_config

COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

EXPOSE 6379 22

# 容器启动命令
CMD ["supervisord"]

```

## 8. 从 Dockerfile 创建镜像

### 8.1 创建 MongoDB 镜像

进入到`/home/shiyanlou/shiyanloumongodb/`目录，执行创建命令。

`docker build` 执行创建，`-t`参数指定镜像名称：

```bash
$ docker build -t shiyanloumongodb:0.1 /home/shiyanlou/shiyanloumongodb/

```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid1709timestamp1457511284792.png)

`docker images` 查看创建的新镜像已经出现在了镜像列表中：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid1709timestamp1457511294074.png)

由该镜像创建新的容器 mongodb：

```bash
$ docker run -P -d --name mongodb shiyanloumongodb:0.1

```

> 这里的 `-P` 参数将容器 EXPOSE 的端口随机映射到主机的端口。查看映射到那个端口的方式是输入 `docker ps` 或者 `docker container ls`，在最后一项 `STATUS` 中有映射的端口信息。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid1709timestamp1457511328107.png)

上述 `docker ps` 命令的输出可以看到 MongoDB 的端口号已经被自动映射到了本地的 32768 端口，后续步骤我们对 MongoDB 是否启动进行测试。

打开 Xfce 终端中输入下面的命令连接 mongodb 容器中的服务：

```bash
$ mongo --host 127.0.0.1 --port 32768

```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid1709timestamp1457511456998.png)

> 如果提示 `command not found mongo` ，可使用 `sudo apt-get install -y mongodb` 安装。

### 8.2 创建 Redis 镜像

进入到`/home/shiyanlou/shiyanlouredis/`目录，执行创建命令。

`docker build` 执行创建，`-t`参数指定镜像名称：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid1709timestamp1457511491952.png)

`docker images` 查看创建的新镜像已经出现在了镜像列表中：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid1709timestamp1457511499237.png)

由该镜像创建新的容器 redis：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid1709timestamp1457511508152.png)

上述`docker ps`命令的输出可以看到 redis 的端口号已经被自动映射到了本地的 32769 端口，SSH 服务的端口号也映射到了 32770 端口。

打开 Xfce 终端中输入下面的命令连接 redis 容器中的 ssh 和 redis 服务：

```bash
$ ssh root@127.0.0.1 -p 32770
$ redis-cli -h 127.0.0.1 -p 32769

```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid1709timestamp1457511585507.png)

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid1709timestamp1457511592966.png)

> 如果提示 `command not found redis-cli` ，就使用 `sudo apt-get install -y redis-server` 安装。

## 9. 总结

本节实验中我们学习了以下内容：

1. MongoDB 的安装
2. Redis 的安装
3. Dockerfile 的编写
4. 从 Dockerfile 构建镜像

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。



# 使用 Docker 运行 Wordpress

## 1. 课程说明

课程为纯动手实验教程，为了能说清楚实验中的一些操作会加入理论内容。理论内容我们不会写太多，已经有太多好文章了，会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

实验环境中可以联网，不受实验楼网络限制。

## 2. 学习方法

实验楼的 Docker 课程包含 14 个实验，每个实验都提供详细的步骤和截图，适用于有一定 Linux 系统基础，想快速上手 Docker 的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容。

如果实验开始部分有推荐阅读的材料，请务必先阅读后再继续实验，理论知识是实践必要的基础。

## 3. 本节内容简介

在本实验里我们将通过完成一个 Wordpress 的容器来学习 Dockerfile 及 Docker 的运行机制。

本节中，我们需要依次完成下面几项任务：

1. Wordpress 的安装及配置
2. Dockerfile 的编写
3. 从 Dockerfile 构建镜像

本次实验的需求是完成一个 Dockerfile，通过该 Dockerfile 创建一个 Wordpress 应用。尽管 Dockerhub 上已经提供了官方的 Wordpress 镜像，本实验仅仅用于学习 Dockerfile 及 Docker 机制。

> 扩展;`WordPress` 是一种使用`PHP` 语言开发的博客平台，用户可以在支持 `PHP` 和 `MySQL` 数据库的服务器上架设属于自己的网站。也可以把 `WordPress` 当作一个内容管理系统（CMS）来使用。
>
> - [百度百科](https://baike.baidu.com/item/WordPress/450615?fr=aladdin)

对于 `Docker` 的镜像仓库来说，国内访问速度较慢，我们添加一个阿里云提供的 `Docker` 镜像加速器。

首先，我们需要编辑 `/etc/docker/daemon.json` 文件：

```bash
$ sudo vi /etc/docker/daemon.json

```

然后加入如下内容：

```bash
{
  "registry-mirrors": ["https://n6syp70m.mirror.aliyuncs.com"]
}

```

修改之后，需要重启 `docker` 服务，让修改生效。使用如下命令：

```bash
$ sudo service docker restart

```

## 4. 实验准备

### 4.1 实验分析

在本实验中，除了部署 `Wordpress` 之外，我们需要安装 `Nginx` 提供对外的外部服务，同时安装一个 `ssh` 服务提供便捷的管理。

Wordpress 依赖的包比较多，至少需要安装 `php`，`mysql` 等依赖软件。所以为了提高 `docker build` 速度，我们直接使用阿里云的 Ubuntu 源。因此要在 Dockerfile 开始位置增加下面一句命令：

```dockerfile
RUN echo "deb http://mirrors.cloud.aliyuncs.com/ubuntu/ trusty main universe" > /etc/apt/sources.list

```

> 注意：如果在自己的电脑操作，请将 `mirrors.cloud.aliyuncs.com` 改为 `mirrors.aliyun.com`。

### 4.2 创建 Dockerfile 文件

首先，需要创建一个目录来存放 Dockerfile 文件，目录名称可以任意，在目录里创建 Dockerfile 文件：

```bash
$ cd /home/shiyanlou
$ mkdir shiyanlouwordpress
$ cd shiyanlouwordpress
$ touch Dockerfile

```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid1710timestamp1457515800852.png)

使用 vim/gedit 编辑 Dockerfile 文件，根据我们的需求输入内容。

## 5. Dockerfile 基本框架

按照上一节学习的内容，我们先完成 Dockerfile 基本框架。

依次输入下面的基本框架内容：

```dockerfile
# Version 0.1

# 基础镜像
FROM ubuntu:14.04

# 维护者信息
MAINTAINER shiyanlou@shiyanlou.com

# 镜像操作命令
RUN echo "deb http://mirrors.cloud.aliyuncs.com/ubuntu/ trusty main universe" > /etc/apt/sources.list
RUN apt-get -yqq update && apt-get install -yqq supervisor && apt-get clean

# 容器启动命令
CMD ["supervisord"]

```

上面的 Dockerfile 创建了一个简单的镜像，使用`Supervisord`启动服务。

## 6. 完善 Dockerfile

在上述基本的架构下，我们根据需求可以增加新的内容到 Dockerfile 中。

### 6.1 安装依赖包

安装的依赖包分为两类，一类是系统需要的服务，比如 `nginx` 等，一类是 Wordpress 依赖的 `php` 组件。

增加下面的内容到 Dockerfile：

```dockerfile
RUN apt-get -yqq install nginx supervisor wget php5-fpm php5-mysql

```

### 6.2 安装 Wordpress

首先创建安装目录：

```dockerfile
RUN mkdir -p /var/www

```

然后下载 Wordpress 4.4.2 版本压缩包并解压：

```dockerfile
ADD http://labfile.oss-cn-hangzhou-internal.aliyuncs.com/courses/498/wordpress-4.4.2.tar.gz /var/www/wordpress-4.4.2.tar.gz
RUN cd /var/www && tar zxvf wordpress-4.4.2.tar.gz && rm -rf wordpress-4.4.2.tar.gz
RUN chown -R www-data:www-data /var/www/wordpress

```

### 6.2 安装 SSH 服务

首先安装所需要的软件包：

```dockerfile
RUN apt-get install -y openssh-server openssh-client

```

创建运行目录：

```dockerfile
RUN mkdir /var/run/sshd

```

设置 root 密码及允许 root 通过 ssh 登录：

```dockerfile
RUN echo 'root:shiyanlou' | chpasswd
RUN sed -i 's/PermitRootLogin without-password/PermitRootLogin yes/' /etc/ssh/sshd_config

```

### 6.3 安装 Mysql

首先通过 `debconf-set-selections` 设置安装过程中需要的 root 密码为 shiyanlou，然后再执行 `apt-get` 安装 mysql：

```dockerfile
RUN echo "mysql-server mysql-server/root_password password shiyanlou" | debconf-set-selections
RUN echo "mysql-server mysql-server/root_password_again password shiyanlou" | debconf-set-selections
RUN apt-get  install -y mysql-server mysql-client

```

安装后需要创建安装 Wordpress 所需的`wordpress`数据库：

```dockerfile
RUN service mysql start && mysql -uroot -pshiyanlou -e "create database wordpress;"

```

### 6.4 开放端口

开放 80（Web 服务）和 22（SSH 服务）端口：

```dockerfile
EXPOSE 80 22

```

### 6.5 配置 Supervisord

添加`Supervisord`配置文件来启动 php5-fpm，nginx，mysql 和 ssh，创建文件`/home/shiyanlou/shiyanlouwordpress/supervisord.conf`，添加以下内容：

```
[supervisord]
nodaemon=true

[program:php5-fpm]
command=/usr/sbin/php5-fpm -c /etc/php5/fpm
autorstart=true

[program:mysqld]
command=/usr/bin/mysqld_safe

[program:nginx]
command=/usr/sbin/nginx
autorstart=true

[program:ssh]
command=/usr/sbin/sshd -D

```

Dockerfile 中增加向镜像内拷贝该文件的命令：

```dockerfile
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

```

### 6.6 添加启动命令

启动`Supervisord`：

```dockerfile
CMD ["/usr/bin/supervisord"]

```

## 7. 配置文件

### 7.1 Nginx 配置文件

为了能让 Nginx 顺利支持 `/var/www/wordpress` 目录下的 Wordpress，我们需要添加文件到 `/etc/nginx/sites-available/default` 。

在 Dockerfile 所在的 `/home/shiyanlou/shiyanlouwordpress` 目录下创建文件 nginx-config，并输入以下内容：

```nginx
server {
    listen *:80;
    server_name localhost;

    root /var/www/wordpress;
    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        try_files $uri =404;
        include fastcgi_params;
        fastcgi_pass unix:/var/run/php5-fpm.sock;
    }
}

```

这是一个基本的 Nginx 配置，包含的核心配置信息：

1. 指定 Wordpress 的目录：`/var/www/wordpress`
2. 设置对 PHP 页面的支持，包括设置默认 index 页面为 `index.php` 等

### 7.2 Wordpress 配置文件

Wordpress 配置文件为 `/var/www/wordpress/wp-config.php` ，有两种方法修改这个文件：

1. 使用 sed 更改文件中需要配置的项目
2. 预先配置好该文件，在 `docker build` 过程中拷贝到镜像中替换原文件

我们这里介绍第一种方法：

```dockerfile
RUN sed -i 's/database_name_here/wordpress/g' /var/www/wordpress/wp-config-sample.php
RUN sed -i 's/username_here/root/g' /var/www/wordpress/wp-config-sample.php
RUN sed -i 's/password_here/shiyanlou/g' /var/www/wordpress/wp-config-sample.php
RUN mv /var/www/wordpress/wp-config-sample.php /var/www/wordpress/wp-config.php

```

可以看到，其中配置了数据库连接的用户名和密码，还记得前面步骤中安装 mysql 时设置的内容吗？

修改完成后的 `wp-config.php` 文件节选：

```php
define('DB_NAME', 'wordpress');
/** MySQL database username */
define('DB_USER', 'root');
/** MySQL database password */
define('DB_PASSWORD', 'shiyanlou');

```

### 7.3 Dockerfile 更新

在 Dockerfile 中 CMD 前面的添加 COPY 命令，用来更新镜像中的配置文件：

```dockerfile
COPY nginx-config /etc/nginx/sites-available/default

RUN sed -i 's/database_name_here/wordpress/g' /var/www/wordpress/wp-config-sample.php
RUN sed -i 's/username_here/root/g' /var/www/wordpress/wp-config-sample.php
RUN sed -i 's/password_here/shiyanlou/g' /var/www/wordpress/wp-config-sample.php
RUN mv /var/www/wordpress/wp-config-sample.php /var/www/wordpress/wp-config.php

```

## 8. 从 Dockerfile 创建镜像

将上述内容完成后放入到 `/home/shiyanlou/shiyanlouwordpress/Dockerfile` 文件中，最终得到的 Dockerfile 文件如下：

```dockerfile
# Version 0.1
FROM ubuntu:14.04

MAINTAINER shiyanlou@shiyanlou.com

RUN echo "deb http://mirrors.cloud.aliyuncs.com/ubuntu/ trusty main universe" > /etc/apt/sources.list
RUN apt-get -yqq update
RUN apt-get -yqq install nginx supervisor wget php5-fpm php5-mysql
RUN echo "daemon off;" >> /etc/nginx/nginx.conf

RUN mkdir -p /var/www
ADD http://labfile.oss-cn-hangzhou-internal.aliyuncs.com/courses/498/wordpress-4.4.2.tar.gz /var/www/wordpress-4.4.2.tar.gz
RUN cd /var/www && tar zxvf wordpress-4.4.2.tar.gz && rm -rf wordpress-4.4.2.tar.gz
RUN chown -R www-data:www-data /var/www/wordpress

RUN mkdir /var/run/sshd
RUN apt-get install -yqq openssh-server openssh-client
RUN echo 'root:shiyanlou' | chpasswd
RUN sed -i 's/PermitRootLogin without-password/PermitRootLogin yes/' /etc/ssh/sshd_config

RUN echo "mysql-server mysql-server/root_password password shiyanlou" | debconf-set-selections
RUN echo "mysql-server mysql-server/root_password_again password shiyanlou" | debconf-set-selections
RUN apt-get  install -yqq mysql-server mysql-client

EXPOSE 80 22

COPY nginx-config /etc/nginx/sites-available/default
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
RUN service mysql start && mysql -uroot -pshiyanlou -e "create database wordpress;"
RUN sed -i 's/database_name_here/wordpress/g' /var/www/wordpress/wp-config-sample.php
RUN sed -i 's/username_here/root/g' /var/www/wordpress/wp-config-sample.php
RUN sed -i 's/password_here/shiyanlou/g' /var/www/wordpress/wp-config-sample.php
RUN mv /var/www/wordpress/wp-config-sample.php /var/www/wordpress/wp-config.php

CMD ["/usr/bin/supervisord"]

```

完成后查看目录文件：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid1710timestamp1457516140339.png)

`docker build` 执行创建，`-t` 参数指定镜像名称：

```bash
$ docker build -t shiyanlouwordpress:0.2 /home/shiyanlou/shiyanlouwordpress/

```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid1710timestamp1457516322070.png)

`docker images` 查看创建的新镜像已经出现在了镜像列表中：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid1710timestamp1457516347855.png)

由该镜像创建新的容器 wordpress，并映射本地的 80 端口到容器的 80 端口：

```bash
$ docker run -d -p 80:80 --name wordpress shiyanlouwordpress:0.2

```

> 注意：一般出现问题都是配置的问题，注意检查配置是否有拼写错误。如果提示是 80 端口被占用，可能是我们本地的 nginx 占用了端口，使用 `sudo service nginx stop` 关闭即可。
>
> 使用 `docker container ls` 可以看到创建出来的容器。

最后打开桌面上的 firefox 浏览器，输入本地地址访问 `127.0.0.1/wp-admin/install.php` ，看到我们的 Wordpress 网站安装配置界面，由于默认会连接 google 的文件，所以打开会比较慢：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid1710timestamp1457516547080.png)

## 9. 总结

本节实验中我们学习了以下内容：

1. Wordpress 的安装及配置
2. Dockerfile 的编写
3. 从 Dockerfile 构建镜像

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。





# 搭建自己的 Docker Registry

## 1. 课程说明

课程为纯动手实验教程，为了能说清楚实验中的一些操作会加入理论内容。理论内容我们不会写太多，已经有太多好文章了，会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

实验环境中可以联网，不受实验楼网络限制。

## 2. 学习方法

实验楼的 Docker 课程包含 14 个实验，每个实验都提供详细的步骤和截图，适用于有一定 Linux 系统基础，想快速上手 Docker 的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容。

如果实验开始部分有推荐阅读的材料，请务必先阅读后再继续实验，理论知识是实践必要的基础。

## 3. 本节内容简介

在本实验里我们将自己动手搭建一个类似 Docker Hub 的 Docker Registry。

本节中，我们需要依次完成下面几项任务：

1. Docker Registry 的部署和配置
2. 使用 Registry 管理仓库和镜像
3. Docker Registry 的配置

本次实验的需求是搭建一个 Docker Registry，通过该 Registry 管理仓库和镜像。

Docker Registry 是一个用来管理 Docker 镜像的服务，本身也是一个 Docker 容器。大部分情况下都可以使用 Docker Hub，私有的 Docker Registry 使用场景主要在当需要对容器镜像存储进行完全控制或需要把镜像管理进行集成的情况。

对于 `Docker` 的镜像仓库来说，国内访问速度较慢，我们添加一个阿里云提供的 `Docker` 镜像加速器。

首先，我们需要编辑 `/etc/docker/daemon.json` 文件：

```bash
$ sudo vi /etc/docker/daemon.json

```

然后加入如下内容：

```bash
{
  "registry-mirrors": ["https://n6syp70m.mirror.aliyuncs.com"]
}

```

修改之后，需要重启 `docker` 服务，让修改生效。使用如下命令：

```bash
$ sudo service docker restart

```

## 4. Docker Registry 部署

由于 Docker Registry 已经被制作成一个 Docker 镜像，所以安装部署非常简单，只需要按照我们通常的`docker run`就可以，如果本地没有 `registry` 的镜像，则会自动从 Docker Hub 上获取。

需要注意的是 Registry 默认的对外服务端口是 5000，如果我们宿主机上运行的 Registry 需要对外提供服务，可以通过映射端口的方式提供。

本节实验中我们使用 `registry:2` 镜像，这个镜像为 2.0 版本的 Registry。

部署 Docker Registry 的命令：

```bash
$ docker run -d -p 5000:5000 --restart=always --name registry registry:2

```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521102681964.png-wm)

停止和删除 Registry 只需要用容器管理实验中学到的 `docker container stop` 和 `docker container rm` 命令。

## 5. 使用 Registry 管理仓库和镜像

以下示例从 Docker Hub 中拉取 redis 镜像，并将其重新标记为 my-redis ，然后将其推送到本地 Registry 。

### 5.1 推送镜像

运行后我们进行简单的测试，可以先将本地的一个 redis 镜像推送到 Registry 上：

```bash
# 拉取镜像 redis:latest
$ docker pull redis

```

使用 `docker tag` 对 `redis` 镜像进行操作，会发现 `docker image ls` 列表中多了一个`localhost:5000/my-redis:latest` 的镜像。

```bash
$ docker tag redis:latest localhost:5000/my-redis

```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521103308999.png-wm)

将 `localhost:5000/my-redis` 镜像推送到 `localhost:5000` 的 Docker Registry 上。

注意 Docker 镜像的命名规则 `localhost:5000/my-redis` 中，`localhost:5000` 表示 Registry 的地址和端口。

```bash
$ docker push localhost:5000/my-redis

```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521103496106.png-wm)

### 5.2 获取镜像

如果我们需要从 `localhost:5000` 的 Registry 上下载镜像，只需要简单的 `docker pull` 命令就可以完成。

首先删除本地的 `localhost:5000/redis:latest` 和 `redis:latest` 镜像，以便从 Registry 下载镜像：

```bash
$ docker image remove redis:latest 
$ docker image remove localhost:5000/my-redis

```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521103811855.png-wm)

然后从 Registry 中下载我们刚才上传的镜像。

```bash
$ docker pull localost:5000/my-redis

```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521104057288.png-wm)

`docker pull` 命令执行的过程中发现所有的数据层都已经存在本地。再次检查 `docker image ls` 就会看到新的`localhost:5000/redis:latest` 镜像。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521104217265.png-wm)

## 6. Docker Registry 的配置

### 6.1 镜像存储

如果我们希望将 Registry 里的镜像都存储在一个数据卷中，这样做的好处是我们可以在宿主机上对该数据卷进行备份和监控。请回忆我们的数据卷参数 `-v`，此外，Registry 中存储镜像的目录是 `/var/lib/registry`。

为了避免端口及命名冲突，将先前实验创建的 Registry 删除，然后在本机创建一个存储路径 `/home/shiyanlou/data` ：

```bash
$ docker container rm -f registry
$ mkdir /home/shiyanlou/data

```

创建一个 Registry，使用宿主机上的 `/home/shiyanlou/data` 目录作为存储镜像的数据卷：

```bash
$ docker run -d \
  -p 5000:5000 \
  --restart=always \
  --name registry \
  -v /home/shiyanlou/data:/var/lib/registry \
  registry:2

```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521104741517.png-wm)

将本地的 `localhost:5000/my-redis` 镜像推送到 Registry，然后查看 `/home/shiyanlou/data` 目录的变化：

```bash
$ docker push localhost:5000/my-redis
$ ls /home/shiyanlou/data

```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521105335725.png-wm)

会看到`/home/shiyanlou/data` 目录多了一个 `docker` 文件夹。在 `/home/shiyanlou/data/docker/registry/v2/repositories` 中可以看到刚刚上传的镜像。

除了使用数据卷做镜像存储之外，Registry 还支持将镜像存储到 亚马逊的 S3，OpenStack 的 Swift/Glance 等存储后端。

详细的配置方式可以见文档 [Registry 存储配置选项](https://docs.docker.com/registry/configuration/#storage) 。

### 6.2 认证机制

Registry 支持多种认证方式，这里仅仅介绍基本的认证方式，如果我们为 Registry 配置一个域名对外提供服务，需要首先配置 `TLS` ，可参考[运行可从外部访问的 Registry](https://docs.docker.com/registry/deploying/#customize-the-storage-back-end) 。在本实验中，我们暂时不适用域名的方式。

(1) 首先我们需要创建一个保存用户名和密码的认证文件夹，并使用 `htpasswd` 创建一个认证信息：

```bash
$ mkdir /home/shiyanlou/auth
$ docker run --entrypoint htpasswd registry:2 -Bbn shiyanlouuser shiyanloupass > auth/htpasswd

```

认证信息中，用户名为 shiyanlouuser，密码是 shiyanloupass。

(2) 删除 Registry

因为我们一会要创建一个 registry 容器，所以先删除我们之前创建的 registry 容器。

```bash
$ docker container rm -f registry

```

(3) 启动 Registry，使用该认证信息，需要将 `/home/shiyanlou/auth` 挂载到 Registry 的 `/auth` 目录：

```bash
$ docker run -d \
  -p 5000:5000 \
  --restart=always \
  --name registry \
  -v $(pwd)/auth:/auth \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
  registry:2

```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521108681547.png-wm)

接下来使用如下命令推送 `my-redis` 到本地 Registry，你会发现推送失败。为了可以通过认证向 Registry 推送本地的镜像，我们需要首先使用 `docker login` 配置认证信息，登录时会提示输入用户名和密码，我们输入之前配置的 `shiyanlouuser` 和 `shiyanloupass` 即可。

```bash
# 登录方式
$ docker login localhost:5000

```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521108975264.png-wm)

> 注意：Linux 上的密码输入默认不显示，不是实验楼的系统卡住了。

其他认证方式的配置参数，可以参考文档 [Registry 认证配置](https://docs.docker.com/registry/configuration/#auth) 。

### 6.3 详细配置

**注意：本部分没有实验操作，仅仅是简单介绍 Registry 更详细的配置方法。**

Registry 的配置信息都存储在 Registry 中的`/etc/docker/registry/config.yml`文件。最简单的配置方法是直接将修改好的 YAML 配置文件挂载到 Registry 容器中。

如果只修改少数几个配置信息，可以通过设置环境变量来改变，比如修改 `storage/filesystem/rootdirectory`，只需要使用 `docker run` 的 `-e` 参数设置相应的`REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY`变量。

本节简单介绍下最常用的配置信息，详细信息可见 [Docker Registry 配置](https://docs.docker.com/registry/configuration/)。

#### 日志级别

设置 `REGISTRY_LOG_LEVEL`，`REGISTRY_LOG_FORMATTER` 来设置日志输出级别和格式，级别可以设置为 `error`，`warn`，`info`，`debug`，默认为 `info`级别。输出格式可以为 `json`，`text`，`logstash`。

#### Hook

Hook 设置可以配置根据日志信息发送邮件。配置范例如下表所示，该例子来自 Docker 官方文档：

```yaml
hooks:
  - type: mail
    levels:
      - panic
    options:
      smtp:
        addr: smtp.sendhost.com:25
        username: sendername
        password: password
        insecure: true
      from: name@sendhost.com
      to:
        - name@receivehost.com

```

#### 存储设置

前面的实验中，我们已经简单接触了本地数据卷存储的设置方式。此外，还可以设置多种存储后端。

每种配置的参数列表（来自 Docker 官方文档）:

```yaml
storage:
  filesystem:
    rootdirectory: /var/lib/registry
  azure:
    accountname: accountname
    accountkey: base64encodedaccountkey
    container: containername
  gcs:
    bucket: bucketname
    keyfile: /path/to/keyfile
    rootdirectory: /gcs/object/name/prefix
  s3:
    accesskey: awsaccesskey
    secretkey: awssecretkey
    region: us-west-1
    bucket: bucketname
    encrypt: true
    secure: true
    v4auth: true
    chunksize: 5242880
    rootdirectory: /s3/object/name/prefix
  rados:
    poolname: radospool
    username: radosuser
    chunksize: 4194304
  swift:
    username: username
    password: password
    authurl: https://storage.myprovider.com/auth/v1.0 or https://storage.myprovider.com/v2.0 or https://storage.myprovider.com/v3/auth
    tenant: tenantname
    tenantid: tenantid
    domain: domain name for Openstack Identity v3 API
    domainid: domain id for Openstack Identity v3 API
    insecureskipverify: true
    region: fr
    container: containername
    rootdirectory: /swift/object/name/prefix
  oss:
    accesskeyid: accesskeyid
    accesskeysecret: accesskeysecret
    region: OSS region name
    endpoint: optional endpoints
    internal: optional internal endpoint
    bucket: OSS bucket
    encrypt: optional data encryption setting
    secure: optional ssl setting
    chunksize: optional size valye
    rootdirectory: optional root directory
  inmemory:
  delete:
    enabled: false
  cache:
    blobdescriptor: inmemory
  maintenance:
    uploadpurging:
      enabled: true
      age: 168h
      interval: 24h
      dryrun: false
  redirect:
    disable: false

```

#### 认证设置

前面的实验已经学习过基本的 htpasswd 认证方式，除此之外，还支持 Token 方式的认证，Token 方式可以使用独立的认证服务器。配置实例：

```yaml
auth:
  token:
    realm: token-realm
    service: token-service
    issuer: registry-token-issuer
    rootcertbundle: /root/certs/bundle

```

#### HTTP 设置

此处包含一些 Registry 提供的 HTTP 服务与协议的基本配置，包括监听地址与端口，TLS 等配置路径：

```yaml
http:
  addr: localhost:5000
  net: tcp
  prefix: /my/nested/registry/
  host: https://myregistryaddress.org:5000
  secret: asecretforlocaldevelopment
  tls:
    certificate: /path/to/x509/public
    key: /path/to/x509/private
    clientcas:
      - /path/to/ca.pem
      - /path/to/another/ca.pem
  debug:
    addr: localhost:5001
  headers:
    X-Content-Type-Options: [nosniff]

```

## 7. 总结

本节实验中我们学习了以下内容：

1. Docker Registry 的部署和配置
2. 使用 Registry 管理仓库和镜像
3. Docker Registry 的配置

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。



# Docker 安全

## 1. 课程说明

课程为纯动手实验教程，为了能说清楚实验中的一些操作会加入理论内容。理论内容我们不会写太多，已经有太多好文章了，会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

实验环境中可以联网，不受实验楼网络限制。

## 2. 学习方法

实验楼的 Docker 课程包含 14 个实验，每个实验都提供详细的步骤和截图，适用于有一定 Linux 系统基础，想快速上手 Docker 的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容。

如果实验开始部分有推荐阅读的材料，请务必先阅读后再继续实验，理论知识是实践必要的基础。

## 3. 本节内容简介

在本实验里我们将通过实验体会几个Docker中的安全设置。

本节中，我们需要依次完成下面几项任务：

1. 使用证书加固 Docker Daemon安 全
2. 设置特权级运行的容器：`--privileged=true`
3. 设置容器权限白名单：`--cap-add`
4. Docker Bench Security

Docker 的安全已经随着容器技术的推广受到越来越多的关注，本节实验中我们将体会几个最常用的Docker安全相关的配置。在开始实验之前，推荐阅读关于Docker 安全性的文章：

1. [Flux7 Docker系列教程（五）：Docker 安全](https://segmentfault.com/a/1190000002711383)
2. [Docker安全性探讨与实践：实践篇](http://www.infoq.com/cn/news/2014/09/docker-safe)

对于 `Docker` 的镜像仓库来说，国内访问速度较慢，我们添加一个阿里云提供的 `Docker` 镜像加速器。

对于 `Docker` 的镜像仓库来说，国内访问速度较慢，我们添加一个阿里云提供的 `Docker` 镜像加速器。

首先，我们需要编辑 `/etc/docker/daemon.json` 文件：

```bash
$ sudo vi /etc/docker/daemon.json

```

然后加入如下内容：

```bash
{
  "registry-mirrors": ["https://n6syp70m.mirror.aliyuncs.com"]
}

```

修改之后，需要重启 `docker` 服务，让修改生效。使用如下命令：

```bash
$ sudo service docker restart

```

## 4. 使用证书加固Docker Daemon安全

Docker Daemon 启动的服务对外提供的是 HTTP 接口，为了增强 HTTP 连接的安全性，我们通过设置 TLS 来认证客户端是可信的，只有通过证书验证的客户端才可以连接 Docker Daemon。

通常情况下服务器和客户端证书都需要通过第三方 CA 签发，在本实验中为了操作方便，我们使用的是自签名的证书。

### 设置环境变量

有时候在创建 CA 密钥的时候可能会产生一个 `unable to write 'random state'` 的报错，我们需要先设置一个 `RANDFILE` 的环境变量：

```bash
$ export RANDFILE=.rnd

```

### 4.1 创建 CA 证书

首先创建一组 CA 私钥和公钥，先创建 CA 私钥，注意需要输入密码，这个密码是不会显示的，请务必记住，下面每次使用 CA 签名时都需要输入：

```bash
$ sudo openssl genrsa -aes256 -out ca-key.pem 4096

Generating RSA private key, 4096 bit long modulus
................................................++
...................................++
e is 65537 (0x10001)
Enter pass phrase for ca-key.pem:        #此处输入你想设置的密码
Verifying - Enter pass phrase for ca-key.pem:  #再次输入

```

然后使用 `ca-key.pem` 创建用来签名的公钥 `ca.pem` ，输入必要的信息：

```bash
$ sudo openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem 

Enter pass phrase for ca-key.pem:  #输入之前设置的密码
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:CN  # 输入国家代码
State or Province Name (full name) [Some-State]:Beijing
Locality Name (eg, city) []:Beijing
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Shiyanlou # 输入组织名
Organizational Unit Name (eg, section) []:CourseTeam
Common Name (e.g. server FQDN or YOUR name) []:localhost # 输入服务器域名
Email Address []:shiyanlou@shiyanlou.com

```

### 4.2 服务端证书配置

创建服务器的 key 和证书 `server.csr`，然后使用 CA 证书签发服务器证书：

```bash
$ sudo openssl genrsa -out server-key.pem 4096

Generating RSA private key, 4096 bit long modulus
............................................................++
................++
e is 65537 (0x10001)
$ sudo openssl req -subj "/CN=localhost" -sha256 -new -key server-key.pem -out server.csr

$ sudo echo subjectAltName = IP:127.0.0.1 >> extfile.cnf
#设置仅用于服务器身份验证
$ sudo echo extendedKeyUsage = serverAuth >> extfile.cnf

$ sudo openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out server-cert.pem -extfile extfile.cnf 

Signature ok
subject=/CN=localhost
Getting CA Private Key
Enter pass phrase for ca-key.pem:   #输入之前设置的密码

```

上面的步骤中 `extfile.cnf` 文件的作用是添加允许连接的 IP 地址，注意服务器证书创建时需要输入服务器的域名，在本实验中我们是用的是 `localhost`。

### 4.3 客户端证书配置

客户端的证书用来连接Docker Daemon服务，同服务器端证书的操作类似，先创建 key 和 csr 证书，然后使用 CA签发：

```bash
$ sudo openssl genrsa -out client-key.pem 4096

Generating RSA private key, 4096 bit long modulus
.......................................++
...................++
e is 65537 (0x10001)

$ sudo openssl req -subj '/CN=client' -new -key client-key.pem -out client.csr 

$ sudo echo extendedKeyUsage = clientAuth > client-extfile.cnf

$ sudo openssl x509 -req -days 365 -sha256 -in client.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out cert.pem -extfile client-extfile.cnf

Signature ok
subject=/CN=client
Getting CA Private Key
Enter pass phrase for ca-key.pem:   #输入之前设置的密码

```

现在查看 `/home/shiyanlou` 目录下所有创建的证书和key文件：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid1712timestamp1458201290334.png)

已经生成了 cert.pem 和 server-cert.pem ，我们就可以删除两个证书签名请求了：

```bash
$ rm -v client.csr server.csr

```

为防止意外损坏密钥，可以删除其写入权限：

```bash
$ sudo chmod -v 0400 ca-key.pem server-key.pem client-key.pem
$ sudo chmod -v 0444 ca.pem server-cert.pem cert.pem

```

### 4.4 配置服务端

在配置之前我们需要先停止 docker 服务：

```bash
$ sudo service docker stop

```

在服务器端运行如下命令让 Docker 守护进程只接受提供 CA 信任的证书的客户端连接：

```bash
$ sudo dockerd --tlsverify --tlscacert=ca.pem --tlscert=server-cert.pem --tlskey=server-key.pem \
  -H=0.0.0.0:2376

```

**注意：** 保持运行状态，不要关掉此命令窗口或者使用 ctrl + c 中断。

### 4.5 客户端连接 Docker

直接使用不增加证书参数的方式连接并执行 `docker image ls` ，系统会返回不能连接。而使用客户端连接则可以正确执行。新开一个终端命令窗口执行如下命令：

```bash
$ sudo docker --tlsverify --tlscacert=ca.pem --tlscert=cert.pem --tlskey=client-key.pem -H=127.0.0.1:2376 image ls

```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521195451476.png-wm)

为了后续实验的方便，我们创建一个alias，来避免输入 docker 命令的 TLS 参数：

```bash
$ alias docker='sudo docker --tlsverify --tlscacert=ca.pem --tlscert=cert.pem --tlskey=client-key.pem -H=127.0.0.1:2376'

$ docker image ls

```

## 5. 设置特权级运行的容器：`--privileged=true`

有的时候我们需要容器具备更多的权限，比如操作内核模块，控制swap交换分区，挂载USB磁盘，修改MAC地址等。本实验中我们给予容器这些权限，仅仅通过一个简单的`--privileged=true` 的参数。

为了对比，我们先后创建两台容器shiyanlou和prishiyanlou，后者具备`--privileged=true`参数和特权。

首先创建一个不具备特权参数的容器shiyanlou，并查看容器的IP地址和MAC地址信息：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid1712timestamp1458201529861.png)

尝试修改 MAC 地址，会收到 `Operation not permitted` 报错信息：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid1712timestamp1458201559515.png)

尝试创建并挂载一个 iso 文件，同样，也会收到错误信息：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid1712timestamp1458201611619.png)

从 shiyanlou 容器中退出，我们创建一个具备 `--privileged=true` 参数的 prishiyanlou 容器，看是否有不同：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid1712timestamp1458201662092.png)

在这个容器中，我们首先尝试修改 MAC 地址，修改成功后使用 ifconfig 命令查看验证：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid1712timestamp1458201689816.png)

再次创建一个 iso 文件，并 mkfs.ext3 格式化分区后，mount 命令挂载到 `/mnt`，验证发现可以成功挂载：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid1712timestamp1458201747483.png)

`Ctrl-P Ctrl-Q` 退出容器后查看两个容器中的配置信息：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid1712timestamp1458201783453.png)

## 6. 设置容器权限白名单：`--cap-add`

`--privileged=true` 的权限非常大，接近于宿主机的权限，为了防止用户的滥用，需要增加限制，只提供给容器必须的权限。此时 Docker 提供了权限白名单的机制，使用 `--cap-add` 添加必要的权限。

为了能够修改 MAC 地址，我们给予新的容器 capshiyanlou 一个 `NET_ADMIN` 的权限。创建该容器，注意命令中的参数：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid1712timestamp1458201833312.png)

尝试修改MAC地址，验证 `--cap-add` 是否起到作用：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid1712timestamp1458201867193.png)

退出容器后，可以在 `docker container inspect` 命令中查看容器的必要配置：

```bash
$ docker container inspect -f {{.HostConfig.Privileged}} capshiyanlou
false

$ docker container inspect -f {{.HostConfig.CapAdd}} capshiyanlou
{[NET_ADMIN]}

```

## 7. Docker Bench Security

Docker Benchmark Security 是一个用于 docker 安全检查的应用程序，它会去检查下面的这些项目，并提供警告信息。它是一个开源工具，参见 [GitHub-Docker Bench Security](https://github.com/docker/docker-bench-security/) 。

- 主机配置
- Docker 守护进程配置
- Docker 守护进程配置文件
- 容器镜像和构建文件
- 容器运行时间
- Docker 安全操作

接下来我们就来安装 Docker Bench Security：

```bash
$ docker run -it --net host --pid host --userns host --cap-add audit_control \
    -e DOCKER_CONTENT_TRUST=$DOCKER_CONTENT_TRUST \
    -v /var/lib:/var/lib \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v /usr/lib/systemd:/usr/lib/systemd \
    -v /etc:/etc --label docker_bench_security \
    docker/docker-bench-security

```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971521798043023-wm)

会显示出很多的提示信息：

![实验楼](https://dn-simplecloud.shiyanlou.com/87971521798200035-wm)

> `PASS` ：这些项目都是很稳固的，不需要关注，pass 越多越好。
>
> `WARN` ：需要修复的项目。
>
> `INFO` ：如果这些项目和你的设置和安全需要相关，建议检查和修复这些项目。
>
> `NOTE` ：一些建议。

## 8. 总结

本节实验中我们学习了以下内容：

1. 使用证书加固 Docker Daemon安 全
2. 设置特权级运行的容器：`--privileged=true`
3. 设置容器权限白名单：`--cap-add`
4. Docker Bench Security

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。



# Docker Compose 项目

## 1. 课程说明

课程为纯动手实验教程，为了能说清楚实验中的一些操作会加入理论内容。理论内容我们不会写太多，已经有太多好文章了，会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

实验环境中可以联网，不受实验楼网络限制。

## 2. 学习方法

实验楼的 Docker 课程包含 15 个实验，每个实验都提供详细的步骤和截图，适用于有一定 Linux 系统基础，想快速上手 Docker 的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容。

如果实验开始部分有推荐阅读的材料，请务必先阅读后再继续实验，理论知识是实践必要的基础。

## 3. 本节内容简介

在本实验里我们将通过实验学习 `Docker Compose` 项目。`Docker Compose` 是一个用来创建和运行多容器应用的工具。使用 `Compose` 首先需要编写 `Compose` 文件来描述多个容器服务以及之间的关联，然后通过命令根据配置启动所有的容器。

Dockerfile 可以定义一个容器，而一个 Compose 的模板文件（YAML 格式）可以定义一个包含多个相互关联容器的应用。Compose 项目使用 python 编写，基于后面的实验中我们将学习的 Docker API 实现。

Docker Compose 项目地址：https://github.com/docker/compose 感兴趣的同学可以关注。

本节中，我们需要依次完成下面几项任务：

1. 安装 Docker Compose
2. 创建 Docker Compose 服务 - Web+redis 网站

对于 `Docker` 的镜像仓库来说，国内访问速度较慢，我们添加一个阿里云提供的 `Docker` 镜像加速器。

首先，我们需要添加编辑 `/etc/docker/daemon.json` 文件，加入如下内容：

```bash
{
  "registry-mirrors": ["https://n6syp70m.mirror.aliyuncs.com"]
}

```

修改之后，需要重启 `docker` 服务，让修改生效。使用如下命令：

```bash
$ sudo service docker restart

```

## 4. Docker Compose

### 4.1 概述

#### Compose

`Compose` 是定义和运行多容器 `Docker` 应用程序的工具。使用 `Compose`，可以通过编辑 `YAML` 文件来配置应用程序的服务。它可以用来管理应用程序的生命周期，例如启动，停止以及重构服务。

#### service

在分布式应用程序中，应用程序的不同部分被称为服务（service），例如常见的提供数据库存储的服务。服务实际上只是生产中的镜像。一个服务仅仅运行一个镜像，但它定义了服务运行的方式，例如使用哪个端口，该容器应该运行多少个副本等。

#### 使用过程

使用 Compose 的三个过程如下：

1. 定义应用程序的环境，即 Dockerfile
2. 定义组成应用程序的服务，一般为定义 `docker-compose.yml` 文件
3. 启动整个应用程序

> 关于 docker-compose.yml 文件的详细编写格式可以参考：https://docs.docker.com/compose/compose-file/#reference-and-guidelines

目前有三种版本的 Compose 文件格式：

1. version 1: 最早的版本使用传统格式，将在未来的 Compose 版本中被弃用
2. version 2: 现在使用最多的文件格式
3. version 3: 最新的版本，旨在 Compose 和被集成到 Docker Engine 中的 swarm mode 之间互相兼容（swarm 在下一节的内容会学习相关的知识）。

### 4.2 安装

在 Linux 中，Compose 需要单独安装，我们需要从 github 上下载 Docker Compose 二进制文件。但是官网提供的从 github 上下载的链接速度十分缓慢，在实验环境中，我们已提供该文件，直接使用以下命令进行下载：

```bash
$ wget http://labfile.oss-cn-hangzhou.aliyuncs.com/courses/980/software/docker-compose-Linux-x86_64

```

下载成功后，为了能够直接使用该可执行文件执行命令，一般将其放入 `$PATH` 的环境变量支持的路径中，并添加可执行权限，使用的命令如下：

```bash
$ sudo mv docker-compose-Linux-x86_64 /usr/local/bin/docker-compose

$ sudo chmod +x /usr/local/bin/docker-compose

```

执行完成后，就能够在终端下直接使用 `docker-compose` 命令了：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517365151690.png)

### 4.3 实例

在这里，我们将创建一个 web 应用程序，该实例参考 Docker Compose 官方文档，有两个服务，并做了一些改变。

在本实验中，我们需要两个容器：

(1) web 容器：提供 web 服务，并连接后端的 redis 服务

(2) redis 容器：提供 redis 服务，接收 web 容器的连接

其文件目录结构如下所示：

```markdown
app
|----web
|     |----web.py
|     |----requirements.txt
|     |----Dockerfile
|
|----docker-compose.yml

```

> 注意，这里 app 目录是在 /home/shiyanlou 目录下

(3) 首先编辑 `app/web/web.py` 文件，写入下面的内容：

```py
from flask import Flask
from redis import Redis

app = Flask(__name__)
redis = Redis(host='redis', port=6379)

@app.route('/')
def hello():
    redis.incr('number')
    return 'Hello Shiyanlou! # %s' % redis.get('number')

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=80, debug=True)

```

上述代码创建一个十分简单的 web 应用程序。该程序会连接 `redis` 服务，在访问 `/` 页面时，会自动将变量 `number` 加一。

(4) 编辑 `app/web/requirements.txt` 文件，输入如下内容

```
flask==0.10
redis==2.10.3

```

创建 `requirements.txt` 文件，输入需要使用的 python 依赖包，方便安装。

(5) 编辑 `app/web/Dockerfile` 文件，添加如下内容：

```dockerfile
FROM python:2.7
COPY ./ /web/
WORKDIR /web
RUN pip install -r requirements.txt
CMD python web.py

```

上述 `Dockerfile` 定义了一个镜像，该镜像基于 `python:2.7` 镜像制作，在其基础上安装相应的 python 包，并执行 `CMD` 命令来启动该应用程序。

(6) 编辑 `app/docker-compose.yml` 文件：

```
services:
  redis:
    image: redis:3.2
  web:
    build:
      context: /home/shiyanlou/app/web
    depends_on:
    - redis
    ports:
    - 8001:80/tcp
    volumes:
    - /home/shiyanlou/app/web:/web:rw
version: '3.0'

```

该 `docker-compose.yml` 文件定义了两个服务，分别为 `web` 和 `redis` 服务。并且我们配置 `web` 服务的端口映射，以及挂载相应的目录。 `depends_on` 定义了依赖关系，被依赖服务的容器需要先创建。

(7) 进入 `app` 目录下，执行 `docker-compose up` 命令来启动服务：

```bash
$ cd app
$ docker-compose up

```

由于此时 `web` 服务的镜像还未构建，所以此时会自动根据 `build`指示，使用 `/home/shiyanlou/app/web/Dockerfile` 文件构建镜像。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517557297094.png)

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517557398547.png)

运行成功后，此时我们可以打开浏览器，输入 `127.0.0.1:8001`，获取到正确的结果：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517368221961.png)

(8) 除此之外，也可以使用 `-d` 参数，即 `docker-compose up -d` 在其在后台运行：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517557688428.png)

(9) 如果需要暂停以及删除容器，可以直接运行 `docker-compose down` 命令即可

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517558444487.png)

## 5. 总结

本节实验中我们学习了以下内容：

1. 安装 Docker Compose
2. 创建 Docker Compose 服务 - Web+redis 网站

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。



# Docker Swarm 项目

## 1. 课程说明

课程为纯动手实验教程，为了能说清楚实验中的一些操作会加入理论内容。理论内容我们不会写太多，已经有太多好文章了，会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

实验环境中可以联网，不受实验楼网络限制。

## 2. 学习方法

实验楼的 Docker 课程包含 14 个实验，每个实验都提供详细的步骤和截图，适用于有一定 Linux 系统基础，想快速上手 Docker 的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容。

如果实验开始部分有推荐阅读的材料，请务必先阅读后再继续实验，理论知识是实践必要的基础。

## 3. 本节内容简介

在本实验里我们将通过实验学习 Docker Swarm 项目。Docker Swarm 是一个 Docker 集群部署和管理工具，可以把一组 Docker 服务器虚拟成一台容器服务器，并提供标准的 Docker API。所有可以直接连接 Docker 服务器的工具都可以连接 Docker Swarm。

由于实验楼只提供了一台实验机，所以无法搭建集群，有部分实验操作只能以单台服务器作为演示，请见谅。

本节中，我们需要依次完成下面几项任务：

1. 下载 Docker Swarm
2. 部署和管理 Swarm 集群

在开始实验之前，推荐先阅读下面的文章，了解 Swarm 的架构特点和概念：

1. [孙宏亮 - 深入浅出 Docker Swarm](http://www.csdn.net/article/2015-01-26/2823714)

对于 `Docker` 的镜像仓库来说，国内访问速度较慢，我们添加一个阿里云提供的 `Docker` 镜像加速器。

首先，我们需要添加编辑 `/etc/docker/daemon.json` 文件，加入如下内容：

```bash
{
  "registry-mirrors": ["https://n6syp70m.mirror.aliyuncs.com"]
}

```

修改之后，需要重启 `docker` 服务，让修改生效。使用如下命令：

```bash
$ sudo service docker restart

```

## 4. 概述

### 4.1 Swarm

Swarm 是 Docker 发布的管理集群的工具，一个集群由多个运行 `Docker` 的主机组成。下面我们简单介绍 Swarm 中的一些关键概念。

Swarm 在 Docker 1.12 之后被集成到 Docker Engine 中，又被称为 `swarm mode`，后面我们所说的 swarm 都是指 swarm mode。

### 4.2 关键概念

#### Role

一个集群由多个运行 Docker 的主机组成，分别作为管理者（Manager）和工作者（Worker）两个角色。管理者管理集群中的成员，而工作者运行集群服务。给定的 Docker 主机可以是一个管理员，也可以是一个工作者，或者同时具备这两个角色。

#### Node

一个节点（Node）是参与到 Swarm 集群中的一个实例。一般表现为运行 Docker 的主机。

#### 服务与任务

一个服务是任务在管理节点或工作节点执行的定义，服务中运行的单个容器被称为任务。

使用集群模式运行服务时，一般有两种选项：

1. replicated services，复制服务，根据设定的值，swarm 调度在节点之间运行指定的副本任务。
2. global services，全局服务，集群在每个可用节点上运行一项任务。

一个服务的多个任务之间没有什么不同，但是对于一些特殊的服务而言，例如涉及到端口映射的服务，即便设定了多个任务，也只能启动一个。

#### 堆栈

堆栈（stack）是一组相互关联的服务，即一个堆栈能够定义和协调整个应用程序的功能（但是一些非常复杂的应用程序可能需要使用多个堆栈）。

对于在上一节我们学习的 Docker Compose 定义的应用程序来说，从技术角度来讲，就可以说我们一直在使用堆栈。但是 Docker Compose 是运行在单个主机上，而这里我们所说的堆栈可以运行在一个集群中，即是一个分布式的应用程序。

## 5. Docker Swarm

### 5.1 环境搭建

注意：由于实验室环境限制，实验楼的在线实验环境不再配备具有公网 IP 的网卡。

下面的实验过程可以替换为以下方式：本地两台安装有 Docker 且在同一局域网的主机。

#### 创建一个 swarm

在 Docker 1.12 版本之后，Swarm 被集成到 Docker Engine 中（即 Swarm mode）。我们可以直接运行以下命令来创建一个集群：

```bash
$ docker swarm init --advertise-addr <IP>

```

> 如果我们使用本地局域网内的两台机器来搭建集群，那么这里要将 <IP> 替换为局域网网卡。

对于单网卡，单 IP 来说，我们可以直接使用 `docker swarm init` 命令，实验环境中只有一张网卡，用于访问外部网络的网卡为 `eth0`，这里可以指定 `` 为 `eth0` 上的地址，因为其它节点必须能够访问管理者的 IP 地址。其运行结果中包含将新节点加入到集群中的命令：

![图片描述](https://doc.shiyanlou.com/courses/uid600404-20191125-1574652465507)

这时，我们可以使用 `docker info` 命令查看 swarm 的状态：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517281019192.png)

除此之外，还可以使用 `docker node ls` 命令查看有关节点的信息：

```bash
$ docker node ls

```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517281062552.png)

#### 向 swarm 中添加节点

在运行 `docker swarm init` 时会输出向 swarm 中添加节点的命令。除此之外，我们也可以使用如下两个命令分别获取向 swarm 中添加管理节点和工作节点的命令，获取到的命令需要在被添加的节点上运行：

```bash
# 管理节点
$ docker swarm join-token manager

# 工作节点
$ docker swarm join-token worker

```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517281108402.png)

由于实验环境中，我们只有一台服务器，如果自己本地搭建的有 docker 环境，可以尝试在自己的机器上运行上图中的命令，如下所示，为作者在本机 windows 安装的 docker ，将其加入到 swarm 中：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517281271834.png)

这时，再次使用 `docker node ls` 命令，就可以看到新增加的 `worker` 节点了，如下图所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517562812869.png)

#### 移除节点

如果需要在集群中移除一个节点，首先该节点的状态必须为 `down`，即该节点不可用，之后就可以正常的删除该节点了，在管理节点使用如下命令：

```bash
# NODE 为该节点的 ID
$ docker node rm NODE

```

#### 提权或撤销权限

对于一个 `worker` 节点来说，我们可以将其提升为一个 `manager` 节点，也可以将一个管理节点变为 `worker` 节点。在管理节点使用下面的命令即可：

```bash
# 提权
docker node promote NODE

# 撤权
docker node demote NODE

```

### 5.2 Docker Compose 与 Docker Swarm

对于 Docker Compose 来说，即使我们可以定义多容器的应用程序，但是多个容器依然只能在单个主机上工作。

这里我们以上一节定义的应用程序为例。进入 `app` 目录下，执行 `docker-compose up -d` 命令来启动服务。

`docker-compose up` 命令的运行结果如下图所示，图中标注的提示信息提示我们 Compose 并没有使用集群模式去部署服务，并将所有的容器调度到当前节点：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517561367768.png)

此时运行成功后打开浏览器，输入 `127.0.0.1:8001`，依旧可以获取到正确的结果：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517368221961.png)

但是如果需要使用集群模式部署服务，需要使用 `docker stack deploy` 命令，并且服务所需的镜像需要提前构建好。所以需要修改 `app/docker-compose.yml` 文件，修改后如下所示：

```yaml
services:
  redis:
    image: redis:3.2
  web:
    image: app_web:latest
    depends_on:
    - redis
    ports:
    - 8001:80/tcp
    volumes:
    - /home/shiyanlou/app/web:/web:rw
version: '3.0'

```

> swarm 只支持 version: '3.0' 版本的 docker-compose.yml 文件

这时首先使用 `docker-compose down` 命令暂停之前使用 `docker-compose` 启动的容器，然后使用 `docker stack deploy` 命令部署该应用程序，使用的命令如下：

```bash
$ docker stack deploy -c docker-compose.yml app

```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517369570872.png)

上述命令会创建两个服务，不过与直接使用 `docker-compose` 不同的是，该命令部署的是一个堆栈（stack），即一组相互关联的服务，服务会被部署在集群中，而不是单个节点。并且此时我们创建的是服务，而不是单个容器，即便我们暂停或删除运行副本任务的容器，该容器也会再次启动。

> 如果你添加了自己运行 docker 的主机到 swarm 中，就有可能发现上述部署的服务的部分容器运行在 swarm 中的不同节点上，即分布式应用程序。

如果有添加自己运行 docker 的主机到集群中，就会发现上述两个服务，其中某一个服务运行在自己的节点上，如下所示，`app_redis` 服务运行在作者的 windows 节点上：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517561818075.png)

而实验环境中运行的则是 `app_web` 服务，如下图所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517562145047.png)

### 5.3 管理堆栈和服务

在创建一个集群之后，`docker stack` 和 `docker service` 两个命令集就变得可用了，他们分别用于管理堆栈和管理服务。

在上面部署了一个堆栈到集群中，我们可以使用下面的命令来查看所有的堆栈：

```bash
$ docker stack ls

```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517563035483.png)

如图所示，该 `app` 堆栈有两个服务，具体的服务可以使用下面的命令来查看：

```bash
$ docker stack services app

```

查看所有的服务则可以使用 `docker service ls` 命令，由于总共只有两个服务，所以下面两个命令的运行结果是一样的：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517563385194.png)

除此之外，对于我们之前引入的任务的概念，我们也可以查看相应的任务，来确定该任务的状态以及具体运行于某个节点，如下所示：

```bash
# 查看 app 堆栈的任务
$ docker stack ps app

# 查看服务 app_redis 的任务
$ docker service ps app_redis

```

由于会显示启动失败的任务，并且集群中是否有添加自己的节点，所以显示结果可能并不一致，这里没有给出具体的命令截图。

并且由于此时实在集群中部署的服务，所以手动暂停或删除容器之后，还会自动启动新的容器，如下所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517564302740.png)

因此对于服务的管理，我们需要直接操作该堆栈或者操作服务，而不是容器。

例如，我们停止服务可以使用如下的命令：

```bash
# 移除一个堆栈，将会移除堆栈中的所有服务
$ docker stack rm app

# 移除一个或多个服务
$ docker service rm app_redis app_web

```

在移除了一个堆栈里的所有服务之后，该堆栈也被自动移除了，如下图所示，移除掉 `app` 堆栈里的所有服务后，该堆栈也被自动移除了：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517565257416.png)

## 6. 总结

本节实验中我们学习了以下内容：

1. 安装 Docker Swarm
2. 部署和管理 Swarm 集群

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。



# Docker API

## 1. 课程说明

课程为纯动手实验教程，为了能说清楚实验中的一些操作会加入理论内容。理论内容我们不会写太多，已经有太多好文章了，会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

实验环境中可以联网，不受实验楼网络限制。

## 2. 学习方法

实验楼的 Docker 课程包含 14 个实验，每个实验都提供详细的步骤和截图，适用于有一定 Linux 系统基础，想快速上手 Docker 的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容。

如果实验开始部分有推荐阅读的材料，请务必先阅读后再继续实验，理论知识是实践必要的基础。

## 3. 本节内容简介

Docker 提供的 API 非常丰富，包括 Registry API，Docker Hub API，Docker OAuth API，Docker Remote API，在本节实验中，我们将通过实验学习 Docker Remote API。Docker Remote API 可以提供 `docker` 命令的全部功能，实验中我们使用 curl 命令来进行测试和学习。

本节中，我们需要依次完成下面几项任务：

1. Docker API 基本概念与认证
2. 使用 API 管理容器：创建，查看，删除等操作
3. 使用 API 管理镜像：创建，查看，删除等操作
4. 使用 API 管理数据卷：创建，查看，删除等操作
5. 使用 API 管理网络：创建，查看，删除等操作

对于 `Docker` 的镜像仓库来说，国内访问速度较慢，我们添加一个阿里云提供的 `Docker` 镜像加速器。

首先，我们需要添加编辑 `/etc/docker/daemon.json` 文件，加入如下内容：

```bash
{
  "registry-mirrors": ["https://n6syp70m.mirror.aliyuncs.com"]
}

```

修改之后，需要重启 `docker` 服务，让修改生效。使用如下命令：

```bash
$ sudo service docker restart

```

查看 Docker API 的版本：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521542295486.png-wm)

## 4. Docker Remote API 基本概念与认证

### 4.1 监听地址

Docker Remote API 是由 Docker 守护进程提供的，默认情况下 Docker 守护进程可以由本地连接访问 Remote API，如果要提供远程访问，则需要绑定到网络接口上。例如，我们通常会添加下面的配置：

```
-H=tcp://0.0.0.0:2375

```

这句表示将 Docker 守护进程监听到所有的网络接口的 2375 端口上。

修改配置文件 `/etc/default/docker` ：

```bash
$ sudo vi /etc/default/docker

```

在末尾添加以下配置：

```
DOCKER_OPTS="-H=tcp://0.0.0.0:2375 -H=unix:///var/run/docker.sock"

```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971521539684174-wm)

> 配置选项 `-H=unix:///var/run/docker.sock` 是允许本地访问连接。
>
> 不要忘记重启 Docker 服务让配置文件起作用： `sudo service docker restart`

重启后我们可以进行测试：

```bash
$ docker -H localhost:2375 info

```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid3858labid1716timestamp1459150490614.png)

为了避免每次`docker` 命令都输入`-H`参数，我们设置环境变量`DOCKER_HOST`：

```bash
$ export DOCKER_HOST=tcp://localhost:2375

```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid3858labid1716timestamp1459150530366.png)

现在 `docker` 命令连接的就是 Docker Remote API 的接口。

### 4.2 使用 TLS 认证

在前面的 `Docker安全` 实验中，我们学习了如何用 TLS 保护 Docker 守护进程，过程与此处完全相同，经过 TLS 服务器端证书保护的 Docker Remote API，需要客户端采用同样 CA 签发的证书才可以连接。

### 4.3 Remote API 使用方法

后续的实验中，我们将学习最常用的 API 接口，我们将使用 `curl` 命令进行实验。

例如 `GET /info` API，需要在 Xfce 终端中输入下面的命令：

```bash
$ curl http://127.0.0.1:2375/info

```

输出的结果如下，类似 `docker info` 命令的输出，不过是个 JSON 格式的内容，可以使用 `python -mjson.tool` 命令对输出信息进行美化：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid3858labid1716timestamp1459150776305.png)

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid3858labid1716timestamp1459150782951.png)

也可以打开 firefox 浏览器，输入如下地址然后回车查看到返回的 json 数据：

![实验楼](https://dn-simplecloud.shiyanlou.com/87971521541061909-wm)

对于 `POST` 操作，仍然可以用 curl 进行测试：

```bash
$ curl -X POST -H "Content-Type: application/json" \
  http://127.0.0.1:2375/containers/create \
  -d '{
        "Image": "redis"
  }'

```

> 如果提示没有镜像的话，可以先使用 docker pull redis 拉取镜像。

执行过程如下，我们会创建一个 redis 容器，并得到 JSON 格式的输出结果：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521614320222.png-wm)

## 5. 使用 API 管理容器：创建，查看，删除等操作

### 5.1 查看所有容器

```
GET /containers/json

```

实验需求是查找所有的容器（包含关机状态的容器），显示最后创建的一个，同时返回容器的大小。

针对这个需求我们将用到这个接口的三个参数：

1. all=1 所有的容器（包含关机状态的容器）
2. limit=1 显示最后创建的一个
3. size=1 返回容器的大小

执行的命令：

```bash
$ curl http://127.0.0.1:2375/containers/json\?all\=1\&limit\=1\&size\=1

```

注意：这里不加 \ 会报错

类比命令：

```
$ docker container ls -a -n 1 -s

```

操作过程截图：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid3858labid1716timestamp1459150967484.png)

### 5.2 创建容器

```
POST /containers/create

```

创建容器的参数非常多，可以回忆下 `docker run` 的参数。

实验需求创建一个 nginx 容器，将容器的 80 端口映射到宿主机 80 端口，挂载宿主机的 `/home/shiyanlou/data` 目录作为数据卷到容器中的 `/data` 目录。

该接口的所有参数都使用 JSON 格式。

定义输入的参数：

```bash
$ curl -X POST -H "Content-Type: application/json" \
http://127.0.0.1:2375/containers/create?name=test_nginx \
-d '{
    "Image": "nginx",
    "HostConfig": {
        "Binds": ["/home/shiyanlou/data:/data"],
        "PortBindings": {"80/tcp": [{"HostPort": "81"}]}
    }
}'

```

操作过程：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521703653568.png-wm)

创建后使用 `docker container inspect` 验证。

此时用浏览器访问地址 `localhost:81` 可以看到 nginx 的页面：

![实验楼](https://dn-simplecloud.shiyanlou.com/87971521703795744-wm)

### 5.3 删除指定的容器

```
DELETE /containers/(id)

```

如果尝试删除一个运行中的容器，需要使用参数`force=1`。

操作过程首先查找容器 ID，然后使用 curl 执行 DELETE 操作：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521704169448.png-wm)

### 5.4 其他接口

其他常用的接口使用方法，主要参数都是容器的 ID，下面列出常用 docker 命令对应的 API：

1. `docker container inspect`：`GET /containers/(id)/json`
2. `docker container top`：`GET /containers/(id)/top`
3. `docker container logs`：`GET /containers/(id)/logs`
4. `docker container export`：`GET /containers/(id)/export`
5. `docker container start`：`POST /containers/(id)/start`
6. `docker container attach`：`POST /containers/(id)/attach`

可以参照上面三个操作进行实验。

## 6. 使用 API 管理镜像：创建，查看，删除等操作

### 6.1 查看所有镜像

```
GET /images/json

```

查看当前系统中的所有镜像：

```bash
$ curl http://127.0.0.1:2375/images/json | python -mjson.tool 

```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971521704613733-wm)

### 6.2 拉取镜像

```
POST /images/create

```

从 Docker Hub 拉取 busybox 镜像：

```bash
$ curl -X POST http://127.0.0.1:2375/images/create\?fromImage\=busybox:ubuntu-14.04

```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971521706974381-wm)

该接口执行的过程中，镜像下载的进度也会输出到屏幕上。

### 6.3 删除镜像

```
DELETE /images/(name)

```

删除刚刚拉取的 busybox 镜像：

```bash
curl -X DELETE http://127.0.0.1:2375/images/busybox

```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid3858labid1716timestamp1459153404158.png)

### 6.4 其他接口

其他常用的接口使用方法，主要参数都是容器的 ID，下面列出常用 docker 命令对应的 API：

1. `docker image inspect`：`GET /images/(name)/json`
2. `docker image tag`：`POST /images/(name)/tag`
3. `docker image push`: `POST /images/(name)/push`
4. `docker image build`：`POST /build`
5. `docker search`：`GET /images/search`

## 7. 使用 API 管理数据卷：创建，查看，删除等操作

### 7.2 创建数据卷

```
POST /volumes/create

```

创建一个名字为 shiyanlou 的数据卷，可以在返回信息中看到数据卷的挂载点

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521705366146.png-wm)

### 7.1 查看数据卷

```
GET /volumes

```

查看系统中的所有数据卷：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521705472113.png-wm)

### 7.3 删除数据卷

```
DELETE /volumes/(name)

```

删除刚刚创建的数据卷 shiyanlou：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521705583679.png-wm)

## 8. 使用 API 管理网络：创建，查看，删除等操作

### 8.1 列出系统中所有的网络

```
GET /networks

```

列出系统中所有的网络:

![实验楼](https://dn-simplecloud.shiyanlou.com/87971521705724622-wm)

### 8.2 创建新的网络

```
POST /networks/create

```

创建一个新的网络，名字为 shiyanlou，驱动类型为`bridge`，配置网段为`172.10.0.0/16`。

创建过程：

```bash
$ curl -X POST -H "Content-Type: application/json" \
http://127.0.0.1:2375/networks/create \
-d '{
    "Name": "shiyanlou",
    "Driver": "bridge",
    "IPAM": {"Config": [{"Subnet": "172.10.0.0/16"}]}
}'

```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521707347839.png-wm)

### 8.3 连接容器到网络

```
POST /networks/(id)/connect

```

创建一个 redis 容器：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521708131392.png-wm)

连接容器到新创建的 shiyanlou 网络：

```bash
curl -X POST -H "Content-Type: application/json" http://127.0.0.1:2375/networks/shiyanlou/connect -d '{"Container": "e132d"}'

```

> e132d 为 容器 id。

### 8.4 删除网络

```
DELETE /networks/(id)

```

首先把关联的容器断开，然后删除网络：

```bash
$ curl -X POST -H "Content-Type: application/json" http://127.0.0.1:2375/networks/shiyanlou/disconnect -d '{"Container": "e132d"}'

$ curl -X DELETE http://127.0.0.1:2375/networks/shiyanlou
$ curl http://127.0.0.1:2375/networks/shiyanlou

```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521708475096.png-wm)

## 9. 总结

本节实验中我们学习了以下内容：

1. Docker API 基本概念与认证
2. 使用 API 管理容器：创建，查看，删除等操作
3. 使用 API 管理镜像：创建，查看，删除等操作
4. 使用 API 管理数据卷：创建，查看，删除等操作
5. 使用 API 管理网络：创建，查看，删除等操作

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。



# 基于 Docker API 开发应用

## 1. 课程说明

课程为纯动手实验教程，为了能说清楚实验中的一些操作会加入理论内容。理论内容我们不会写太多，已经有太多好文章了，会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

实验环境中可以联网，不受实验楼网络限制。

## 2. 学习方法

实验楼的 Docker 课程包含 14 个实验，每个实验都提供详细的步骤和截图，适用于有一定 Linux 系统基础，想快速上手 Docker 的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容。

如果实验开始部分有推荐阅读的材料，请务必先阅读后再继续实验，理论知识是实践必要的基础。

## 3. 本节内容简介

在本实验里我们将基于 Docker API 开发应用学习 Docker API。本实验将使用 Python 语言，应用尽量实现的简单，让不熟悉 Python 语言的同学也能够完成。

本节中，我们需要依次完成下面几项任务：

1. `docker` 包的安装与使用
2. 获取容器信息
3. 使用 Dockerfile 创建镜像

对于 `Docker` 的镜像仓库来说，国内访问速度较慢，我们添加一个阿里云提供的 `Docker` 镜像加速器。

首先，我们需要添加编辑 `/etc/docker/daemon.json` 文件，加入如下内容：

```bash
{
  "registry-mirrors": ["https://n6syp70m.mirror.aliyuncs.com"]
}

```

修改之后，需要重启 `docker` 服务，让修改生效。使用如下命令：

```bash
$ sudo service docker restart

```

## 4. `docker` 包的安装与使用

本实验中将使用 `docker` 包进行实现。 `docker` 是对 Docker Remote API 的 Python 封装，可以通过调用 Python 类和函数来实现 Docker Remote API 的操作，比如创建容器，镜像管理等。

> docker 包的详细文档见： http://docker-py.readthedocs.org/en/latest/ 。其中包含的方法很多，要学会举一反三，学会使用官方文档。

### 4.1 更新 pip 源

由于阿里云源地址的变动，本实验环境中的 pip 源需要重新配置。首先使用以下命令打开 pip 的配置文件，修改其中的内容：

```bash
$ vim ~/.pip/pip.conf

```

> 实验环境中的 pip.conf 已经有内容，需要修改为和给出的内容一致

修改后配置文件中的内容为：

```
[global]
index-url = https://mirrors.aliyun.com/pypi/simple/

[install]
trusted-host=mirrors.aliyun.com

```

然后使用以下命令，更新 pip 的版本：

```bash
$ sudo pip install --upgrade pip

```

### 4.2 配置远程访问

Docker Remote API 是由 Docker 守护进程提供的，默认情况下 Docker 守护进程可以由本地连接访问 Remote API，如果要提供远程访问，则需要绑定到网络接口上。例如，我们通常会添加下面的配置：

```
-H=tcp://0.0.0.0:2375

```

这句表示将 Docker 守护进程监听到所有的网络接口的 2375 端口上。

修改配置文件 `/etc/default/docker` ：

```bash
$ sudo vi /etc/default/docker

```

在末尾添加以下配置：

```
DOCKER_OPTS="-H=tcp://0.0.0.0:2375 -H=unix:///var/run/docker.sock"

```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971521539684174-wm)

> 配置选项 `-H=unix:///var/run/docker.sock` 是允许本地访问连接。
>
> 不要忘记重启 Docker 服务让配置文件起作用： `sudo service docker restart`

重启后我们可以进行测试：

```bash
$ docker -H localhost:2375 info

```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid3858labid1716timestamp1459150490614.png)

如果出现以上结果，就说明远程连接成功了，那么我们就可以使用 python 连接 Docker 了。

### 4.3 配置虚拟环境

使用 `virtualenv` 配置虚拟环境：

```bash
$ cd ~
# 安装 virtualenv
$ sudo pip install virtualenv

# 由于实验环境中没有 Python3.5，而相关包对 Python3.5 有依赖，这里先安装 Python3.5
$ sudo add-apt-repository ppa:fkrull/deadsnakes
$ sudo apt-get update
$ sudo apt-get install python3.5

# 使用 python3.5 ,venv 为虚拟环境目录名
$ virtualenv -p /usr/bin/python3.5 venv

# 激活虚拟环境
$ source venv/bin/activate
**注意：后面的全部实验操作都在虚拟环境中进行。**

```

### 4.4 安装 ipython

在虚拟环境中执行如下命令：

```bash
$ pip3 install ipython

```

### 4.5 安装 `docker` 包

在虚拟环境中输入下面的命令进行安装：

```bash
$ pip3 install docker

```

### 4.6 基本使用

安装完成后，我们可以在 python 中通过 `docker` 模块来使用。

在虚拟环境中输入 ipython，本节后续步骤在 ipython 下进行操作。

连接和操作 Docker 服务或 Swarm 服务的基础类需要首先实例化客户端：

```python
import docker
client=docker.DockerClient(base_url='tcp://127.0.0.1:2375')
client.containers.list()
client

```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521714956880.png-wm)

> 此处配置一个连接地址 `tcp://127.0.0.1:2375` 实例化一个客户端。

> 如果 `client.containers.list()` 的输出为空，说明 Docker 中没有容器，我们可以新建一个容器，然后再查看，使用如下命令新建容器：`docker container run -itd ubuntu /bin/bash`。

之前我们用 `curl` 命令调用 Docker Remote API 启动了 nginx 容器，这里我们用 python 实现同样效果。绑定主机的 `/home/shiyanlou/data` 目录和容器的 `/data` 目录，绑定主机端口 `84` 和容器端口 `80` ：

```python
container=client.containers.run(name='syl_nginx',image='nginx:latest',volumes={'/home/shiyanlou/data':{'bind':'/data','mode':'rw'}},ports={'80/tcp':84},detach=True)

```

> 其中 `detach` 表示立即在后台启动容器，类似 `docker run -d` 。这里为 true，返回了容器对象。

> 这里指定镜像 `image='nginx:latest'`，如果不加 `latest` 则必须先在终端输入 `docker pull nginx:latest` 先拉取镜像，否则此过程会持续很长时间。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521772891008.png-wm)

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521772989578.png-wm)

接下来用浏览器访问地址 `localhost:84` 可看到 nginx 欢迎页面：

![实验楼](https://dn-simplecloud.shiyanlou.com/87971521773045986-wm)

然后我们尝试列出当前系统的所有镜像：

```python
client.images.list()

```

![图片描述](https://doc.shiyanlou.com/courses/uid600404-20190809-1565330083433)

## 5. 获取容器信息

上面我们简单学习了 docker 包的基本用法，接下来学习如何获取容器信息。

使用到的 API：

1. `client.containers.list()` 获得容器列表及基本信息
2. 容器对象的各种属性

实验可以继续在 ipython 中执行，最后将执行无误的代码合并成 python 脚本。

为了本次实验，我们需要多创建几个容器：

首先在终端中执行 `docker pull redis:latest` 命令获取最新版本的 redis 镜像，然后在 ipython 中执行如下命令：

```python
c1=client.containers.run(image='redis',detach=True)
c2=client.containers.create(image='nginx',volumes={'/home/shiyanlou/data':{'bind':'/data','mode':'rw'}},ports={'80/tcp':85})

```

nginx 使用的 create 创建，并没有启动，我们来启动它：

```python
c2.start()

```

首先我们需要获得所有的容器的基本信息：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521775587125.png-wm)

> `all = True` 代表列出所有容器，不加这个参数只会列出运行中的容器。

返回的 containers 列表中已经包含了我们所需的大部分信息，下一步是继续获取其他相关信息。

使用 `results` 字典作为返回信息的存储，获得容器的各种属性，如容器 id 前 10 个字符，容器的镜像，容器的运行进程，容器的流统计信息。

```python
results={}

container_list=client.containers.list()

for container in container_list:
     results['short_id']=container.short_id
     results['image']=container.image
     results['top']=container.top()
     results['stats']=container.stats()

```

上述程序的输出：

![实验楼](https://dn-simplecloud.shiyanlou.com/87971521784976450-wm)

## 6. 使用 Dockerfile 创建镜像

依然在虚拟环境中执行如下操作。

学习了获取容器，接下来我们学习 python 的 docker 包从 dockerfile 创建镜像。需要调用 `client.images.build()` 方法。

### 6.1 程序结构

我们将要实现的程序放在 `/home/shiyanlou/build-image.py`，使用 vim 或 gedit 打开该文件输入下面的内容，初始化 `Client` 对象：

```python
import docker

client=docker.DockerClient(base_url='tcp://127.0.0.1:2375')

```

程序逻辑很简单，包含两部分内容：

1. 处理参数：判断是否输入的是两个参数，读取第一个参数判断是否为路径并且路径下存在 Dockerfile。
2. 如果存在 Dockerfile，就调用创建镜像的 API 创建镜像。

### 6.2 处理逻辑

判断是否输入了两个参数：

```python
import sys

if len(sys.argv) != 3:
    print("Parameter ERROR:args number")
    sys.exit()

```

判断第一个参数是否为路径并且路径存在 Dockerfile，如果存在则创建镜像，如果不存在，就返回错误信息：

```python
import os

if os.path.isdir(sys.argv[1]):
    dockerFile = os.path.join(sys.argv[1],'Dockerfile')
    if os.path.exists(dockerFile):
        #调用创建镜像的 API
        image=client.images.build(path=sys.argv[1],tag=sys.argv[2])
        #打印镜像信息
        print(image)
        #列出所有镜像
        print(client.images.list())
else:
    print("build error,not found Dockerfile")

```

> `path` ：Dockerfile 所在路径
>
> `tag` ：定义的镜像标签

完整的程序文件内容如下：

```python
import sys
import os
import docker

client=docker.DockerClient(base_url='tcp://127.0.0.1:2375')

if len(sys.argv) != 3:
    print("Parameter ERROR:args number")
    sys.exit()

if os.path.isdir(sys.argv[1]):
    dockerFile = os.path.join(sys.argv[1],'Dockerfile')
    if os.path.exists(dockerFile):
        image=client.images.build(path=sys.argv[1],tag=sys.argv[2])
        print(image)
        print(client.images.list())
else:
    print("build error,not found Dockerfile")

```

### 6.4 测试程序

创建测试所需的 Dockerfile 文件：

```bash
$ mkdir /home/shiyanlou/imagetest
$ cd /home/shiyanlou/imagetest
$ vim Dockerfile

```

输入 Dockerfile 的内容可以很简单：

```bash
FROM ubuntu:14.04

ENV HOSTNAME=shiyanlou

```

从 Dockerfile 构建容器：

```bash
$ python build-image.py /home/shiyanlou/imagetest build_test

```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971521795675096-wm)

查看最终创建的镜像：

```bash
$ docker image ls

```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1521796489930.png-wm)

## 7. 总结

本节实验中我们学习了以下内容：

1. python `docker` 包的安装与使用
2. 获取容器信息
3. 使用 Dockerfile 创建镜像

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。