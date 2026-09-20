# 基础组件 MeterialApp

整个应用被 MaterialApp 包裹，方便对整个应用的属性进行整体设计。

```dart
MaterialApp(
    title: ...,
    theme: ...,
    home: ...,
),
```

## title

决定了页面的标题。不需要加 Text，直接使用字符串即可。可以不设置。

```dart
title: "xxx的Demo02"
```

## theme

决定了页面的风格，其类型为 ThemeData。可以不设置。

ThemeData 中有多个属性，其中 scaffoldBackgroundColor 属性决定了 Scaffold 组件的背景色。

```dart
theme: ThemeData(
    scaffoldBackgroundColor: Colors.amber
)
```

## home

home 属性决定了该 MeterialApp 返回的页面。

```dart
home: Scaffold()
```

# 基础组件 Scaffold

Scaffold 组件是一个用于 Material Design 风格页面的核心布局组件，提供标准、灵活配置的骨架结构，分为上中下三部分。

```dart
Scaffold(
    appBar: ...,
    body: ...,
    bottomNavigationBar: ..., 
    backgroundColor: ...,
    floatingActionButton: ...,
)
```

## 属性

### appBar

标题栏，该属性类型为 AppBar。AppBar 组件有默认高度，其标题属性为 title，且该 title 属性类型为 Text，需要使用 Text() 类型组件，在其中传入想要的标题字符串。

```dart
appBar: AppBar(
    title: Text("xxx的Demo02")
)
```

### body

主体部分，占据了除去 appBar 和 bottomNavigationBar 以外的全部空间。

```dart
body: Container(
    child: Center(
        child: Text("Hello, World!")
    )
)
```

### bottomNavigationBar

底部。没有默认高度，如果不设置高度，则默认占据所有高度。使用时为了避免占据 body 空间，需要手动使用 height 属性设置组件高度。

```dart
bottomNavigationBar: Container(
    height: 80,
    child: Center(child: Text("底部区域")),
)
```

**height**
用于设置组件的高度。

**child**
用于设置该组件的子组件。

### backgroundColor

设置整个 Scaffold 的背景颜色。

### floatingActionButton

悬浮操作按钮，常用于触发页面的主要动作。

## 组件

### Container

Container，即容器。该组件可以作为一个容器类，用于承接其他组件。

**child**
Container 中 child 属性用于存放子组件。

### Center

Center 为居中组件，可以将其中的元素在页面中进行上下居中处理。

### Text

文本组件。

# 无状态组件

自定义组件的一种，纯展示型组件，没有用户交互操作。可以使用快捷指令 statelessW 快速创建模板。创建自定义无状态组件时，需要继承 StatelessWidget 类，并重写其 build 方法。build 方法的返回值即为组件结构。

**build**
build 方法需要返回一个 Widget 类型的对象，可以使用如 MaterialApp() 构造方法进行创建。

```java
class MainPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp();
  }
}
```

## 无状态组件 vs 有状态组件

| 特征     | StatelessWidget                | StatefulWidget                     |
| -------- | ------------------------------ | ---------------------------------- |
| 核心特征 | 一旦创建内部不可更改           | 持有可在其生命周期内改变的状态     |
| 使用场景 | 静态展示，外观只由配置参数决定 | 交互式组件，如计数器、表单输入框等 |
| 生命周期 | 相对简单，只有build            | 复杂，包含状态创建、更新和销毁     |
| 代码结构 | 单个类                         | 两个相互关联的类                   |

# 有状态组件

有状态组件是构建动态交互界面的核心，能够管理变化的内部状态，当状态变化时，组件会更新显示内容。

要创建一个有状态组件，通常分为两部分。第一个部分是对外展示的类，第二个部分是实际负责管理数据，处理业务逻辑，渲染视图的内部类。

可以使用 statefulW 快捷指令创建有状态组件模板。

```dart
// 有状态组件 第一个类 对外
class MainPage extends StatefulWidget {
    ...
}

// 有状态组件 内部类 负责管理数据, 处理业务逻辑, 并且渲染视图
class _MainPageState extends State<MainPage> {
    ...
}
```

## 对外类

对外展示的类继承自 StatefulWidget 类，需要重写 createState 方法，该方法返回一个 `State<StatefulWidget>`  类型的返回值，在不需要额外处理的情况下，可以直接创建并返回有状态组件的内部类。

```dart
class MainPage extends StatefulWidget {
    @override
    State<StatefulWidget> createState() {
        return _MainPageState();
    }
}
```

