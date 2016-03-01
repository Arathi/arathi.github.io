---
layout: post
title:  "虚拟机下RHEL 5.4安装Oracle 10g R2"
date:   2014-01-26 23:30:00 +0800
categories: linux
tags: Oracle Linux
---
### 一、安装RHEL 5.4

正常安装就行了，只要注意，Oracle安装需要图形用户界面。

### 二、yum安装源设置

#### 2.1 虚拟机光驱方式  
首先要插入dvd  

    mount -t auto /dev/cdrom /mnt/cdrom/

然后添加一个yum源，添加/etc/yum.repos.d/rhel-cdrom.repo
<!--more-->

    [rhel-cdrom]
    name=Red Hat Enterprise Linux $releasever - $basearch - CDROM
    baseurl=file:///mnt/cdrom/Server/
    enabled=1
    gpgcheck=0
    gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release

#### 2.2 iso文件方式
(to be write)

### 三、Oracle运行环境检查、设置与安装

(1) 检查网络环境、主机名
uname -n的输出、hostname的输出以及hosts中内网ip对应的名称要一致
(2) 一些硬件要求检查
(to be write)
(3) 一些依赖包的安装
需要依赖的包在RHEL 5.4的安装盘中都有，使用root用户输入以下命令：

    yum -y install binutils compat-db compat-gcc-34 compat-gcc-34-c++ compat-libstdc++-33 compat-libstdc++ gcc gcc-c++ glibc glibc-common glibc-devel glibc-headers libgcc libXp libXt libXtst libaio libaio-devel libstdc++ libstdc++-devel libgomp make numactl-devel sysstat

### 四、添加oracle用户

(1) 新增用户组 oinstall 和 dba  
(2) 新增用户 oracle，设定其用户组为oinstall和dba  
(3) 为oracle用户设置密码  

    groupadd oinstall
    groupadd dba
    useradd -g oinstall -G dba oracle
    passwd oracle
    #然后为该用户输入密码
    usermod -g oinstall -G dba oracle
    ls -l /home | grep oracle

### 五、修改内核参数

修改 /etc/sysctl.conf 文件，在文件尾部添加以下内容：

    fs.file-max = 65536
    kernel.sem = 250 32000 100 128
    kernel.shmmni = 4096
    net.core.rmem_default = 4194304
    net.core.rmem_max = 4194304
    net.core.wmem_default = 262144
    net.core.wmem_max = 262144
    net.ipv4.ip_local_port_range = 1024 65500

修改完执行 sysctl -p，使刚才的设置生效

### 六、设置shell限制

(1) 修改 /etc/security/limits.conf 文件，在文件尾部添加以下内容：

    oracle soft nproc 2047
    oracle hard nproc 16384
    oracle soft nofile 1024
    oracle hard nofile 65536

(2) 编辑 /etc/pam.d/login 文件，添加一行：

    session required pam_limits.so

(3) 修改环境变量配置  
修改系统环境变量配置文件（/etc/profile），添加以下内容：

    #ksh和其他shell的限制设置
    if [ $USER = "oracle" ]; then
        if [ $SHELL = "/bin/ksh" ]; then
            ulimit -p 16384
            ulimit -n 65536
        else
            ulimit -u 16384 -n 65536
        fi
    fi

在 /etc/csh.login 修改csh的限制：

    if ( $USER == "oracle" ) then
        limit maxproc 16384
        limit descriptors 65536
    endif

### 七、建立Oracle安装目录

以下假设Oracle安装在 /home/app/oracle 目录下

    mkdir -p /home/app/oracle
    chown -R oracle:oinstall /home/app/oracle
    chmod -R 775 /home/app/oracle

然后为刚才建立的目录设置环境变量：  
编辑oracle用户的.bash_profile文件，添加以下内容：

    export ORACLE_BASE=/home/app/oracle
    export ORACLE_HOME=/home/app/oracle/product/10201
    export ORACLE_SID=ywxx
    export PATH=$PATH:$HOME/bin:$ORACLE_HOME/bin

然后使用source .bash_profile命令，使环境变量生效

*注意：ywxx可以换成其他的名称*

### 八、上传并解压Oracle安装文件

假设现将安装包(10201_database_linux_x86_64.cpio)上传到/home/oracle/setup/10201_database_linux_x86_64.cpio下。
解压安装包：

    cpio -idmv < 10201_database_linux_x86_64.cpio

然后可以准备安装了。

### ⑨、进入安装界面

进入oracle用户的图形界面，执行安装目录下的`runInstall.sh`，然后按照正常的安装流程走。
不建议服务端、数据库、listener分开装，建议使用第一个模式（Basic Installation）。
不过要注意字符集问题。