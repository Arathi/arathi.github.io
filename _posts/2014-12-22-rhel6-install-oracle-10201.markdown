---
layout: post
title:  "RHEL 6.4安装Oracle 10g R2 (10201)"
date:   2014-12-22 11:01:00 +0800
categories: linux
tags: Oracle Linux
---

## 一. 安装RHEL 6.4

使用预置的Desktop配置，安装1110个包。  
为了避免不必要的麻烦，安装后请关闭iptables和SELinux。

## 二. 配置yum源

### 2.1 新建挂载目录

    mkdir -p /mnt/rheliso

### 2.2 修改root用户的profile，自动挂载安装盘

在`/root/.bash_profile`添加以下内容

    if [ ! -d "/mnt/rheliso/Packages"]; then
        mount /dev/cdrom /mnt/rheliso/
    fi

### 2.3 清空与重建yum缓存

    yum clean all
    yum list

## 三. 检查环境

### 3.1 主机名一致

    uname -n
    hostname
    hosts

### 3.2 安装必要的软件包

执行如下命令：

    yum -y install binutils compat-db compat-gcc-34 compat-gcc-34-c++ \
    compat-libstdc++-33 compat-libstdc++ gcc gcc-c++ glibc glibc-common \
    glibc-devel glibc-headers libgcc libXp libXt libXtst libaio \
    libaio-devel libstdc++ libstdc++-devel libgomp make numactl-devel \
    sysstat glibc.i686 glibc-devel.i686 libXp.i686 libXt.i686 \
    libXtst.i686 libgcc.i686

## 四. 添加oracle用户

### 4.1 新增用户组 oinstall 和 dba

### 4.2 新增用户 oracle，设定其用户组为oinstall和dba

### 4.3 为oracle用户设置密码

以上步骤命令如下：

    groupadd oinstall
    groupadd dba
    useradd -g oinstall -G dba oracle
    passwd oracle
    #然后为输入密码
    usermod -g oinstall -G dba oracle
    ls -l /home | grep oracle

## 五. 内核参数与shell配置

### 5.1 内核参数

修改`/etc/sysctl.conf`文件，在文件尾部添加以下内容：

    fs.file-max = 65536
    kernel.sem = 250 32000 100 128
    kernel.shmmni = 4096
    net.core.rmem_default = 4194304
    net.core.rmem_max = 4194304
    net.core.wmem_default = 262144
    net.core.wmem_max = 262144
    net.ipv4.ip_local_port_range = 1024 65500

修改完执行`sysctl -p`，使刚才的设置生效

### 5.2 shell限制

修改`/etc/security/limits.conf`文件，在文件尾部添加以下内容：

    oracle    soft    nproc    2047
    oracle    hard    nproc    16384
    oracle    soft    nofile   1024
    oracle    hard    nofile   65536

### 5.3 session相关

编辑`/etc/pam.d/login`文件，添加一行：

    session    required     pam_limits.so 

### 5.4 ksh与其他shell的limit修改

修改`/etc/profile`文件，末尾添加：

    if [ $USER = "oracle" ]; then
        if [ $SHELL = "/bin/ksh" ]; then
            ulimit -p 16384
            ulimit -n 65536
        else
            ulimit -u 16384 -n 65536
        fi
    fi

### 5.5 csh的limit修改

在`/etc/csh.login`末尾添加：

    if ( $USER == "oracle" ) then
        limit maxproc 16384
        limit descriptors 65536
    endif

## 六. Oracle安装准备

假设Oracle安装在`/app/oracle`目录下

### 6.1 创建目标目录

    mkdir -p /app/oracle
    chown -R oracle:oinstall /app/oracle
    chmod -R 775 /app/oracle

### 6.2 配置环境变量

oracle用户的环境变量设置如下：

    export ORACLE_BASE=/app/oracle
    export ORACLE_HOME=$ORACLE_BASE/product/10201
    export ORACLE_SID=ywxx
    export PATH=$PATH:$HOME/bin:$ORACLE_HOME/bin

### 6.3 上传并解压安装文件

    mkdir /home/oracle/setup
    #上传10201_database_linux_x86_64.cpio.gz上传到/home/oracle/setup
    #解压gz，然后再解压cpio
    gunzip 10201_database_linux_x86_64.cpio.gz
    cpio -idmv < 10201_database_linux_x86_64.cpio

## 七. 配置vnc

### 7.1 安装VNC服务端

    yum install -y tigervnc-server

### 7.2 修改VNCServer的配置文件

在`/etc/sysconfig/vncservers`中添加以下内容：

    VNCSERVERS="1:oracle"
    VNCSERVERARGS[1]="-geometry 800x600"

表示Oracle用户的分辨率为800*600

### 7.3 切换到oracle用户，使用vncpasswd设置密码

### 7.4 执行vncserver，启动服务，并初始化配置

### 7.5 编辑配置文件~/.vnc/xstartup

其实没什么需要改的

## 八. 安装Oracle服务端
登录oracle用户，执行`/home/oracle/setup/database/runInstaller`，然后按照向导操作

## 九. 创建数据库
执行dbca命令，然后按照向导操作

## 十. 创建监听与服务
执行netca命令，然后按照向导操作

## 十一. 启动服务

###11.1 启动监听

    lsnrctl start 

### 11.2 启动服务

    sqlplus / as sysdba
    > startup;

### 11.3 停止服务

    > shutdown immediate;
