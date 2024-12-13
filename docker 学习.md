## docker 学习

版本更新 我在我的电脑上可以运行！

开发即运维哈哈哈哈



环境配置     每一个机器都要部署环境（集群Redis ） 费时费力

服务器配置应用环境（非常麻烦 不能跨平台 配置麻烦）

windows 配置到linux上 



开发＋打包 一套流程



Docker解决了这个方案

下载镜像直接运行



隔离 镜像 打包装箱 每个箱子相互隔离 

Docker 隔离机制 将服务器利用到极致

集群



本质：所有技术 出现问题需要解决 才去学习 才去发展



**开源**

加速一个软件的开发



之前时VM软件（笨重）在Docker出来之前

虚拟机属于虚拟化技术 Docker 容器技术（本质也是虚拟化技术）



dockers 镜像（最核心环境 4m大小）需要什么就安装什么（十分小巧）（只需要运行镜像） 几兆



docker官网：

**基于文档学习！！！！！！**

文档地址： https://docs.docker.com/ Docker文档非常详细

dockerhub： https://hub.docker.com/



- 容器没有内核 用宿主机的内核  也不用虚拟出硬件 所以就轻便了

- 容器内部相互隔离，每个容器内部有自己的文件系统，互不影响

- 测试环境高度一致

Dockers 时内核级别的虚拟化 在1核2g 上可以运行很多实例



Docker架构图

镜像：镜像 run起来 容器 镜像可以创建多个容器 最终服务运行在容器中

容器：Dokcer 利用容器技术 独立运行或者一个独立应用 通过镜像来创建



仓库：

Docker hub

华为云

阿里云等等

国内需要配置镜像加速



 客户端 服务器 仓库



#### 设置docker仓库

看文档啊



默认在国外

run运行 要寻找镜像 去仓库寻找镜像下载 能不能找到这个镜像 



#### 底层原理

Docker是怎么工作的？

Dockers client-server架构 Docker的守护进程运行在主机上 通过socket 从客户端访问

DockerServer 即受到Dockers-Client指令 



![联想截图_20241123123231](C:\Users\30413\Pictures\联想截图\联想截图_20241123123231.png)

容器之间相互隔离 并且外部的linux服务器 与容器也是隔离的访问不到 你需要做些什么 让linux 访问容器



Docker为什么快

- 更少的抽象层

- dockers利用的宿主机的内核 

![image-20241123123614247](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241123123614247.png)

新建容器的时候 不需要向虚拟机一样加载操作系统内核 避免引导 虚拟机加载 Guest Os

docker直接使用宿主机内核



拉取镜像 如果不写 tag 默认 latest

指定版本下载

分层下载 docker image的核心 联合文件系统



`docker run -it centos /bin/bah`

这个指令 相当于启动了linux容器

容器启动的linux 与外部的隔离没有关联



docker 发先 没有前台应用 容器  启动后发现自己没有提供服务 自动停止 只启动centos但没有应用     





# Docker学习



##### 禁用不可使用的仓库

~~~
 sudo yum-config-manager --disable hub-mirror.c.163.com
Repository extras is listed more than once in the configuration
[root@iZt4nbaeq7uzlvq978l1xqZ ~]# sudo yum-config-manager --disable root_docker.registry.cyou
Repository extras is listed more than once in the configuration
[root@iZt4nbaeq7uzlvq978l1xqZ ~]# sudo yum-config-manager --disable root_dockerpull.org
Repository extras is listed more than once in the configuration
[root@iZt4nbaeq7uzlvq978l1xqZ ~]# sudo yum-config-manager --disable root_hub.docker.com
Repository extras is listed more than once in the configuration
[root@iZt4nbaeq7uzlvq978l1xqZ ~]# sudo yum-config-manager --disable root_hub.littlediary.cn
Repository extras is listed more than once in the configuration

~~~



##### 删除不可使用的仓库

~~~
[root@iZt4nbaeq7uzlvq978l1xqZ ~]# sudo rm -f /etc/yum.repos.d/hub-mirror.c.163.com.repo
[root@iZt4nbaeq7uzlvq978l1xqZ ~]# sudo rm -f /etc/yum.repos.d/root_docker.registry.cyou.repo
[root@iZt4nbaeq7uzlvq978l1xqZ ~]# sudo rm -f /etc/yum.repos.d/root_dockerpull.org.repo
[root@iZt4nbaeq7uzlvq978l1xqZ ~]# sudo rm -f /etc/yum.repos.d/root_hub.docker.com.repo
[root@iZt4nbaeq7uzlvq978l1xqZ ~]# sudo rm -f /etc/yum.repos.d/root_hub.littlediary.cn.repo

