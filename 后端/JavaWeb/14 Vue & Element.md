
# Vue

## 概述

- Vue是一套前端框架, 免除原生JavaScript中的DOM操作, 简化书写
- 基于MVVM(Model-View-View-Model)思想, 实现数据的**双向绑定**, 将编程的关注点放在数据上
- 官网: https://cn.vuejs.org

**前代缺点**
MVC : 只能实现模型到视图的单项展示
![[后端/JavaWeb/Inbox/Pasted image 20240205015318.png]]

**本代优点**
MVVM : 可以实现数据的双向绑定
![[后端/JavaWeb/Inbox/Pasted image 20240205015310.png]]

## 快速入门
*注: 黑马中使用的是vue2*

1. 新建HTML, 引入Vue.js文件
- Vue3 : `<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>`
- Vue2 : `<script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>`

2. 在JS代码区域, 创建Vue核心对象, 进行数据绑定
```js
// 创建vue核心对象  
new Vue({  
    // 选择器选择要控制的代码块  
    el: "#app",  
    data(){  
        return {  
            username: ""  
        }  
    }  
});
```

3. 编写视图
```html
<div id="app">  
    <input v-model="username">  
    <!-- 插值表达式 -->  
    {{username}}  
</div>
```

## Vue常用指令

**指令**
- HTML标签上带有v-前缀的特殊属性, 不同指令就有不同的晗意.

- v-bind : 为html标签绑定属性值, 如设置href,css样式等
	- `<a v-bind:href="url">百度一下</a>`
	- `<a :href="url">百度一下</a>` : v-bind可以省略
- v-model : 在标案元素上创建双向数据绑定
	- `<input v-model="url">`
- v-on : 为html标签绑定事件
	- `<input type="button" value="a button" v-on:click="show()">`
	- `<input type="button" value="a button" @click="show()">`
		- 没有参数时括号可写可不写
	- vue代码展示
```js
// 创建vue核心对象  
new Vue({  
    // 选择器选择要控制的代码块  
    el: "#app",   
    methods:{  
        show(){  
            alert("我被点了")  
        }  
    }  
});
```

*v-if 直接不创建, v-show设置 display为 none*
- v-if : 条件性的渲染某元素, 判定为true时渲染,否则不渲染
- v-else
- v-else-if
	- `<div v-if="count==3">div1</div>`
	- `<div v-else-if="count==4">div2</div>`
	- `<div v-else>div3</div>`
- v-show : 根据条件展示某元素, 与if区别在于是切换的display属性的值
	- `<div v-show="count==3">dov-show</div>`

- v-for : 列表渲染, 遍历容器的元素或者对象的属性
	- 生成循环次数个标签
```js
<div v-for="addr in addrs">  
    {{addr}} <br>  
</div>
```

**带角标 - 第一个是内容, 第二个是角标**
```js
<div v-for="(addr,i) in addrs">  
    {{i}}--->{{addr}}  
</div>
```

## Vue生命周期

- 生命周期分为8个阶段 : 每触发一个生命周期事件, 就会自动执行一个生命周期方法(钩子函数)
- `beforeCreate` - 创建前
- `created` - 创建后
- `beforeMount` - 载入前
- `mounted` - 挂载完成
	- vue初始化成功, html页面渲染成功
	- 发送异步请求, 加载数据
- `beforeUpdate` - 更新前
- `updated` - 更新后
- `beforeDestroy` - 销毁前
- `destroyed` - 销毁后

![[后端/JavaWeb/Inbox/Pasted image 20240205113809.png]]

**示例**
```js
new Vue({
	el:"#app",
	mounted(){
		alert("vue挂载完毕, 发送异步请求");
	}
})
```

## Vue使用示例

**页面加载**
```js
new Vue({  
    el: "#app",  
    data() {  
        return {  
            brands: [],  
        }  
    },  
    mounted() {  
        // 提升this范围  
        var _this = this;  
        // 页面加载完成后发送异步请求查询数据  
        axios({  
            method: "get",  
            url: "http://localhost:8080/brand-demo/selectAllServlet",  
        }).then(function (resp) {  
            _this.brands = resp.data;  
        })  
    },  
    method: {  
        toAdd() {  
            location.href = "http://localhost:8080/brand-demo/add.html"  
        }  
    }  
})
```

**更新数据**
```js
new Vue({  
    el: "#app",  
    data() {  
        return {  
            brand: {},  
        }  
    },  
    methods: {  
        submitForm(){  
            _this = this;  
            axios({  
                method: "post",  
                url: "",  
                data: _this.brand,  
            }).then(function (resp){  
                if(resp.data == "success"){  
                    location.href = "http://localhost:8080/brand-demo/brand.html"  
                }  
            })  
        }  
    }  
})
```


# Element

- Element : 饿了么公司前端开发团队提供的一套基于Vue的网站组件库, 用于快速构建网页
- 组件 : 组成网页的部件 , 如 超链接, 按钮, 图片, 表格 等等

[组件 | Element](https://element.eleme.cn/#/zh-CN/component/layout)

## 快速入门

1. 引入Element的css, js文件 和 Vue.js
```html
<script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>
<!-- 引入样式 --> 
<link rel="stylesheet" href="https://unpkg.com/element-ui/lib/theme-chalk/index.css"> 
<!-- 引入组件库 --> 
<script src="https://unpkg.com/element-ui/lib/index.js"></script>
```
2. 创建Vue核心对象
```html
<script>
new Vue({  
    el: "#app",  
})
</script>
```
3. 官网复制Element组件代码

## Element布局

- Layout布局 : 通过基础的24分栏, 迅速简便的创建布局
- Container布局容器 : 用于布局的容器组件, 方便快速搭建页面的基本结构

**自行搜索代码复制**

## Element组件

**以下全部复制即可**

### 表格

### 表单

#### 对话框+表单

### 分页


