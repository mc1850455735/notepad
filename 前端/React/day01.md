# React 基本程序

React 使用 default function App() 创建默认组件，并通过 export 关键字导出。

JSX 中使用 `{ }` 嵌入表达式，类似 Vue 中的 `{{ }}`。

通过 useState()，返回一个通过数组解构状态和内容的数据。前者负责渲染状态值，后者负责更新状态。
useState() 中传入的值决定了会创建一个什么类型的状态数据。

```jsx
const [count, setCount] = useState(0)
const [title, setTitle] = useState("")
const [todos, setTodos] = useState([])
```

通过组件的事件属性接收函数。

通过 html 代码，直接返回组件。

取二者之间的最大值，通过 Math.max()。

# 输入框与列表

`crypto.randomUUID()` 由浏览器提供，可以生成一个随机 UUID。

`...list` 负责将 list 展开，在列表中展开可以进行列表的拼接。
```jsx
const newTodo = {
	id: crypto.randomUUID(),
	title: text,
}

setTodos(preTodos => [...preTodos, newTodo])
```

使用表单时，`event.preventDefault()` 可以用来阻止表单默认的页面提交行为。

React 的条件元素。这种用法是一种条件表达式的短路用法。也可以使用三元表达式。
```jsx
{todos.length == 0 && <p>还没有任务, 添加一个吧...</p>}
```

React 循环创建元素。返回元素时，只能返回一个父元素。除了 map，也可以用其他循环语句，但是要注意必须收集到对应元素。
```jsx
<ul>
	{todos.map(todo => (
		<li key={todo.id}>{todo.title}</li>
	))}
</ul>
```

React 中通过 value 和 onChange 两个属性，对一个状态变量进行绑定。
```jsx
<input
	id="task-title"
	value={title}
	onChange={event => setTitle(event.target.value)}
	placeholder="请输入任务"
/>
```
