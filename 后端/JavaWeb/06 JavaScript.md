# 概述

**简介**
- JavaScript是一门跨平台、面向对象的脚本语言
- 控制网页行为，使网页可交互

**注意**
- JavaScript和Java使完全不同的语言，但是基础语法类似。
- JavaScript在1995年由Brendan Eich发明，并于1997年成为ECMA标准。
- ECMAScript6(ES6)是最新的JavaScript版本

# Javascript引入方式

1. 内部脚本：定义在HTML中
	- 使用`<script>`标签
		- 可以在任意地方定义任意数量的`<script>`标签
		- 一般放在`<body>`元素底部, 可以改善显示速度, 因为脚本执行会拖慢显示
2. 外部脚本：定义在js文件中，使用时引入
	- 定义一个外部的js文件，后缀为`.js`
	- 引入方式 `<script src="xxx"></script>`
		- `<script>`不能自闭合

# JavaScript基础语法

##  书写语法

1. 变量名区分大小写
2. 结尾分号可有可无
3. 注释
	- 单行注释: `//`
	- 多行注释: `/* 注释内容 */`
4. 大括号标识代码块

## 输出语句

- `windows.alert()` - 写入警告框
- `document.write()` - 写入HTML输出
- `console.log()` - 写入浏览器控制台

## 变量类型

- JavaScript使用`var`关键字声明变量
- JavaScript是一门弱类型语言, 变量可以存放不同类型的值
- 变量名需要遵守如下规则
	- 组成字符可以是任何字母、数组、下划线(`"_"`)或者美元符号(`"$"`)
	- 数字不能开头
	- 建议使用驼峰命名

- var:  
	1. 作用域 - 全局变量  
		- 代码块里面或者外面都是可以用的  
	2. 变量可以重复定义
- let(ES6新增):
	1. 作用域 - 代码块有效
	2. 不允许变量重复定义
- const(ES6新增):  
	1. 声明一个只读的常量, 一旦声明, 值无法被改变

## 数据类型

Javascript中分为原始类型和引用类型
- number - 整数、小鼠、NaN(Not a Number)
- string - 字符、字符串
- boolean
- null - 对象为空, 在类型上被认为是Object
- undefined - 声明的变量未初始化的默认值是undefined

**`typeof`运算符可以获取数据类型**
```js
var ch = 'a';  
var name = 'majinliang';  
var addr = "北京"  
document.write(typeof ch + "<br>");  
document.write(typeof name + "<br>");  
document.write(typeof addr + "<br>");
```

## 运算符

- 一元运算符 : ++, --
- 算数运算符
- 赋值运算符
- 关系运算符  - 注意 `==`与`===`区别
- 逻辑运算符
- 三目运算符

`==`:  
  - 先判断类型是否一样, 如果不一样, 则进行类型转换  
  - 类型一致后, 再进行值的比较  
`===`:全等于  
  - 判断类型是否一样, 不一样直接返回false  
  - 再比较值

**类型转换**

* 其他类型转为number  
	1. `string`: 按照字符串字面值转为数字. 如果字面值不是数字, 则转为`NaN`(不是数字的数字)  
		* 字符到数字转换: 字符串前加正号`+`  
		* 一般使用parseInt  
	2. `boolean`: true为1, false为0  
		* 好像不能用parseInt  
* 其他类型转为boolean  
	1. `number`: 0和NaN转为false, 其他数字转为true  
	2. `string`: 空字符串转为false, 其他转为true  
	3. `null`:false    
	4. `undefined`:false

## 流程控制语句

- `if`
- `switch`
- `for` - 循环关键字定义建议用`let`
- `while`
- `do...while`

## 函数

函数是被设计为执行特定任务的代码快

**定义: Javascript函数通过function关键词进行定义**
- 形参不需要类型
- 函数不需要定义返回值类型, 直接使用return即可

**定义方式1**
```js
function functionName(参数1, 参数2, ..){
	要执行的代码
}
```

**定义方式2**
```js
var functionName = function(参数列表){
	要执行的代码
}
```

- JavaScript中, 函数调用可以传递**任意个数参数**, 函数调用只和函数名称有关

# JavaScript对象

## Array
- Array对象用来定义数组
- JS数组类似于Java集合
	- 变长变类型

**定义**
- `var 变量名 = new Array(元素列表);`
- `var 变量名 = [元素列表];`

**访问**
`arr[索引] = 值;`
没定义的数组值默认为`undefined`

**属性**
- length: 数组的长度
**方法**
- `push`: 添加元素方法  
- `splice`: 删除元素方法
	- `start`: 开始位置
	- `count`: 删除数量

## String

**定义**
- `var 变量名 = new String(s)`
- `var 变量名 = s`

**属性**
- `length`
**方法**
- `charAt` - 获取索引位置字符
- `indexOf` - 检索字符串
- `trim` - 去除字符串两端空白字符

