
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
~ docker --version
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
~ docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: evanxie
Password:
Login Succeeded
 ~ docker logout
Removing login credentials for https://index.docker.io/v1/
```
使用login命令后，输入用户名与密码即可登录，只用logout即可退出。

* 搜索/下载镜像

获得镜像（image）的方式有多种，比较推荐的一种方式是从网上下载镜像。通过docker serarch命令即可搜索，下面以搜索与下载centos镜像为例，结果如下：
```
 ~ docker search centos
NAME                               DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
centos                             The official build of CentOS.                   4934                [OK]
ansible/centos7-ansible            Ansible on Centos7                              119                                     [OK]
jdeathe/centos-ssh                 CentOS-6 6.10 x86_64 / CentOS-7 7.5.1804 x86…   99                                      [OK]
```
通过搜索命令，得到多个结果，其中第一个是官方centos镜像。

通过docker pull命令可以下载镜像
```
~ docker pull centos
Using default tag: latest
latest: Pulling from library/centos
aeb7866da422: Pull complete
Digest: sha256:67dad89757a55bfdfabec8abd0e22f8c7c12a1856514726470228063ed86593b
Status: Downloaded newer image for centos:latest
```

下载后，可以通过docker images查看镜像
```
~ docker images
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

从镜像centos:latest创建容器，并且运行命令、bin/bash
```
~ docker run -it centos:latest /bin/bash
[root@c1bbd66a1bf9 /]#
```
使用exit命令可以退出并关闭容器，使用Ctrl+P+Q可以在不停止容器的情况下退出容器。

* start/stop/restart
通过ps -a命令可以查看所有容器信息：
```
docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                        PORTS               NAMES
c1bbd66a1bf9        centos:latest       "/bin/bash"         2 minutes ago       Exited (127) 13 seconds ago                       unruffled_blackwell
```
可以知道目前我们的容器ID为c1bbd66a1bf9。知道ID后可以对容器进行操作：
```
~ docker start c1bbd66a1bf9
c1bbd66a1bf9
~ docker stop c1bbd66a1bf9
c1bbd66a1bf9
~ docker restart c1bbd66a1bf9
c1bbd66a1bf9
```
* rm

在容器没有运行的情况下，可以使用rm删除容器，并查看容器是否还存在
```
~ docker restart c1bbd66a1bf9
c1bbd66a1bf9
~ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```




