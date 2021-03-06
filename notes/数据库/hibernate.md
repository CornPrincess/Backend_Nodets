# hibernate 简介

首先要明白我们为什么需要使用 hibernate 或者 mybatis 等 ORM 框架。其原因就是Java 是面向对象的语言，关系型数据库是关系模型，这两者之间**模型不匹配（阻抗不匹配）**。

解决的办法有两种：

1.使用JDBC手工转换

2.使用ORM框架来解决，ORM的思想为：将关系数据库中表中的记录映射成为对象，以对象的形式展现。程序员可以把对数据库的操作转化为对对象的操作。目前主流的框架为 

- Hibernate：
  - 完成对象的持久化操作
  - Hibernate 允许开发者采用面向对象的方式来操作关系数据库
  - 消除那些针对特定数据库厂商的 SQL 代码
- Mybatis：
  - 相比 Hibernate 灵活，运行速度快
  - 开发速度慢，不支持村岁的面向对象操作，需熟悉 sql 语句，并且熟练使用sql 语句优化功能

## hello world

首先 hibernate 开发大致分为四步：

![hibernate steps](https://blog-1300663127.cos.ap-shanghai.myqcloud.com/Hibernate/hibernate-steps.png)

开始