## 内部类

内部类继承自一个以外部类作为泛型参数的泛型类。如 MainPage 的内部类为 `_MainPageState`，则该内部类应该继承自 `State<MainPage>`。

内部类应该重写父类的 build 方法，该 build 方法同样返回一个 Widget 类，用于表示组件结构。

```dart
class _MainPageState extends State<MainPage> {
    @override
    Widget build(BuildContext context) {
        return MaterialApp(...);
    }
}
```

# 生命周期

生命周期就是 Widget 从创建到销毁的整个阶段，在这个过程中，会有一些函数被调用和执行。

## 无状态组件的生命周期

无状态组件只有一个生命周期，即 build 方法。

当组件被创建或者父组件状态变化导致其需要重新构建时，build 方法会被调用。无状态组件自身不会变化，但是可以通过外部参数的变换，即父组件状态变化，来使无状态组件发生变化。

## 有状态组件的生命周期

有状态组件生命周期分为三个阶段，分别是创建阶段、更新阶段和销毁阶段。

### 创建阶段

1. StatefulWidget 被创建
2. createState()：Widget 初始化时调用，用于创建 State 对象。
3. State 对象被创建
4. initState()：State 对象插入 Widget 树立即执行。**仅执行一次**。
5. didChangeDependencies()：initState 后立即执行，当所依赖的 InheritedWidget 更新时调用。**可执行多次**。
	- InheritedWidget 专用于在 Widget 树种自顶向下高效共享数据，顶层提供数据，子孙节点直接获取。
6. build()：用于构建 UI 的方法，初始化或更新后调用。**可多次调用**。

### 更新阶段

1. 父组件重构 -> 配置变更
2. didUpdateWidget()：父组件传入新配置时调用，用于比较新旧配置。**可执行多次**。
3. build()：更新结束后，同样需要进行一次 UI 渲染。

### 销毁阶段

1. 组件被移除
2. deactivate()：当 State 对象从 Widget 树中暂时移除时调用。**可执行多次**。
3. dispose()：当 State 对象被永久移除时调用，释放资源。**仅执行一次**。

# 点击事件

事件指的是用户与应用程序交互时触发的各种动作，如触摸、滑动、点击等操作。点击事件，即点击某个元素时触发的动作。

## 点击组件

点击组件名为 GestureDetector，是 Flutter 中最常用、功能最丰富的手势检测组件。在 GestureDetector 中，通过 child 属性来指定被包裹的 Widget 元素，并通过 GestureDetector 中的各种方法指定对应的动作。

GestureDetector 中提前定义了多个手势检测方法，如：
- onTap
- onDoubleTap
- onLongPress
- ...

```dart
GestureDetector(
    onTap: () {
        print("点击了该区域");
    },
    onDoubleTap: () {
        print("双击了该区域");
    },
    onLongPress: () {
        print("长按该区域");
    },
    child: Text("点我发消息"),
),
```

## 其他组件

Flutter 提供了多种方式为组件添加点击交互。

**专用按钮组件**：内置点击动画和样式，通过 onPressed 参数处理点击逻辑。
- ElevatedButton
- TextButton：可以通过 Container 设置 Button 形式。
- OutlineButton
- FloatingActionButton

**视觉反馈组件**：提供点击时间 onTap，有 MaterialDesign 风格的水纹扩散效果。
- InkWell

**其他交互组件**：具有特定功能的交互式空间、点击事件
- IconButton
- Switch
- CheckBox

```dart
TextButton(
    onPressed: () {
        print("按钮被按了");
    },
    child: Text("按钮, 点我"),
),
```

# 状态更新

## setState

当数据的变化需要更新 UI 视图时，需要执行 setState 方法，该方法会造成 build 方法的重新执行，从而实现 UI 视图的刷新。

其中数据更新的代码可以作为回调函数传入 setState 中，或不放入 setState 中，但是需要在 setState 前执行。最后通过执行 setState 方法，数据的变化可以被组件感知到，UI 视图被更新。

```dart
TextButton(
    onPressed: () {
        setState(() {
            count--;
        });
    },
    child: Text("减"),
)
```

## 组件

**Row**
通过 Row 组件，可以实现多个组件的并排排放。其中用于存放子组件的属性名为 children，向其中传入一个 `List<Widget>`，即组件形式的数组，实现多个组件的并排排放。

```dart
Row(
    children: [
        Text("减"), 
        Text("数字"), 
        Text("加")
    ]
)
```