~~~

##### 列出仓库

~~~
sudo yum repolist
Repository extras is listed more than once in the configuration
repo id                                                            repo name
AppStream                                                          CentOS-8.5.2111 - AppStream - mirrors.cloud.aliyuncs.com
PowerTools                                                         CentOS-8.5.2111 - PowerTools - mirrors.cloud.aliyuncs.com
appstream                                                          CentOS Linux 8 - AppStream
base                                                               CentOS-8.5.2111 - Base - mirrors.cloud.aliyuncs.com
baseos                                                             CentOS Linux 8 - BaseOS
docker-ce-stable                                                   Docker CE Stable - x86_64
epel                                                               Extra Packages for Enterprise Linux 8 - x86_64
epel-archive                                                       Extra Packages for Enterprise Linux 8 - x86_64
extras                                                             CentOS Linux 8 - Extras
powertools                                                         CentOS Linux 8 - PowerTools
registry.docker-cn.com                                             created by dnf config-manager from https://registry.docker-cn.com

~~~





##### 事实证明如果不配置国内镜像源一定会失败



原文链接：https://blog.csdn.net/qq_35794202/article/details/131016299





设置国内镜像【不设置可能会导致拉取镜像失败】
进入/etc/docker文件夹下，修改daemon.json。如果文件不存在则，创建该文件。



daemon.json文件内容如下



~~~
{
"registry-mirrors" : [
    "https://jkfdsf2u.mirror.aliyuncs.com",
    "https://registry.docker-cn.com"
  ],
  "insecure-registries" : [
    "docker-registry.zjq.com"
  ],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "10"
  },
  "data-root": "/data/docker"
}
~~~





### 1. **清除所有现有的 Docker 密钥和配置**

首先，移除可能存在的过时或损坏的 Docker 密钥文件。

```
bash复制代码sudo rm -rf /etc/apt/trusted.gpg.d/docker.gpg
sudo rm /etc/apt/sources.list.d/docker.list
```

### 2. **重新添加 Docker GPG 密钥**

然后，我们重新从 Docker 官方下载并添加 GPG 密钥：

```
bash


复制代码
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo tee /etc/apt/trusted.gpg.d/docker.asc
```

### 3. **确保 Docker 仓库配置正确**

确认 Docker 仓库配置文件存在且正确，您可以手动添加 Docker 仓库。使用以下命令创建配置文件：

```
bash


复制代码
echo "deb [arch=amd64] https://download.docker.com/linux/ubuntu jammy stable" | sudo tee /etc/apt/sources.list.d/docker.list
```

### 4. **更新并安装 Docker**

更新包索引并安装 Docker：

```
bash复制代码sudo apt update
sudo apt install docker-ce
```

### 5. **使用 `apt-key` 强制导入公钥**

如果仍然无法解决问题，可以尝试使用 `apt-key` 强制导入 Docker 的公钥：

```

sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 7EA0A9C3F273FCD8
```

然后再执行更新：

```

sudo apt update
```

### 6. **检查是否有网络或代理问题**

如果您使用的是企业网络或有防火墙限制，可能会影响 GPG 密钥的下载。确保没有网络代理或防火墙阻止访问 `keyserver.ubuntu.com`。

### 7. **查看更详细的错误信息**

如果仍然出现问题，可以通过运行以下命令来查看更详细的错误信息，以帮助进一步诊断：

```
bash


复制代码
sudo apt-get update -o Debug::Acquire::http=true
```

GPT是真的强







### docker start

- **启动已存在的容器:** docker start 命令只能用来启动一个已经存在的容器，而不能创建新的容器。
- **快速启动:** 如果你已经创建了一个容器，并且想多次启动和停止它，那么使用 docker start 就非常方便。





## ocker run 和 docker start 的区别

**docker run** 和 **docker start** 都是用来启动容器的命令，但它们的使用场景和功能略有不同。

### docker run

- **创建并启动容器:** docker run 命令不仅会创建一个新的容器，还会立即启动它。
- **基于镜像:** 这个新创建的容器是基于指定的镜像创建的。
- **配置选项丰富:** 你可以在运行时通过各种参数（如 -d, -p, -v 等）来配置容器的行为。
- **一次性操作:** 如果你只是想运行一次容器，或者每次运行都需要不同的配置，那么 docker run 是一个很好的选择。