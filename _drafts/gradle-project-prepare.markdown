---
layout: post
title:  "创建一个Gradle项目后要做的事情"
date:   2016-02-26 15:52:00 +0800
categories: java
tags: Java
---

## 1. 创建初始目录

IDEA中有个选项，可以创建src的目录结构：  


当然也可以手动敲命令创建：

Windows：

    md src\main\java src\main\resources src\test\java src\test\resources

Linux:

    mkdir -p src/main/java src/main/resources src/test/java src/test/resources

## 2. 修改编译版本和编码
自动生成的build.gradle的Java编译版本是1.5，文件编码是GBK。  
如果要修改成其他的，在build.gradle中添加：

    sourceCompatibility = 1.7
    [compileJava, compileTestJava]*.options*.encoding = 'UTF-8'

## 3. 添加Maven仓库
为了加快编译时下载包的速度，应该添加maven镜像，例如oschina的maven镜像。  
在build.gradle中找到repositories，替换：  

    repositories {
        maven{ url 'http://maven.oschina.net/content/groups/public/'}
        mavenCentral()
    }
