# 概述

用于管理和构建java项目的工具

**主要功能**
- 提供了标准化项目结构
	- 可以忽略各个ide的区别
- 提供了标准化构建流程
	- 编译
	- 测试
	- 打包
	- 发布
- 提供了一套依赖管理机制
	- 管理项目所依赖的第三方资源
		- 下载jar包
		- 复制jar包到项目
		- 将jar包加入工作环境
	- Maven使用标准的坐标配置来管理各种依赖, 只需要简单的配置就可以完成依赖管理

**Maven结构**
- 项目名
	- src - 代码
		- main - 源代码
			- java - java源代码
			- resources - 源配置文件
			- webapp - web项目核心目录
		- test - 测试代码
			- java - java测试代码
			- resources - 测试配置文件
	- pom.xml - maven配置文件

# 简介

apache maven - 项目管理和构建工具, 基于POM概念

- 项目对象模型
- 依赖管理模型
- 插件

![[后端/JavaWeb/Inbox/Pasted image 20240128232204.png]]

- 本地仓库 - 自己计算机上的目录
- 中央仓库 - Maven团队维护的全球唯一性目录
- 远程仓库 - 公司团队搭建的私有仓库

本地仓库没有时, 会从中央仓库下载, 并保存到本地仓库
如果架设了私服, 会先找私服, 再找中央仓库, 同时私服也会同步

下载好Maven需要配置系统变量, 而且需要配置本地仓库, 同时最好使用镜像网站

# Maven基本使用

## 常用命令

- compile - 编译
- clean - 清理
- test - 测试
- package - 打包
- install - 安装 当前项目安装到本地maven仓库

target中存放着编译后的文件

## 生命周期

**三套生命周期**
- clean - 清理
	- pre-clean -> clean ->post-clean
- default - 核心工作
	- compile -> test -> package -> install
- site - 产生报告,发布站点
	- pre-site -> site -> post-site

同一生命周期内, 后面的命令执行, 前面的所有命令也会自动执行

## Maven坐标

**什么是坐标**
- 坐标是资源的唯一标识
- 使用坐标来定义项目或者引入项目中需要的依赖

**Maven坐标组成**
- groupId - 定义当前的Maven项目隶属组织名称
- artifactId - 定义当前Maven项目名称
- version - 定义当前的项目版本号

## 依赖管理

### 依赖下载

```xml
<!-- 当前项目的坐标 -->  
<groupId>top.majinliang</groupId>  
<artifactId>maven-demo</artifactId>  
<version>1.0-SNAPSHOT</version>  
<packaging>jar</packaging>
```

```xml
<!-- 依赖项 -->  
<dependencies>  
  <!-- junit依赖 -->  
  <dependency>  
    <groupId>junit</groupId>  
    <artifactId>junit</artifactId>  
    <version>3.8.1</version>  
    <scope>test</scope>  
  </dependency>  
  <!-- mysql依赖 -->  
  <dependency>  
    <groupId>com.mysql</groupId>  
    <artifactId>mysql-connector-j</artifactId>  
    <version>8.2.0</version>  
  </dependency>   
</dependencies>
```

在`xml`页面中`Alt` + `Insert`可以快速插入依赖

### 依赖范围

```xml
<!-- 测试环境下有效 -->
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<version>8.0.12</version>
	<scope>test</scope>
</dependency>
```

- 编译环境
	- java目录下可用jar包
- 测试环境
	- test
- 运行环境
	- 导入jar包后运行有效

`<scope>`默认值是`compile`

|依赖范围|编译|测试|运行|例子|
|-|-|-|-|-|
|compile|Y|Y|Y|logback|
|test|-|Y|-|junit|
|provided|Y|Y|-|servlet-api|
|runtime|-|Y|Y|jdbc驱动|
|system|Y|Y|-|存储在本地jar包|
|import||||引入DependencyManagement|
