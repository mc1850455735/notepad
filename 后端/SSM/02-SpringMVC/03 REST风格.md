# REST简介

- REST (Representational State Transfer) , 表现形式状态转换
	- 可以理解为**访问网络资源的格式**
	- 传统风格
		- `http://localhost/user/getById?id=1`
		- `http://localhost/user/saveUser`
	- REST风格描述
		- `http://localhost/user/1`
		- `http://localhost/user`
- 优点
	- 隐藏资源访问行为, 无法根据地址得知对资源是何种操作
	- 书写简化
- 按照REST风格访问资源时, 使用**行为动作区分**对资源进行了何种操作
	- 使用如`GET`, `POST`, `PUT`, `DELETE`等不同的动作
- 所谓的风格指的是一种约定形式而不是一种规范, 不遵循REST风格开发也是允许且可行的, 但是建议使用更加规范的REST风格进行开发
- 根据REST风格对资源进行访问被称为 **`RESTful`**

**常用的4种method**
- GET - 查询
- POST - 新建/保存
- PUT - 修改/更新
- DELETE - 删除

# RESTful入门案例

- `@RequestMapping` 中, 可以通过 method 设定该路径对应的行为
```java
@RequestMapping(value = "/users", method = RequestMethod.POST)  
@ResponseBody  
public String save() {  
    System.out.println("user save ...");  
    return "{'module':'user save'}";  
}
```
- 如果想要请求参数来自于路径而不是使用问号形式传递, 那么应该在对应的变量前添加注解 `@PathVariable` (参数名和路径中的名称要对应)
- 同时, 在路径配置中说明参数来自于路径的哪一个位置
```java
@RequestMapping(value = "/users/{id}", method = RequestMethod.DELETE)  
@ResponseBody  
public String delete(@PathVariable Integer id) {  
    System.out.println("user delete ... " + id);  
    return "{'module':'user delete'}";  
}
```
- 如果是来自Body的参数, 正常书写 `@RequestBody` 即可
```java
@RequestMapping(value = "/users", method = RequestMethod.PUT)  
@ResponseBody  
public String update(@RequestBody User user) {  
    ...
}
```

**建议**
- 对于已经学的三种参数描述, 其区分如下
	- `@RequestParam`用于接收url地址传参或者表单传参
	- `@RequestBody`用于接收json数据
	- `@PathVariable`用于接收路径参数, 使用 `{参数名}` 描述路径参数
- 其应用如下所示
	- 后期开发中, 发送请求参数超过一个时, 以json格式为主, `@RequestBody`应用较广. 会将大量数据封装为pojo以json串的形式传输
	- 如果发送非json格式数据, 选用`@RequestParam`接收请求参数 (用量较少)
	- 采用RESTful进行开发时, 如果参数数量较少, 则使用`@PathVariable`接收请求路径变量, 通常用于传递id值等. 对于多个参数的情况, 仍建议使用json

# REST快速开发

- 为简化开发, 可以将路径中重复的部分提取到外部, 以下是两个重复较多的部分

### 简化思路

**每个方法路径中重复的前缀, 如book**
- 直接在类上添加`@RequestMapping`注解, 将重复的部分添加到类的路径中
```java
@Controller  
@RequestMapping("/books")  
public class BookController {
		...
}
```

**每个方法上重复的@ResponseBody注解**
- 直接在类上添加`@ResponseBody`注解, 表示该类中所有的方法都带该注解
```java
@Controller  
@ResponseBody  
@RequestMapping("/books")  
public class BookController {
		...
}
```

### 注解简化

- 因为`@Controller`和`@ResponseBody`大多时间在一起, 所以可以合并成为一个新注解`@RestController`
```java
@RestController  
@RequestMapping("/books")  
public class BookController {
		...
}
```

- 对于 `@RequestMapping(method = "xxx")` , 其书写过于繁琐且固定, 所以使用一套相似的注解同时表示方法和路径
	- 如PostMapping , GetMapping, PutMapping, DeleteMapping等
```java
@GetMapping("/{id}")  
public String getById(@PathVariable Integer id) {  
    ...
}
```

# 案例 : 基于RESTful页面数据交互

### 放行静态资源

- 如果直接访问 `Resources` 内的资源, 由于SpringMvc的拦截, SpringMvc检测到没有对其设置对应的Get, 会导致浏览器无法获取`Resources`内的页面
- 为了不让SpringMvc拿走请求静态资源的访问, 应设法不让SpringMvc拦截试图访问静态资源的请求
- 新建名为SpringMvcSupport的类, 继承 `WebMvcConfigurationSupport`, 用来设置资源管理接口, 使访问某个url时, 指定访问某个路径下的静态资源
- 设置完成后, 将该配置加入`SpringMvcConfig`
```java
@Configuration
public class SpringMvcSupport extends WebMvcConfigurationSupport {  
    @Override  
    protected void addResourceHandlers(ResourceHandlerRegistry registry) {  
        // 设置当访问 pages/xxx 的时候, 走/pages目录下的内容
        registry.addResourceHandler("/pages/**")
				        .addResourceLocations("/pages/");  
    }  
}
```


