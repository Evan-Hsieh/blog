
### 安装docker
#### MacOS平台安装
* homebrew
通过在命令行中使用homebrew下载安装
```
brew cask install docker
```
* 官网下载dmg文件安装
在[Docker官网](https://www.docker.com/products/docker-desktop)下载dmg文件，双击后拖拽到Application目录即可。

安装后，执行
```
~ # docker --version
Docker version 18.09.0, build 4d60db4
```
显示版本为18.09，表示安装成功。

### Docker核心操作

#### 镜像与仓库交互命令
* login ： 登录
* logout : 登出
* search : 搜索镜像
* pull ： 下载镜像
* push ： 上传镜像

#### 实践操作范例
* 登录/登出

通过login可以登录Docker镜像仓库，若不指定服务器，默认连接官方仓库Docker Hub.
```
~ # docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: evanxie
Password:
Login Succeeded
 ~ # docker logout
Removing login credentials for https://index.docker.io/v1/
```
使用login命令后，输入用户名与密码即可登录，只用logout即可退出。

* 搜索/下载镜像

获得镜像（image）的方式有多种，比较推荐的一种方式是从网上下载镜像。通过docker serarch命令即可搜索，下面以搜索与下载centos镜像为例，结果如下：
```
~ # docker search centos
NAME                               DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
centos                             The official build of CentOS.                   4934                [OK]
ansible/centos7-ansible            Ansible on Centos7                              119                                     [OK]
jdeathe/centos-ssh                 CentOS-6 6.10 x86_64 / CentOS-7 7.5.1804 x86…   99                                      [OK]
```
通过搜索命令，得到多个结果，其中第一个是官方centos镜像。

通过docker pull命令可以下载镜像
```
~ # docker pull centos
Using default tag: latest
latest: Pulling from library/centos
aeb7866da422: Pull complete
Digest: sha256:67dad89757a55bfdfabec8abd0e22f8c7c12a1856514726470228063ed86593b
Status: Downloaded newer image for centos:latest
```

下载后，可以通过docker images查看镜像
```
~ # docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
centos              latest              75835a67d134        5 weeks ago         200MB
```

#### 容器生命周期管理命令
* run : 从镜像创建容器,并运行一个命令
* create ： 从镜像创建容器，但是不运行
* exec ： 在容器中执行命令
* start/stop/restart ： 启动/停止/重启容器
* kill ：杀死正在运行的容器
* rm ：删除容器
* pause/unpause ：停止或回复容器中的所有进程

#### 实践操作范例
* run

使用run命令可以从镜像centos:latest创建容器，并且运行命令bin/bash，其中-i表示以交互模式运行，-t表示为容器分配一个伪输入终端
```
~ # docker run -it centos:latest /bin/bash
[root@c1bbd66a1bf9 /]#
```
使用exit命令可以退出并关闭容器，使用Ctrl+P+Q可以在不停止容器的情况下退出容器。

* start/stop/restart
通过ps -a命令可以查看所有容器信息：
```
~ # docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                        PORTS               NAMES
c1bbd66a1bf9        centos:latest       "/bin/bash"         2 minutes ago       Exited (127) 13 seconds ago                       unruffled_blackwell
```
可以知道目前我们的容器ID为c1bbd66a1bf9。知道ID后可以对容器进行操作：
```
~ # docker start c1bbd66a1bf9
c1bbd66a1bf9
~ # docker stop c1bbd66a1bf9
c1bbd66a1bf9
~ # docker restart c1bbd66a1bf9
c1bbd66a1bf9
```
* rm

在容器没有运行的情况下，可以使用rm删除容器，并查看容器是否还存在
```
~ # docker restart c1bbd66a1bf9
c1bbd66a1bf9
~ # docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```
从上面的结果中可以看到，目前已经没有容器。

* create

最初的容器是通过run命令创建并运行的。如果使用create命令，也可以创建容器，但是并不运行，其结果实例如下：
```
~ # docker create centos:latest
7426616f8402ad4e2b44bec10286f8906132d3dcd4c036b922b6bd884da03993
~ # docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
7426616f8402        centos:latest       "/bin/bash"         7 seconds ago       Created                                 elated_gagarin
```
可以看到容器的状态STATUS是Created表示被创建。

* exec

使用start可以启动容器，并使用exec可以执行命令，尝试结果如下：
```
~ # docker start 7426616f8402
7426616f8402
~ # docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
7426616f8402        centos:latest       "/bin/bash"         4 minutes ago       Exited (0) 3 seconds ago                       elated_gagarin
~ # docker exec 7426616f8402 -it /bin/bash
Error response from daemon: Container 7426616f8402ad4e2b44bec10286f8906132d3dcd4c036b922b6bd884da03993 is not running
```
使用start发现正常启动了容器，但是使用ps查看其状态，他的状态是Exited.然后使用exec对该容器执行/bin/bash命令时，发现报错了，报错原因是该容器没有运行。

* 容器状态
实际上，容器主要有5种状态：

| 状态名  | 描述   |
| :---: | :---: | 
| Created | 创建容器后的状态 |
| Running | 运行状态 |
| Stopped | 停止状态|
| Exited | 退出状态 | 
| Dead | 死亡状态 |

前面的问题就出在使用start命令后，容器并不在running运行状态，而是在Exited状态。其根本原因是容器中运行的进程如果退出后，那么容器也会退出，也就是会变成Exited状态。

为了让容器中能有一个持久运行的进程，我们需要在运行容器时，也执行一个命令。由于start命令后不能接上其他命令，所以需要通过run命令来执行其他命令：
```
 ~ # docker run -t -d centos:latest
6e053b799fb28a0d62c972aa3128b2d677ed9e0c416b1dac8807a4901c424fdd
 ~ # docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
6e053b799fb2        centos:latest       "/bin/bash"         2 seconds ago       Up 1 second                                     cranky_chatterjee
 ~ # docker exec 6e053b799fb2 /bin/bash
 ~ # docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS                          PORTS               NAMES
6e053b799fb2        centos:latest       "/bin/bash"         31 seconds ago       Up 29 seconds                                       cranky_chatterjee
```
使用-d参数是让容器在后台运行，-t使用伪终端，并且在伪终端会在后台一直运行，所以执行ps查看其STATUS就不是Exited了，并且执行exec也不会报错了。

* pause/unpause
```
 ~ # docker pause 6e053b799fb2
6e053b799fb2
 ~ # docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
6e053b799fb2        centos:latest       "/bin/bash"         8 minutes ago       Up 8 minutes (Paused)                           cranky_chatterjee
 ~ # docker unpause 6e053b799fb2
6e053b799fb2
 ~ # docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
6e053b799fb2        centos:latest       "/bin/bash"         8 minutes ago       Up 8 minutes                                    cranky_chatterjee
```
通过使用pause命令可以暂停容器中的所有进程，容器的状态会变为（Paused)，使用unpause即可恢复。

* kill 

```
 ~ # docker run -td centos:latest
cd19eaf270b04f6f457c4b8af1d329aa5f1c4207160010dc948fb5a5cfba8164
 ~ # docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
cd19eaf270b0        centos:latest       "/bin/bash"         3 seconds ago       Up 1 second                             peaceful_ramanujan
 ~ # docker kill cd19eaf270b0
cd19eaf270b0
 ~ # docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
cd19eaf270b0        centos:latest       "/bin/bash"         13 seconds ago      Exited (137) 1 second ago                       peaceful_ramanujan
```
通过run重建一个容器后，可以通过kill杀死一个正在运行的容器。




