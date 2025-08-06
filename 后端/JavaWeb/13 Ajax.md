# Ajax
- 概念 : AJAX(Asynchronous JavaScript And XML) - 异步的JavaScript和XML

## 作用

1. 与服务器进行数据交换 : 通过AJAX可以给服务器发送请求, 并获取服务器响应的数据
	- 使用AJAX和服务器进行通信, 就可以使用HTML+AJAX来替换JSP页面
![[后端/JavaWeb/Inbox/Pasted image 20240204173845.png]]
2. 异步交互 : 可以在不重新加载整个页面的情况下与服务器交换数据并更新部分网页的技术 (局部更新)
	- 搜索联想 , 用户名是否可以的校验, ... 

**同步和异步**

同步 : 客户端请求服务器后, 在服务器响应前, 客户端无法进行其他操作
异步 : 客户端请求服务器后, 客户端可以在服务器处理过程中进行其他操作

使用form表单传递的是同步的请求, 所以form表单的action应该置空且提交按钮类型设置为button

## 快速入门


1. 编写AjaxServlet, 使用response输出字符串
2. 创建XMLHttpRequest对象 : 用于和服务器交换数据
3. 向服务器发送请求 - 异步交互请求路径写**全路径**(为前后端分离做准备)
4. 获取服务器响应数据(一般只有这个地方需要修改)

```js
xhttp = new XMLHttpRequest();
xhttp.open("GET", "http://localhost:8080/ajax-demo/ajaxServlet");  
xhttp.send();
xhttp.onreadystatechange = function() {
    if (this.readyState == 4 && this.status == 200) {
	    // 根据response的相关文本进行判断和处理
        if (this.responseText == "true") {} 
        else {}
    }
};
```

[AJAX 简介 (w3school.com.cn)](https://www.w3school.com.cn/js/js_ajax_intro.asp)

ajax的请求type为xhr, 即异步请求

# Axios

## 使用

- 对原装的Ajax代码进行封装, 简化书写
[起步 | Axios Docs (axios-http.com)](https://axios-http.com/zh/docs/intro)

1. 引入axios的js文件
	- `<script src="https://unpkg.com/axios/dist/axios.min.js"></script>`
2. 使用axios发送请求, 并获取响应结果

- `resp.data` - 返回的是数据原本的类型
- 返回JSON时可能查找不到其中的值, 保证名称正确即可

**get**
```js
axios({  
    method: "get",  
    url: "http://localhost:8080/ajax-demo/axiosServlet?username=zhangsan"  
}).then(function (resp){     // 回调函数
    alert(resp.data);  
})
```
**post**
```js
axios({  
    method: "post",  
    url: "http://localhost:8080/ajax-demo/axiosServlet",  
    data: "username=zhangsan"  
}).then(function (resp){  
    alert(resp.data);  
})
```

## 请求方式别名

axios给其中请求方式封装了对应的别名, 简化了开发

**get & post**
```js
axios.get("http://localhost:8080/ajax-demo/axiosServlet?username=zhangsan").then(function (resp){  
    alert(resp.data);  
})  

axios.post("http://localhost:8080/ajax-demo/axiosServlet?username=zhangsan","username=zhangsan").then(function (resp){  
    alert(resp.data);  
})
```

# JSON

- 概念 : JavaScript Object Notation. JavaScript对象表示法
- 由于其语法简单, 层次结构鲜明, 现多用于作为**数据载体**, 在网络中进行数据传输

```json
{
	"name": "zhangsan",
	"age": 23,
	"city": "北京"
}
```

## 基础语法

**定义**
```js
var 变量名={
	"key1": value1,
	"key2": value2,
	"key3": [values1,values2,values3]
	...
};
```
最后一个位置没有逗号

**获取数据**
`变量名.key`

**数据类型**
- 数字(整数 或 浮点数)
- 字符串( 双引号中 )
- 逻辑值( true或false )
- 数组( 方括号中 )
- 对象( 在花括号中 )
- null

**JSON避免中文乱码**
*注意 text后跟json*
`response.setContentType("text/json;charset=utf-8");`

## JSON与Java对象转换

- 请求数据 : JSON字符串转为Java对象
- 响应数据 : Java对象转为JSON字符串

### post请求读取请求体
- post请求使用`request.getParameter`无法接收数据
- 使用获取请求体的方式`request.getReader()`获取json字符串, 再将转换成为对象
- 如果只有一个对应的属性, 那么该对象的其他属性会置null


### Fastjson
- Fastjson是阿里巴巴提供的一个Java语言编写的高性能功能完善的Json库, 是目前Java语言中最快的JSON库, 可以实现Java对象与JSON的相互转换

**使用**
1. 导入坐标
```xml
<dependency>  
    <groupId>com.alibaba</groupId>  
    <artifactId>fastjson</artifactId>  
    <version>1.2.62</version>  
</dependency>
```
2. Java对象转Json
`String userJson = JSON.toJSONString(user);`
3. Json转Java对象
`JSON.parseObject(jsonString, User.class);`

**如果在pojo类中添加了额外的get方法, 应该在toJSONString方法中添加SerializerFeature.IgnoreErrorGetter参数**
`String jsonString = JSON.toJSONString(brands, SerializerFeature.IgnoreErrorGetter);`

### Jaskson



# 示例

```js
// 一个使用axios批量创建表格的示例
axios({  
    method: "get",  
    url: "/brand-demo/selectAllServlet"  
}).then(function (resp) {  
    let tableData = "<tr>\n" +  
        "        <td>id</td>\n" +  
        "        <td>brandName</td>\n" +  
        "        <td>companyName</td>\n" +  
        "        <td>ordered</td>\n" +  
        "        <td>description</td>\n" +  
        "        <td>status</td>\n" +  
        "        <td>operation</td>\n" +  
        "    </tr>\n";  
    // resp.data获取数据  
    let brands = resp.data;  
  
    for (let i = 0; i < brands.length; i++) {  
        let brand = brands[i];  
        tableData +=  
            "<tr>\n" +  
            "    <td>" + (i + 1) + "</td>\n" +  
            "    <td>" + brand.brandName + "</td>\n" +  
            "    <td>" + brand.companyName + "</td>\n" +  
            "    <td>" + brand.ordered + "</td>\n" +  
            "    <td>" + brand.description + "</td>\n" +  
            "    <td>" + brand.status + "</td>\n" +  
            "    <td> " +  
            "       <a href='/brand-demo/add.html'>添加</a> " +  
            "       <a href='/brand-demo/updateServlet'>修改</a> " +  
            "    </td>\n" +  
            "</tr>\n";  
    }  
  
    document.getElementById("brandTable").innerHTML = tableData;  
})
```













