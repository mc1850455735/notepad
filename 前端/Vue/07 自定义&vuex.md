### 自定义创建项目

- 目标: 基于VueCli自定义创建项目架子
- 步骤: 安装脚手架->创建项目->选择自定义(Manually select features) 
	- 空格选中, 回车确定

- 需要加入的有 => 
	- `Babel`(语法降级)
	- `Router`(路由)
	- `CSS Pre-processors`(CSS预处理器)
	- `Linter / Formatter`(代码规范)
- 版本选择2.x，路由模式写n (no), 表示使用hash模式
- CSS预处理器选择`less`(因为我只会less)
- ESLint规范选择 -> `ESLint + Standard config` (无分号规范, 标准化, 目前最流行)
- 校验时机 => `Lint on save` 保存时进行校验
- 最后选择`In dedicated config files`, 将配置文件放在单独的文件中

### ESlint代码规范

**认识代码规范**
- 代码规范: 一套写代码时约定的规则, 如赋值符号左右是否加空格
- JavaScript Standard Style规范说明: [https://standardjs.com/rules-zhcn.html]
- 当代码不符合规范时, 会报错并由ESlint提醒

**解决规范错误**
- 手动修正
	- 根据错误提示一项项手动修改
	- 不认识语法报错含义的, 可以去ESlint规则表中查看
	- 应该具有手动修正错误的能力
- 自动修正
	- VSCode插件ESLint高亮错误, 并通过配置 **自动** 帮助修复错误

### vuex

#### 基本概念
- 是什么: vuex是vue的一个状态管理工具, 状态就是数据, 用于管理vue通用的数据
	- 比如多组件共享的数据
- 场景: 
	- 某个状态在很多个组件来使用 (比如个人信息)
	- 多个组件共同维护一份数据(比如购物车)
- 优势
	- 共同维护同一份数据, 数据集中化管理
	- 响应式变化
	- 操作简洁易懂
	- 避免了重复大量的父传子/子传父操作, 提高了系统效率

#### 基本使用

- 目标: 安装vuex插件, 初始化一个空仓库
- 步骤:
	- 安装vuex => 安装`vuex@3`
	- 新建vuex模板文件 => 在`store/index.js`中专门存放vuex
	- 创建仓库 => 1. `Vue.use(Vuex)` 2. `new Vuex.store()`
	- main.js导入挂载
- 通过`this.$store`获取仓库

```js
// 仓库文件创建
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)
// 创建空仓库
const store = new Vuex.Store()

export default store

// 仓库的挂载
import store from '@store/index.js'
new Vue({
  render: h => h(App),
  store
}).$mount('#app')
```

#### 核心概念

##### state状态

- 如何提供给仓库数据
	- 使用state: state提供唯一的公共数据源
	- 所有共享的数据要统一放到Store中的State中存储
	- state对象中可以添加我们想要共享的数据

- 如何使用仓库中数据
	- 通过store直接访问
		- this.$store
		- import导入store
	- 通过辅助函数(简化)

**state的两种提供方法**
- 使用函数方式 (推荐)
```js
export default {
	state() {
		return {
			数据
		}
	}
}
```
- 使用对象方式
```js
export default {
	state: {
		数据
	}
}
```

**通过store直接访问**
- 模板中: `{{ $store.state.xxx }}`
	- `button`的`@click`中直接试图访问其中数据也不需要加`this`
- 组件逻辑中: `this.$store.state.xxx`
- JS模块中: `store.state.xxx`

**通过辅助函数简化**
- 通过计算属性, 可以将原本复杂的`$store.state.xxx`转变为简单的xxx进行表示, 比如`computed: { count: this.$store.state.count }`
- `mapState`是一个辅助函数, 帮助把store中的数据**自动**映射到组件的计算属性当中
	- 步骤
		- 导入mapState => `import {mapState} from 'vuex'`
		- **数组方式**引入state => `mapState(['count'])`, 数组中填写`store`中的属性名
		- 展开运算符映射 => `computed: {...mapState(['count']), 其他计算属性}`

```js
import { mapState } from 'vuex'
computed: {
	...mapState(['title', 'count'])
}
```

##### mutations

**概念**
- 目标: vuex同样遵循单向数据流, 组件中不能直接修改仓库中的数据
	- 默认情况下不是**严格模式**, 所以可以直接修改, 但是这种方式不遵循规范, 不建议这样使用
	- 可以在仓库的设置中使用`strict: true`开启严格模式 - 有利于初学者检测不规范代码, 上线时需要关闭
	- 应该由组件向仓库发起申请, 再由仓库对数据实现修改
- 应该通过`mutation`进行修改数据, 所有的修改都是在仓库中进行的

**流程**
1. 定义mutations对象, 对象中存放修改state的方法
	- 所有mutation, 第一个参数都是state
2. 组件中提交调用mutations -> `this.$store.commit('方法名')`
```js
// 在store对象的声明中进行mutations的创建
const store = new Vuex.Store({
  // 开启严格模式
  strict: true,
  state: {
    count: 100,
    title: 'nihao'
  },
  // mutations进行仓库中的修改
  mutations: {
    addOne (state) {
      state.count += 1
    },
  }
})
// 在组件中调用
handleAddOne () {
    this.$store.commit('addOne')
},
```

**传参**
- 语法: 
	- `this.$store.commit('函数名', 参数)`
	- 有且只能有**一个**参数
	- `函数(state, 参数名){函数体}`
- 步骤: 
	1. 提供带参数的mutations函数(带参数 -> 提交载荷payload)
	2. 在页面中调用时使用对应的mutation -> `this.$store.commit('函数名', 参数)`
- 代码
```js
// mutations中的带提交载荷的函数
changeTitle (state, e) {
    state.title = e
}
// 参数的传递过程
handleChangeTitle (e) {
    this.$store.commit('changeTitle', e.target.value)
}
```

**mapMutations**
- 将mutations中的方法提取出来, 映射到组件methods中 => 相当于组件多了一个对应的方法, 可以进行直接使用
```js
import {mapMutations} from 'vuex'
export default {
	methods: {
		...mapMutations(['subCount']), // 方法名映射
		其他方法,
	}
}
```


##### actions
- 用于进行**异步**操作的处理
	- **mutations**必须是**同步**的(便于监测数据变化, 记录调试)
	- 大多数时候用来进行发请求
- 步骤
	1. 提供action方法, 其参数为context
	2. 页面中dispatch调用
		- `$store.dispatch('actions方法名')`
- 注意
	- actions中不能直接操作state, 还是需要commit
	- 只是异步相关的部分由actions进行处理 
	- actions中的参数为context(上下文), 在分模块的时候有用
		- 未分模块时, 直接当成store仓库
```js
// 在store中的声明
actions: {
	aysncChangeTitle (context) {
		setTimeout(() => {
			// 模拟异步
			// 同样需要调用mutations中的方法
			context.commit('changeTitle', 'sss')
		}, 1000)
	}
}
// 在组件中的调用
<button @click="$store.dispatch('aysncChangeTitle')">1秒后修改标题</button>
// 在对应函数中的调用应该加上this, 而在html中直接调用的不能加this
aysncChange () {
	this.$store.dispatch('aysncChangeTitle')
}
```

**辅助函数**
- mapActions用于将actions中的方法映射到组件的methods中
- 类似于mapState和mapMutations
	- 不同的是, state的映射必须映射在computed中, 而mutations和actions映射在methods中
```js
import { mapActions } from 'vuex'
export default {
	methods: {
		...mapActions(['actions方法名']), // 方法名映射
		其他方法,
	}
}
```


##### getters
- 类似于计算属性, 利用state中的数据派生出一些状态
	- getters函数的第一个参数是state
	- getters函数必须有返回值

1. 定义getters
```js
getters: {
	filterList(state) {
		return state.相应数据;
	}
}
```
2. 使用getters
	1. 通过store访问getters
	2. 通过辅助函数mapGetters映射
```js
// 1.通过store访问
{{$store.getters.filterList}}
// 2.通过辅助函数映射
computed: {
	...mapGetters(['filterList'])
}
```

#### 模块module
- 问题: 如果vuex使用单一状态树, 应用的所有状态会集中到一个对象, 当应用变得越来越大时, store对象会变得难以维护

**基础概念**
- 模块拆分: 目录`store/modules/xxx.js`
	- 每个js文件就是一个模块
	- 模块通过export导出, 通过import导入, 并通过Store声明时的modules进行使用
```js
// 模块创建与导出
const state = {
  userInfo: {
    name: '张三',
    age: 18
  }
}
const mutations = {}
const actions = {}
const getters = {}
export default {
  state,
  mutations,
  actions,
  getters
}
// 模块使用
import user from '@/store/modules/user'
import setting from '@/store/modules/setting'
const store = new Vuex.Store({
  modules: {
    user,
    setting
  }
})
```

**模块使用**
- 尽管已经分模块, 但是子模块的状态还是会挂载到根级别的state, 属性名就是模块名
- state访问
	- 直接通过模块名访问 `$store.state.模块名.xxx`
	- 通过mapState映射访问
		- 默认根级别的映射 -> `mapState(['xxx'])`
		- 子模块的映射 -> `mapState('模块名', ['xxx'])` -> 需要开启命名空间 (在导出选项中使用: `namespaced: true`)
- getters访问
	- 直接访问: `$store.getters['模块名/xxx']`
	- mapGetters映射 -> 同state
	- 分模块后, state指代子模块的state
- mutation调用
	- 默认模块中的mutation和actions会被挂载到全局, 需要**开启命名空间**, 才会挂载到子模块
	- 调用子模块中的mutation
		- 直接通过store调用`$store.commit('模块名/xxx', '参数')` - 需要开启命名空间
		- 通过mapMutations映射
			- 根级别映射 -> `mapMutations(['xxx'])`
			- 子模块的映射 -> `mapMutations('模块名', ['xxx'])` - 需要开启命名空间
- action调用
	- 类似mutation
	- actions中调用mutations, 不需要增加模块名前缀


### 综合案例

- 不同的数据应该分模块, 比如 购物车一个模块, 用户一个模块 等等
- 通过axios进行接口访问

**生成后端构建接口 - json-server**
1. 安装全局工具json-server
2. 代码根目录新建db目录 - 与src同级
3. 在db目录中构建index.json
4. 进入db目录, 执行命令, 启动后端接口服务 `json-server index.json`
5. 访问接口测试 `localhost:3000/cart`

**更新数据**
- 在vuex中进行数据修改时, 要同时更新前端vuex中的数据和后端数据库中的数据
- json-server中, 使用patch请求方式以及对应的id, 在参数中传入对应的属性名及其新值, 就可以完成对于值的更新操作
- 