---
title: Linux安装jdk-tomcat教程
date: 2021-01-31 23:02:08
image: /images/title_images/jdk_logo.jpg  #设置本地图片
keywords: jdk安装, tomcat安装,
tags: 
---

#  1. JDK 安装

#### 1.1 [#](#下载地址)下载地址

http://www.oracle.com/technetwork/java/javase/downloads/index.html

#### 1.2 [#](#解压缩并移动到指定目录)解压缩并移动到指定目录

```bash
root@UbuntuBase: tar -zxvf jdk-8u152-linux-x64.tar.gz  #解压缩
root@UbuntuBase: mkdir -p /usr/local/java              #创建目录
root@UbuntuBase: mv jdk1.8.0_152/ /usr/local/java/     #移动安装包
root@UbuntuBase: chown -R root:root /usr/local/java/   #设置所有者
```

#### 1.3 [#]()配置jdk系统环境变量

```bash
#配置jdk系统环境变量
root@UbuntuBase: nano /etc/environment

#添加如下语句
PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games"
export JAVA_HOME=/usr/local/java/jdk1.8.0_152
export JRE_HOME=/usr/local/java/jdk1.8.0_152/jre
export CLASSPATH=$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib
```

#### 1.4 [#](#配置用户环境变量)配置用户环境变量

```bash
#配置用户环境变量
root@UbuntuBase: nano /etc/profile

#添加如下语句
if [ "$PS1" ]; then
  if [ "$BASH" ] && [ "$BASH" != "/bin/sh" ]; then
    # The file bash.bashrc already sets the default PS1.
    # PS1='\h:\w\$ '
    if [ -f /etc/bash.bashrc ]; then
      . /etc/bash.bashrc
    fi
  else
    if [ "`id -u`" -eq 0 ]; then
      PS1='# '
    else
      PS1='$ '
    fi
  fi
fi

export JAVA_HOME=/usr/local/java/jdk1.8.0_152
export JRE_HOME=/usr/local/java/jdk1.8.0_152/jre
export CLASSPATH=$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib
export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH:$HOME/bin

if [ -d /etc/profile.d ]; then
  for i in /etc/profile.d/*.sh; do
    if [ -r $i ]; then
      . $i
    fi
  done
  unset i
fi
```

#### 1.5 [#](#使用户环境变量生效)使用户环境变量生效

```bash
root@UbuntuBase: source /etc/profile
```

#### 1.6 [#](#测试是否安装成功)测试是否安装成功

```bash
root@UbuntuBase:/usr/local/java# java -version
java version "1.8.0_152"
Java(TM) SE Runtime Environment (build 1.8.0_152-b16)
Java HotSpot(TM) 64-Bit Server VM (build 25.152-b16, mixed mode)
```

#### 1.7 [#](#为其他用户更新用户环境变量)为其他用户更新用户环境变量

```bash
root@UbuntuBase: su lusifer
root@UbuntuBase: source /etc/profile
```



# 2. Tomcat 安装

#### 2.1 下载地址

https://tomcat.apache.org/

#### 2.2 [#](https://www.funtl.com/zh/linux/Linux-安装-Tomcat.html#解压缩并移动到指定目录)解压缩并移动到指定目录

```bash
root@UbuntuBase: tar -zxvf apache-tomcat-8.5.23.tar.gz #变更目录名
root@UbuntuBase: mv apache-tomcat-8.5.23 tomcat        #变更目录名
root@UbuntuBase: mv tomcat/ /usr/local/                #移动目录
```

#### 2.3 [#]()常用命令

```bash
root@UbuntuBase: /usr/local/tomcat/bin/startup.sh  #启动
root@UbuntuBase: /usr/local/tomcat/bin/shutdown.sh #停止
```

