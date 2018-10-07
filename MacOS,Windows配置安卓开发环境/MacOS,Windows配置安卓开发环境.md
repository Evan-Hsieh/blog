#MacOS,Windows配置安卓开发环境

本文介绍了安卓开发环境的配置方法。若有错漏，烦请斧正。
作者：Evan Xie
邮箱： evanyixie@gmail.com

# 0. 概述
安卓开发环境的配置可分为如下几步：
* 配置JDK
* 安装配置Android Studio
* 配置代理
* 下载Android SDK

# 1. 配置JDK
## 1.1 Windows
### 下载JDK
访问[Oracle官网JDK下载页面](https://www.oracle.com/technetwork/java/javase/downloads/index.html),可以看到有若干版本的JDK（即Java SE）版本提供下载。
若没有合适的版本，可在页面底部找到Java Archive，在其中选择合适的版本进行下载。

* 若选择exe版本下载，则双击安装即可。
* 若选择zip版本下载，解压后放到合适目录即可。

### 配置JAVA环境变量
环境变量的作用是程序能够获取到JDK的安装路径。
+ 右击我的电脑-属性-高级-系统配置-高级-环境变量。
+ 将JDK的安装路径配置为JAVA_HOME。
+ 同时添加JAVA_HOME/bin到PATH中。
> 以最新的情况来看，JDK目录下的其他目录，如tools等不需要再配置。

### 验证JDK安装情况
打开cmd，输入java -version，若显示JAVA版本即表示安装配置完成。

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