## 自定义对象

```js
var 对象名称 = {
	属性名称: 属性值,
	属性名称: 属性值,
	...
	函数名称: function(参数){
		语句
	}
}
```

## BOM
- Browser Object Model 浏览器对象模型
- JavaScript将浏览器的各个组成部分封装成为对象

**组成**
- `Window` : 窗口对象
- `Navigator` : 浏览器对象 - 基本用不到
- `Screen` : 屏幕对象 - 基本用不到
- `History` : 历史记录对象
- `Location` : 地址栏对象

### Window对象
获取: 直接使用 `window`, 其中 `window.` 可忽略

**属性 获取其他BOM对象**
- `history`
- `Navigator`
- `Screen`
- `location`

**方法**
- `alert` - 弹出警告框
- `confirm` - 弹出带有确认和取消按钮的提示框
	- 确定返回true, 取消返回false
- `setTimeout(function, 毫秒时)`: 
	- 一定时间间隔后执行一个function, 只执行一次  
- `setInterval(function, 毫秒时)`: 
	- 一定时间间隔后执行一个function, 循环执行

### History对象
获取: 使用`window.history`获取, `window.`可省略

**方法**
- back - 加载history列表的前一个url
- forward - 加载history列表的下一个url

### Location对象
获取: 使用`window.location`获取, `window.`可省略

**属性**
- href - 设置或返回完整的URL

## DOM
- Document Object Model 文档对象模型
- 将标记语言各个组成部分封装成为对象

**简介**
- DOM是W3C的标准
- DOM定义了访问HTML和XML文档的标准

- W3C DOM标准分为3个不同的部分
1. 核心DOM
	- Document: 文档对象
	- Element: 元素对象
	- Attribute: 属性对象
	- Text: 文本对象
	- Comment: 注释对象
2. XML DOM :  针对XML文档的标准模型
3. HTML DOM : 针对HTML文档的标准模型
	- Image: `<img>`
	- Button: `<input type='button'>`

**通过DOM, 可以对HTML进行操作**
- 改变HTML元素内容
- 改变HTML元素样式
- 对HTML DOM事件作出反应
- 添加和删除HTML元素

### 获取Element

**获取**
- `getElementById` : 通过id属性值获取, 返回一个Element对象
- `getElementsByTagName` : 通过标签名获取, 返回一个Element对象数组
- `getElementsByName` : 通过name属性值获取, 返回一个Element对象数组
- `getElementsByClassName` : 通过class属性值获取, 返回一个Element对象数组

### 常见HTML Element的使用
**一般查看文档使用**
https://www.w3school.com.cn/jsref/dom_obj_all.asp

- `<img>` 通过设置src改变显示的图片
- `element` - 所有Element对象都继承自element
	- style属性: 设置css样式
	- innerHTML: 设置元素内容
- `<input> checkbox` 
	- `checked`: true - 被选中; false - 不被选中

# 事件监听

**事件**
HTML事件是发生在HTML元素上的事情

**事件监听**
Javascript可以在事件被侦测到的时候**执行代码**

## 事件绑定

- 方式1: 通过HTML标签中的事件属性进行绑定
```Html
<input type="button" onclick="on()">
<script>
function on(){ alert("be clicked"); }
</script>
```

- 方式2: 通过DOM元素属性绑定
```html
<input type="button" id="btn">
<script>
document.getElementById("btn").onclick=function{ alert("be clicked"); }
</script>
```

## 常见事件
https://www.w3school.com.cn/jsref/dom_obj_event.asp

- onblur : 在元素失去焦点时发生
- onfocus : 元素获得焦点
- onchange : 域的内容发生改变
- onclick : 被点击
- onkeydown : 按键被按下
- onmouseover : 鼠标移到元素上
- onmouseout : 鼠标从元素移开
- onsubmit : 确认按钮被点击(表单中)
	- 该函数返回true表单提交, false不提交


# 表单验证

## 正则表达式

**概念**
定义了字符串组成的规则

**定义**
1. 直接量 - 不加引号
	- `var reg = /^\w{6,12}$/`
2. 创建RegExp对象

**方法**
`test(str)`
判断字符串是否符合规则, 返回true或false

**语法**
- `^` - 开始
- `$` - 结束
- `[]` - 某个范围内的单个字符
- `.` - 任意单个字符, 除了换行符和行结束符
- `\w` - 单词字符: 字母, 数字, 下划线, 相当于`[A-Za-z0-9_]`
- `\d` - 数字字符 , 相当于`[0-9]`
**量词**
- `+` - 至少一个
- `*` - 零个, 一个或多个
- `?` - 零个或一个
- `{x}` - x个
- `{m,}` - 至少m个
- `{m,n}` - 至少m个, 最多n个

**手机号正则(简单)**
`/^[1][3579][0-9]\d{8}$/`








