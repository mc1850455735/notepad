
# for循环

### 基础语法

- for循环的最大价值 : **循环数组**
	- 明确次数用for
	- 不明确次数用while
- 利用arr.length与for配合
- 使用数组时越界, 会返回undefined
- 可以通过`for(;;)`构建**无限循环**
```js
for(起始值;终止条件;变化量) {
	循环体
}
```

**退出循环**
- 使用continue退出当次循环时, 会自动执行**for循环**中的变化量语句
- 如果是while循环, 使用continue要提前进行递增

### 循环嵌套

```js
for(;;){
	for(;;){
		语句
	}
}
```

# 数组

### 数组是什么
- 数组(Array)是一种可以按顺序保存数据的**数据类型**
- 多个数据用数组保存 , 存放在一个变量中

### 数组使用

##### 声明数组
- `let arr = [元素1, 元素2, ...]`
- `let arr = new Array(数据1, 数据2, ...)`
- 数组**按顺序保存**, 每个元素都有自己的编号
- 数组中可以存放任意类型的元素

##### 取值语法
- `数组[下标]`

##### 遍历数组
- 把其中的每个元素都访问到
```js
for(let i = 0; i < arr.length; i++){
	arr[i]
}
```

### 操作数组
- 增删改查
- 查已经有了 `数组[下标]`

##### 修改数组
- `数组[下标] = 新的值`
- 可以通过**修改不存在的下标**的形式, 达到向数组中插入数据的作用
	- 不如push
- arr.length会以最大的有值的数组元素作为数组的length
- 当arr.length被撑大以后, 即使将之前赋值的数组元素修改为undefined, 数组也**不会缩小**

##### 添加元素
- `arr.push(新增内容)`
- `arr.unshift(新增内容)`

**数组.push()**
- `arr.push(元素1, 元素2, ..., 元素n)`
- 将一个或者多个元素添加到数组的**末尾**, 并**返回该数组的新长度**(push后的长度)

**数组.unshift()**
-  `arr.unshift(元素1, 元素2, ..., 元素n)`
- 将一个或者多个元素添加到数组的**开头**, 并**返回该数组的新长度**

##### 删除元素
- `arr.pop()`
- `arr.shift()`
- `arr.splice(操作的下标, 删除的个数)`
- 会导致length的减小
- 后期会用到删除操作, 特别是splice

**数组.pop()**
- 删除数组中最后一个的值, 并返回该元素的值
**数组.shift()**
- 删除数组中第一个的值, 并返回该元素的值
**数组.splice()**
- 删除指定的元素
- `arr.splice(start, deleteCount)`
- `start` : 表示修改开始下标位置
- `deleteCount` : 表示要移除的元素个数
	- 省略表示移除到最后

##### 数组排序

- 冒泡排序
```js
let arr = [12, 32, 21, 23, 12, 43, 54, 27, 21, 54]
let t
for (let i = 0; i < arr.length - 1; i++) {
  for (let j = 0; j < arr.length - i - 1; j++) {
	if (arr[j] > arr[j + 1]) {
	  t = arr[j]
	  arr[j] = arr[j + 1]
	  arr[j + 1] = t
	}
  }
}
console.log(arr)
```
- **数组.sort()**
升序排序
```js
arr.sort(function (a, b) {
  return a - b
})
console.log(arr);
```
降序排序
```js
arr.sort(function (a, b) {
  return b - a
})
console.log(arr);
```







