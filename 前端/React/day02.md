# 事件传参与过滤

删除时，可以使用 filter 过滤，直接去掉对应 id 的项。
```jsx
setTodos(preTodos => preTodos.filter(todo => todo.id !== id))
```

通过这个方式，可以先复制任务对象的属性，再使用后面的属性覆盖前面的属性。
```jsx
{...todo, completed: !todo.completed}
```

为按钮，复选框等绑定函数时，不要直接写函数调用，而是应该传入函数，如果需要固定传参的，可以使用匿名函数。
```jsx
<button onClick={() => handleDelete(todo.id)}>删除</button>
```

# 组件拆分

可以定义其他的 jsx 文件作为组件，类似 Vue。定义时，可以添加 default 关键字作为该文件组件的默认导出，也可以通过具名导出进行导出而不使用 default 关键字。

传入关键字时，可以通过对象解构的方式定义需要的参数，随后在父对象中传入这些关键字。使用函数时，类似之前的方式。

父组件负责保存状态，通过 props 传数据和函数；子组件通过回调请求更新，避免直接修改传入的数据。

定义：
```jsx
export default function TodoItem({todo, onToggle, onDelete}) {
	...
}
```
传参：
```jsx
todos.map(todo => (
    <TodoItem
        key={todo.id}
        todo={todo}
        onToggle={handleToggle}
        onDelete={handleDelete}
        ></TodoItem>
))
```

React 中的列表 `key` 使用稳定、唯一的任务 ID，放在 `map` 直接返回的元素上。它是 React 使用的特殊属性，子组件不能通过普通 props 读取 `key`。

# 存储与状态保存

通过 localStorage，可以保存并读取数据，其用法大致如下：
```jsx
localStorage.setItem("name", "小明")
localStorage.getItem("name")
localStorage.removeItem("name")
```

可以使用 `const STORAGE_KEY = "xxx"`
用来定义默认的存储 key。

保存到 localStorage 之前，要先将其转为字符串形式；加载时再从 JSON 字符串转为 JavaScript 对象形式。
```jsx
const jsonString = JSON.stringify(object)
const object = JSON.parse(text)
```

随后，初始化任务时，默认使用 localStorage 进行加载，找不到后使用默认数据或空数据。
```jsx
const [users, usersState] = useState(() => {
	try {
		const saved = localStorage.getItem(STORAGE_KEY)
		if (saved === null) return []
		const parsed = JSON.parse(saved)
		return Array.isArray(parsed) ? parsed : []
	} catch {
		return []
	}
})
```

当用户对数据进行修改时，调用相关函数，保存数据。需要注意：
- 根据状态计算显示数据时，直接计算即可；
- 需要将状态同步到浏览器存储时，使用 Effect。如：
```jsx
useEffect(() => {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(todos))
}, [todos])
```

useEffect 需要两个参数，第一个表示保存时要执行的函数，第二个表示依赖的数据。通过这个方法，当依赖数据发生变化时，执行对应的保存函数。
