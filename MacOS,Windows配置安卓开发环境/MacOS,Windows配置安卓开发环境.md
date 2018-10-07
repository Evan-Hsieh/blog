# MacOS,Windows配置安卓开发环境

本文介绍了安卓开发环境的配置方法。若有错漏，烦请斧正。
* 作者：Evan Xie
* 邮箱：evanyixie@gmail.com

# 0. 概述
安卓开发环境的配置可分为如下几步：
* 配置JDK
* 配置代理
* 安装配置Android Studio
* 下载Android SDK

# 1. 配置JDK
## 1.1 Windows
### 下载JDK
访问[Oracle官网JDK下载页面](https://www.oracle.com/technetwork/java/javase/downloads/index.html),可以看到有若干版本的JDK（即Java SE）版本提供下载。
若没有合适的版本，可在页面底部找到Java Archive，在其中选择合适的版本进行下载。

* 若选择exe版本下载，则双击安装即可。
* 若选择zip版本下载，解压后放到合适目录即可。

### 配置JAVA环境变量
环境变量的作用是程序能够获取到JDK的安装路径。
+ 右击我的电脑-属性-高级-系统配置-高级-环境变量。
+ 将JDK的安装路径配置为JAVA_HOME。
+ 同时添加JAVA_HOME/bin到PATH中。
> 以最新的情况来看，JDK目录下的其他目录，如tools等不需要再配置。

### 验证JDK安装情况
打开cmd，输入java -version，若显示JAVA版本即表示安装配置完成。

## 1.2 Mac OS
### 方法一：使用默认JDK
在.bash_profile中添加：
```
# The path of system java which has been installed in the Mac OS
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdkXXX.jdk/Contents/Home
```

### 方法二：手动下载配置
下载方式同Windows方法。
配置环境变量则需打开用户目录下的.bash_profile,添加：
```
export JAVA_HOME=<pathToJdk>
```
并执行
```
source .bash_profile
```

### 方法三：使用homebrew
安装homebrew后，执行
```
brew cask search java
brew cask install java
```
需要安装 JDK 7 或者 JDK 6，可以使用homebrew-cask-versions：
```
brew tap caskroom/versions
brew cask install java6
```



# 2. 配置代理或镜像服务器
为了能够下载SDK或其他资源，一般情况下都需要配置代理。本文中将**代理**广义地理解为转发代理与镜像服务器。
* 代理服务器指客户端与目标服务器之间的中介，可以理解为客户端每一次请求，都是由该代理服务器转发给目标服务器的，响应也是由该代理转发回来的。
* 镜像服务器并不做转发，其与目标服务器保持同步，直接给客户端提供服务。可以理解为，镜像服务器将目标服务器上的资源复制了一份，当客户端请求相应资源时，直接将所需资源返回。

## 使用转发代理
通过上面的介绍，可以知道转发代理并不限定请求的资源（镜像服务器只能返回已经复制的资源。），同时由于实时地将相应返回，故获取到的资源也是最新的。所以，如果条件允许，建议使用转发代理。

介绍转发代理的文章很多，本文就暂不做介绍了。

## 使用镜像服务器
目前国内的镜像服务器一般由若干高校提供，主要有：

中国科学院开源协会镜像站地址:
IPV4/IPV6: http://mirrors.opencas.cn 端口：80
IPV4/IPV6: http://mirrors.opencas.org 端口：80

上海GDG镜像服务器地址:
http://sdk.gdgshanghai.com 端口：8000

北京化工大学镜像服务器地址:
IPv4: http://ubuntu.buct.edu.cn/ 端口：80
IPv6: http://ubuntu.buct6.edu.cn/ 端口：80

镜像服务器的特点是免费，但提供的资源有限。若在使用时发现无法下载，可以更换其他服务器。


# 3. 安装Android Studio
* 登录[安卓开发者网站](https://developer.android.com/studio/),点击下载Android Studio
> * 网上的其他教程需要手动额外下载Android SDK，但如果配置代理后，可以通过SDK manager来进行下载。本文将会在下文以这种方式下载SDK。
> * 若不想使用Android Stuidio作为IDE进行开发，可在该页面中下载命令行工具，使用其中的Sdk manager进行下载。

* 安装Android Studio

# 4. 下载Android SDK
启动Android Studio时，若以前没有安装过Android SDK，则会无法启动。

以Android Studio 3.2为例，在没有安装SDK的情况下，会提示用户选择代理。输入代理的host与端口，即可下载Android SDK。 

![img_downloadAndroidSdk](https://i.loli.net/2018/10/07/5bb9d56d9195b.png)

下载完成后，即可正常启动。

为了保证Android的一些工具（如adb)也能正常使用，需将Android SDK的两个子目录platform-tools与tools配置到环境变量PATH中。