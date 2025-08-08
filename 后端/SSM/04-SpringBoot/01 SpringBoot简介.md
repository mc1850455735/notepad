
# SpringBoot项目构建和使用

### 项目构建

1. 创建新项目, 选择Spring初始化(Spring Initializr) , 并配置模块相关基础信息
	 - SpringBoot工程的创建必须联网, 因为其是由Spring Initializr初始化完成的
	 - 可以在 start.spring.io 中手工创建对应项目
2. 填写对应的名称和路径, 设置Java版本
3. 选择该模块需要的依赖, 创建项目
4. 书写需要的代码
5. 直接运行Application类, 启动Tomcat服务器

### SpringBoot关键配置

##### 关键配置
- SpringBoot项目的pom.xml继承自boot父工程, 具体配置如下
```xml
<parent>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-parent</artifactId>  
    <version>3.5.4</version>  
    <relativePath/>
</parent>
```
- SpringBoot工程含有两个依赖, 分别为 `spring-boot-starter-web` 和 `spring-boot-starter-test`
	- `spring-boot-starter-web` : 用于实现web功能
	- `spring-boot-starter-test` : 用于对工程进行测试
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

##### 最简文件
- pom.xml
- Application类
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.5.4</version>
    </parent>
    <groupId>top.majinliang</groupId>
    <artifactId>springboot_01_quickstart</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
</project>
```


### 与Spring对比

| 类/配置文件            | Spring   | SpringBoot |
| ---------------------- | -------- | ---------- |
| pom文件中的坐标        | 手工添加 | 勾选添加   |
| Servlet web3.0配置类   | 手工制作 | 无         |
| Spring/SpringMVC配置类 | 手工制作 | 无         |
| 控制器Controller       | 手工制作 | 手工制作   |

### SpringBoot快速启动

**前后端分离合作开发**
- boot程序可以不依赖tomcat和idea, 直接运行 (程序也需要连接对应数据库)
	- 对项目进行package命令, 对应的target会生成一个jar文件(在没有特殊说明情况下)
	- 在该jar文件所在目录打开cmd, 使用指令`java -jar {jar包名称}`即可运行后台程序
- 任何maven工程都可以生成jar包, 但是只有boot程序可以直接使用`java -jar`运行
- jar包支持命令行启动需要依赖如下maven插件

```xml
<build>
	  <plugins>
    	  <plugin>
        	  <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

**为什么该插件可以使jar直接运行**
- 该插件不仅将该工程本身打包成为一个jar包, 而且将该工程依赖的jar包也都打包到了其中, 并且设置了运行该jar包时通过哪一个类进行

# SpringBoot简介

### 简介
- SpringBoot是由Privotal团队提供的全新框架, 目的是为了**简化**Spring应用的初试搭建以及开发过程

- **Spring缺点**
	- 配置繁琐
	- 依赖设置繁琐
- **SpringBoot程序优点**
	- 自动配置
	- **起步依赖**, 简化依赖配置
	- 辅助功能 (内置服务器等)

### 起步依赖
- starter
	- SpringBoot中常见项目名称, 定义了当前项目使用的所有项目坐标, 以达到**减少依赖配置**的目的
	- 这种依赖可以被统称为**起步依赖(starter)** , 想要使用boot集成的哪种技术, 就加上其对应的起步依赖
- parent
	- 所有SpringBoot项目要继承的项目, 定义了若干个坐标版本号, 以达到**减少依赖冲突**的目的
	- 不同版本的springboot其坐标版本也不尽相同
- 实际开发
	- 实际开发使用任意坐标时, 仅书写GAV中的G和A, 而 V(Version) 由SpringBoot提供
	- 如发生坐标错误以后, 再指定版本

### 辅助功能

**辅助功能**
- 对于Springboot内置tomcat等用于项目构建和启动的功能, 就叫做springboot的辅助功能

**程序启动**
- 启动方式
```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```
- Springboot创建项目时, 采用jar的打包方式
- SpringBoot的引导类是项目的入口, 运行main方法即可启动项目

**更换jetty服务器**
- 在 ` spring-boot-starter-web ` 中exclude tomcat服务器相关依赖
- 添加jetty服务器相关依赖
	- jetty比tomcat更**轻量级**, 可扩展性更强, 谷歌应用引擎目前已经全面切换为使用jetty
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```
