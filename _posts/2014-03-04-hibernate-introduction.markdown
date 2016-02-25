---
layout: post
title:  "Hibernate入门笔记"
date:   2014-03-04 11:45:00 +0800
categories: java
tags: Java
---

### 一. 什么是持久化

#### 1.1 瞬时状态(Transient)的定义
数据保存在内存中，程序退出后会消失。

#### 1.2 持久状态(Persistent)的定义
数据保存在磁盘中，程序退出后仍然存在。

#### 1.3 持久化
持久化就是将数据从瞬时状态转换为持久状态。
 
### 二. JDBC与Hibernate

#### 共同点：
√ 都是数据库操作中间件  
√ 非线程安全，要及时释放资源  

#### 不同点：
× JDBC直接使用SQL，Hibernate使用自己的HQL  
× JDBC操作的是数据，Hibernate操作的是持久化对象(简称PO)  
 
### 三. ORM是什么

ORM全称Object/Relation Mapping，也就是“对象/关系数据库映射”。现在流行的编程语言是面向对象的，而现在主流的数据库是关系数据库。ORM框架可以理解成面向对象的程序与关系型数据库之间沟通的桥梁。
 
### 四. Hibernate的持久化对象

Hibernate是低侵入式设计，也就是说Hibernate所操作的对象不需要继承于Hibernate中的哪一个类，也不需要实现哪一个特有的接口，完全可以是一个POJO（不过也有建议实现的方法，基本上也是Object中已有的）。

只是一个POJO，Hibernate当然无法通过它和数据库关联起来，因此还需要一个映射文件(*.hbm.xml)，将对象和表、属性和字段映射起来，因此，一个PO相当于POJO+映射xml。
 
### 五. 创建基于Hibernate的项目的步骤

5.1 添加Hibernate的类包  
5.2 添加Hibernate的配置文件(hibernate.cfg.xml)  
5.3 根据数据库结构创建各个PO的类  
5.4 添加每个PO的映射文件(*.hbm.xml)  
 
### 六. Hibernate对数据进行CURD的流程

6.1 创建Configuration  
6.2 创建SessionFactory  
6.3 打开Session(类似于JDBC中的Connection)  
6.4 开始事务(Transaction)  
6.5 进行持久化操作(CURD)  
6.6 提交事务  
6.7 关闭Session  
 
### 七. 一个简单的例子

以下是一个简单的插入一条数据的例子：

    //假设已经存在User.hbm.xml并配置好了映射关系
    public static void main(String[] args) throws Exception{
        Configuration conf = new Configuration().configure();
        SessionFactory sf = conf.buildSessionFactory();
        Session sess = sf.openSession();
        Transaction tx = sess.beginTransaction();
        User u = new User();
        u.setId(1);
        u.setName("Arathi");
        sess.save(u);
        tx.commit();
        sess.close();
        sf.close();
    }

#### 注意：  
Configuration在创建时默认载入的是hibernate.cfg.xml这个文件，也可以指定其他路径，甚至可以在程序中硬编码配置信息。  
SessionFactory开销很大，所以最好只创建一个，不用了以后也要及时释放。  
Session的save方法是插入(INSERT)，update方法是更新(UPDATE)，delete方法是删除(DELETE)，get方法是查询(SELECT)。还有一些别的方法。
