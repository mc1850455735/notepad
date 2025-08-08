# 私服简介

- 私服是一台独立的服务器, 用于解决团队内部资源共享与资源同步问题
- 相当于一台模拟的中央服务器

**私服下载与启用**
- Nexus是Sonatype公司的一款maven私服产品
	- 太高版本可能和教程中不同, 建议下载更低版本 (实测3.63与教程相似, 再低的版本官网中不提供)
- 在nexus的bin目录中, 存在文件`nexus.vmoptions`, 内含nexus的一些配置
	- 其中 `-Dkaraf.data=../sonatype-work/nexus3` 表示私服使用的库的位置
- bin目录下, 执行 `nexus.exe /run nexus` 启动私服, 其为一个jetty服务器
	- 启动较慢 (约2分钟), 不建议经常关闭重启私服
- 浏览器中, 输入 `localhost:8081` 访问私服, 进入后先登录
	- 安装目录下etc目录中`nexus-default.properties`文件保存有nexus基础配置信息, 如访问端口
	- `nexus.vmoptions`中存有nexus服务器启动对应的配置信息, 如默认占用的内存空间
	- 用户名为admin, 密码在 `sonatype-work/nexus3` 文件夹的 `admin.password` 中
	- 进入后, 设置新密码, 并允许匿名访问

**私服分为两种使用**
- 浏览使用
- 设置使用

# 私服仓库分类

### 操作流程
- 使用私服的过程, 实际上就是与私服中的仓库打交道的过程
- 私服中具有多个不同种类的仓库, 如
	- 存储和管理中央服务器中包的仓库
	- 存储私人开发的包的仓库
	- 存储各个包快照版本的仓库 等
- 私服中的各个仓库组合成为一个**仓库组**, 项目请求和下载依赖时, 向仓库组发送请求, 由私服进行从仓库组中的哪个仓库寻找对应的包的管理

### 操作分类
- 宿主仓库 (hosted) : 保存**自主研发**+**第三方**资源, 用于上传 
- 代理仓库 (proxy) : 代理连接**中央仓库**, 用于下载
- 仓库组 (group) : 为仓库编组, **简化**下载操作, 用于下载

对于多个不同的项目组, 每个项目组都会有一个自己的仓库组, 一个仓库组中具有若干个不同的宿主仓库, 所有的不同的项目组**公用**一个代理仓库 (避免多次下载)

# 资源上传与下载

- 与私服打交道的过程实际上是本地仓库与私服的互动过程, 相关的配置需要书写在本地仓库的配置中
- 访问私服需要提供
	- 上传的位置, 既宿主地址
	- 访问私服需要的用户名&密码
	- 下载所在的地址, 既仓库组的地址
- 配置本地仓库, 配置文件所在的位置是maven目录下的conf文件夹其中的setting.xml

### 私服仓库
- `<servers>`标签下的`<server>`配置了各种访问私服的账户以及要访问的仓库
	- `<id>`指的是私服中的仓库名称
	- `<username>`为用户名
	- `<password>`为用户名对应的密码
```xml
<servers>
	<server>
		<id>majinliang-snapshot</id>
		<username>admin</username>
		<password>123456</password>
	</server>
	<server>
		<id>majinliang-release</id>
		<username>admin</username>
		<password>123456</password>
	</server>
</servers>
```

- 在nexus的设置使用模式(Servevr administration and configuration)中, 在左侧Repository中, 可以创建一个仓库
	- 分别创建 snapshot 仓库和 release 仓库, 记得选择Version policy
	- 仓库类别选择 maven2(hosted) , 名称自选

### 私服镜像
- `<mirrors>`标签下的`<mirror>`用于配置执行的私服仓库组所在的访问路径, 需要将自己的仓库加入仓库组中
	- `<id>` 表示仓库组所在的id 
	- `<mirrorrOf>` : 表示谁使用来自于该镜像的包, 默认可以配置通配符 `*`
	- `<url>` : 配置私服仓库组所在的地址
	- 以maven-public为例, 使用时要将自己的仓库加入其中
```xml
<mirrors>
	<mirror>
		<id>maven-public</id>
		<mirrorOf>*</mirrorOf>
		<url>http://localhost:8081/repository/maven-public/</url>
	</mirror>
</mirrors>
```

### 项目发布

##### 位置配置
- 在工程中配置与私服中的哪个仓库打交道
- 使用`<distributionManagement>`标签管理项目的发布位置, 其中`<repository>`表示正式版发布的位置, `<snapshotRepository>`表示快照版发布位置
- `<id>`用来标识库的名称, `<url>`用来表示库的位置
```xml
<distributionManagement>  
    <repository>        
        <id>majinliang-release</id>  
        <url>http://localhost:8081/repository/majinliang-release/</url>  
    </repository>    
    <snapshotRepository>        
        <id>majinliang-snapshot</id>  
        <url>http://localhost:8081/repository/majinliang-snapshot/</url>  
    </snapshotRepository>
</distributionManagement>
```

##### 上传须知
- 对于maven项目, `install`指令是将工程打包后上传到本地, 而`deploy`将工程打包后上传到私服
	- 工程版本号如果为xxx-SNAPSHOT, 就会自动将当前项目上传到**快照版**对应的库中
- deploy发布时, 由于私服需要保证当前工程所用的所有资源私服上都存在, 所以需要花很多时间进行处理
- parent项目进行设置后, child项目可以选择继承parent, 也可以重新在自己的pom.xml中进行项目发布的设置
	- 既, 该设置要么继承, 要么自己设置, 但是不能没有

##### 私服镜像源设置
- 私服中的maven-center库连接的是远程仓库
- 可以在其中进行配置镜像源 => 在 Remote Storage 中设置镜像源地址
	- aliyun地址 : `https://maven.aliyun.com/repository/public`
