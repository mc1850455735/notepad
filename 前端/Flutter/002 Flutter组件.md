# 布局组件

Flutter 提供了丰富的布局组件用来构建各种用户界面。部分核心组件如下表

| 组件类别 | 核心组件                          | 主要特点和使用场景                                           |
| -------- | --------------------------------- | ------------------------------------------------------------ |
| 基础容器 | Container、Center、Align、Padding | 提供装饰、对齐、边距等基础样式和布局控制。使用频率高。       |
| 线性布局 | Row、Column                       | 在水平或垂直方向线性排列子组件，使构建页面的基础。           |
| 弹性布局 | Flex、Expanded、Flexible          | 按照比例分配剩余空间，实现自适应布局。常与 Row 和 Column 搭配使用 |
| 层叠布局 | Stack、Positioned                 | 让子组件重复堆叠，进而实现如图片上叠文字、悬浮按钮等效果     |
| 流式布局 | Wrap、Flow                        | 当主轴空间不足时自动换行或换列，常用于标签、滤镜等动态宽高内容的排列 |
| 滚动布局 | ListView、GridView                | 提供可滚动的列表或网格视图，高效展示大量数据                 |

## Container

Container 是功能丰富的布局组件，是一个多功能组合容器。

**尺寸控制**

可通过多种方式定义大小，有明确优先级规则：父组件约束 > Container 显式约束  > 自适应组件大小。通过向 width 和 height 属性传入 `double.infinity`，可以使对应宽高占满能占满的最大值。

**装饰系统**

通过 decoration 属性实现视觉效果，和 color 属性互斥。decoration 属性传入的类型为 `BoxDecoration()`。其中可以定义多种装饰类型，如背景色，边框弧度，边框样式等；

```dart
decoration: BoxDecoration(
    color: Colors.blue,
    borderRadius: BorderRadius.circular(15),
    border: Border.all(width: 3, color: Colors.amber),
)
```

**边框弧度**通过 `borderRadius` 属性进行设置，圆角属性为 `BorderRadius.circular(圆角弧度)`；

**边框样式**通过 border 属性进行设置，接收 Border 类型，通过 width 设置宽度，color 设置颜色。

**布局控制**

提供内外边距和对齐方式。其中：
- `margin`：外边距，接收 `EdgeInsets` 类型。
- `padding`：内边距，接受 `EdgeInsets` 类型。

**可选变化**

支持绘制时进行矩形变换，如旋转、倾斜、平移。

通过 transform 属性对 Container 进行旋转，传入 Matrix4，通过 `rotationX()` 等方法进行旋转，传入参数的单位为弧度。

**常见属性**

| 属性类别 | 关键属性                     | 作用说明                                                     |
| -------- | ---------------------------- | ------------------------------------------------------------ |
| 布局定位 | alignment                    | 控制其 child 在组件内部对齐的方式。                          |
| 尺寸控制 | width / height / constraints | 设置容器的宽度和高度 / 为容器设置尺寸约束。                  |
| 间距留白 | padding / margin             | 按照比例分配剩余空间，实现自定义布局，常与 Row / Column 配合使用。 |
| 装饰效果 | color / decoration           | 为容器设置简单的背景颜色 / 为容器设置复杂的背景装饰。        |
| 变换效果 | transform                    | 对容器及其内容进行矩阵变换。                                 |
| 子组件   | child                        | 容器内包含的唯一直接子组件。                                 |

```dart
Container(
    alignment: Alignment.center,
    margin: EdgeInsets.all(20),
    transform: Matrix4.rotationZ(0.05),
    width: 200,
    height: 200,
    // color: Colors.blue,
    decoration: BoxDecoration(
        color: Colors.blue,
        borderRadius: BorderRadius.circular(15),
        border: Border.all(width: 3, color: Colors.amber),
    ),
    child: Text(
        "Hello, Container!",
        style: TextStyle(color: Colors.white),
    ),
)
```

## Center

Center 组件将其子组件在父组件的空间内进行水平和垂直方向上的居中排列。在父组件是 Container 的情况下，也可以直接使用 `alignment: Alignment.Center` 实现元素居中。

```dart
Center(
    child: Container(
        width: 100,
        height: 100,
        color: Colors.blue,
        child: Center(child: Text("居中内容")),
    ),
)
```

**应用场景**
- **页面内容整体居中**，如将一个登录表单或一个加载中提示图在页面中央显示；
- Center 不能设置宽高，Center 的最终大小取决于**父组件传递给他的约束**，会尽可能向父组件申请**尽可能大的空间**；
- 实现宽高固定且居中的组件：使用 Center 包裹一个具有固定宽高的子组件 (Container / SizeBox)；

## Align

Center 主要用于居中对齐，而 Align 可以精确控制其子组件在父容器空间中的对齐位置。Center 实际上就是 Align 的一种特殊形式，他继承自 Align，相当于将 alignment 属性设为 Align.center。

当需要将一个组将放在父容器的特定角落时，选择使用 Align。

通过 Icon 组件，可以实现在页面中放入 Flutter 内置图标，同时可以使用 size 和 color 指定图标的大小和颜色等信息。

```dart
Icon(
    Icons.star, 
    size: 150, 
    color: Colors.amber
)
```

**属性**
- alignment：对齐方式，子组件在父容器中的对齐方式。
- widthFactor：宽度因子，Align 的宽度是子组件宽度乘以该因子。
- heightFactor：高度因子，Align 的高度是子组件高度乘以该因子。

宽度因子和高度因子在动态布局中很有用。

```dart
Align(
    alignment: Alignment.center,
    widthFactor: 3,
    heightFactor: 5,
    child: Icon(Icons.star, size: 50, color: Colors.amber),
),
```

## Padding

Padding 的作用是为子组件添加内边距。该组件只有两个属性，child 用于接收子组件，而 padding 用于设置内边距的大小。

padding 属性的类型为 EdgeInsets。在 Padding 中，该属性是必须的，其定义了内边距的大小和方向。

当四个方向设置相同间距时，使用 `EdgeInsets.all()`；

```dart
Padding(
    padding: EdgeInsetsGeometry.all(10),
    child: Container(color: Colors.blue),
),
```

当需要设置某个或某几个方向的 Padding 时，使用 `EdgeInsets.only(top, bottom, left, right)`。

```dart
Padding(
    padding: EdgeInsetsGeometry.only(top: 10, bottom: 10),
    child: Container(color: Colors.blue),
),
```

当想要设置的 Padding 方向恰好对称时，可以使用 `EdgeInsets.only(vertical, horizontal)`。

```dart
Padding(
    padding: EdgeInsetsGeometry.symmetric(vertical: 10, horizontal: 20),
    child: Container(color: Colors.blue),
)
```

除了使用 Padding 外，还有很多方式用于添加内边距，他们的使用方法都是相同的：

- Container 容器的 padding 属性；
- decoration 属性的 BoxDecoration 类中，也允许添加 padding 属性。

当不需要使用 Container 的其他功能，只需要 padding 时，优先选择 Padding 而非 Container。

## Column

Column 用于垂直排列其子组件的核心布局容器，其中所有子组件都会按纵向的方式排列。几乎所有需要垂直排列元素的界面中都能看到他，如：表单列表、设置列表、卡片布局、图文混排 等。

注意，Column 本身不支持滚动，如果内容超出，需要使用 ListView 或 SingleChildScrollView 包裹；同时，父组件的大小直接影响到 Column 的最终大小和子组件布局行为，需要明确尺寸约束；代码书写时，需要避免过度嵌套，嵌套过深会影响性能并增加代码维护难度。

Column 的默认主轴是垂直方向的，交叉轴是水平方向的。

| 属性               | 类型               | 作用说明                                                     |
| ------------------ | ------------------ | ------------------------------------------------------------ |
| mainAxisAlignment  | MainAxisAlignment  | 控制子组件在主轴上的排列方向，如顶部对齐、居中或均匀分布。   |
| crossAxisAlignment | CrossAxisAlignment | 控制子组件在交叉轴上的排列方向，如左对齐、右对齐或拉伸填满。 |
| mainAxisSize       | MainAxisSize       | 决定 Column 本身在垂直方向上的尺寸策略：占满所有空间或仅包裹子组件内容。 |
| children           | `List<Widget>`     | 需要被垂直排列的子组件列表。                                 |

Column 的 mainAxisAlignment 和 crossAxisAlignment 都支持多种排列方式；

在 `mainAxisAlignment`，支持的排列方式如下：
- spaceEvently
- spaceAround
- spaceBetween
- start
- end
- center

同时，在排列的过程中，可以使用 margin 外边距控制各个组件之间的距离 (参考前端)。

```dart
Column(
    // mainAxisAlignment: MainAxisAlignment.spaceBetween,
    // mainAxisAlignment: MainAxisAlignment.spaceAround,
    // mainAxisAlignment: MainAxisAlignment.spaceEvenly,
    mainAxisAlignment: MainAxisAlignment.center,
    children: [
        Container(
            width: 100,
            height: 100,
            margin: EdgeInsets.only(top: 10),
            color: Colors.blue,
        ),
        Container(
            width: 100,
            height: 100,
            margin: EdgeInsets.only(top: 10),
            color: Colors.blue,
        ),
        Container(
            width: 100,
            height: 100,
            margin: EdgeInsets.only(top: 10),
            color: Colors.blue,
        ),
    ],
),
```

`crossAxisAlignment` 支持的排列方式如下：
- start
- center
- end
- baseline
- stretch

可以使用这些属性指定位置。

```dart
crossAxisAlignment: CrossAxisAlignment.start
```

## Row

类似于 Column，用于水平排列其子组件的核心布局容器。属性列表与 Column 相同。

对于 Row，几乎所有需要水平排列元素的界面都需要使用它，如导航栏、图文混排、表单行等。

类似的，Row 本身不支持滚动，如果内容超出，需要使用 ListView 或 SingleChildScrollView 包裹；父组件大小直接影响 Row 的最终大小和子组件布局行为。

```dart
Row(
    mainAxisAlignment: MainAxisAlignment.spaceAround,
    children: [
        Container(width: 100, height: 100, color: Colors.blue),
        Container(width: 100, height: 100, color: Colors.blue),
        Container(width: 100, height: 100, color: Colors.blue),
    ],
)
```

## Flex

Flex 组件允许沿一个主轴 (水平或垂直) 排列其子组件，灵活的控制这些子组件在主轴上的尺寸比例和空间分配，其可以看作 Column 组件和 Row 组件的结合体。其常用分布属性参见 Column。

Flex 布局受其父组件传递的约束影响，使用时需要确保父组件提供了适当的布局约束。

通过 direction 属性，可以设定 Flex 组件的主轴方向；`Axis.horizontal` 为水平方向，`Axis.vertical` 为垂直方向。

Flex 的子组件常使用 Expanded 或 Flexible 来控制空间分配，通过 flex 属性分配 Flex 组件空间，flex 属性默认为 1。

Expanded 和 Flexible 都可以实现空间适应的效果，但 Expanded 组件强制子组件填满所有剩余空间，而 Flexible 默认使用最小子组件空间。如果需要 Flexible 也强制填满空间，需要设置 fit 属性为 FlexFit.tight 以开启。

```dart
Flex(
    direction: Axis.vertical,
    children: [
        Expanded(
            flex: 2,
            child: Container(width: 100, height: 100, color: Colors.red),
        ),
        Expanded(
            flex: 1,
            child: Container(width: 100, height: 100, color: Colors.green),
        ),
    ],
),
```

## Wrap

Wrap 是流式布局组件，当子组件在主轴方向上排列不下时，会自动换行 / 换列。对于 Column / Row / Flex，内容超出均不会换行。

Wrap 组件更像是 Flex 组件加了换行特性。当子组件内容是根据数据动态生成的时，使用 Wrap 可以保证布局始终适配。

| 属性         | 常用值                          | 作用说明                        |
| ------------ | ------------------------------- | ------------------------------- |
| direction    | Axis.horizontal / Axis.vertical | 设置主轴方向，即排列方向        |
| spacing      | 数值                            | 主轴方向上，子组件之间的间距    |
| runSpacing   | 数值                            | 交叉轴方向上，行 / 列之间的间距 |
| alignment    | WrapAlignment                   | 子组件在主轴方向上的对齐方式    |
| runAlignment | WrapAlignment                   | 交叉轴方向上的对齐方式          |

```dart
Wrap(
    spacing: 5,
    runSpacing: 5,
    alignment: WrapAlignment.spaceAround,
    direction: Axis.horizontal,
    children: getList(),
),
```



当需要批量生成组件时，可以利用函数，同时在函数中使用 `List.generate(nums, function)` 方法进行批量生产。

```dart
List<Widget> getList() {
    return List.generate(10, (index) {
        return Container(width: 100, height: 100, color: Colors.blue);
    });
}
```

## Stack / Positioned

Stack 是层叠布局组件，允许将多个子组件按照 Z 轴方向进行叠加排列。直接使用 Stack 堆叠时，children 数组中靠后的元素覆盖在在靠前元素的上面。Stack 和 Positioned 元素适用于几乎所有需要叠加效果的界面上，如图像上的水印、提示弹窗、悬浮按钮等。

Stack 也需要注意明确尺寸约束，父组件的大小直接影响 Stack 的最终大小和子组件的布局行为。

如果想要 Stack 拥有大小，可以使用 Container 组件包裹 Stack。

```dart
Container(
    width: double.infinity,
    height: double.infinity,
    color: Colors.amber,
    child: Stack(...),
),
```

| 属性         | 类型              | 作用说明                                          |
| ------------ | ----------------- | ------------------------------------------------- |
| alignment    | AlignmentGeometry | 控制非定位子组件在 Stack 内的对齐方式，默认左上角 |
| fit          | StackFit          | 控制非定位子组件如何适应 Stack 的尺寸             |
| clipBehavior | Clip              | 控制子组件超出 Stack 边界时的裁剪方式             |
| children     | `List<Widget>`    | 需要被层叠排列的子组件列表                        |

```dart
Stack(
    alignment: Alignment.topRight,
    children: [
        Container(width: 300, height: 300, color: Colors.blue),
        Container(width: 200, height: 200, color: Colors.red),
        Container(width: 100, height: 100, color: Colors.amber),
        Container(width: 50, height: 50, color: Colors.green),
    ],
),
```

### Positioned组件

Positioned 组件是 Stack 的黄金搭档，可以实现对子组件的精确定位控制。Positioned 必须为 Stack 的直接子组件，可以不用 Positioned，但是如果需要使用必须作为直接子组件。

Positioned 通过 left、right、top、bottom 将子组件放置在 Stack 的角落或边缘。当 Postion 同时设置 left & right / top & bottom 时，会拉伸组件大小至满足 Positioned 定位属性宽度。

```dart
Stack(
    children: [
        Container(width: 300, height: 300, color: Colors.grey),
        Positioned(
            left: 20,
            top: 20,
            child: Container(width: 100, height: 100, color: Colors.red),
        ),
        Positioned(
            right: 20,
            bottom: 20,
            child: Container(width: 100, height: 100, color: Colors.blue),
        ),
        Positioned(
            right: 20,
            left: 20,
            bottom: 0,
            child: Container(width: 10, height: 10, color: Colors.green),
        ),
    ],
),
```

<img src="D:\Majinliang\Documents\笔记\前端\Flutter\Inbox\image-20260322171736783.png" alt="image-20260322171736783" style="zoom:50%;" />

# 功能组件

## Text

Text 组件用于在用户界面中显示文本，所有的文本显示都需要使用 Text 组件。如果需要在同一段文本中显示不同样式，可以使用 Text.rich 构造函数配合 TextSpan 实现。

Text 组件本身和其 TextStyle 中都有可能有 overflow 属性，此时 Text 组件优先级更高。

大量重复使用的文本样式建议统一定义，有助于保持一致性，提升性能。

| 属性      | 类型      | 作用说明                                           |
| --------- | --------- | -------------------------------------------------- |
| data      | String    | 必需。要显示的文本内容。                           |
| style     | TextStyle | 文本样式，可设置颜色、大小、粗细等。               |
| textAlign | TextAlign | 文本在容器内的水平对齐方式，如 `.left`，`.right`。 |
| maxLines  | int       | 文本显示的最大行数。                               |

### style属性的使用

```dart
Text(
    style: TextStyle(
        fontSize: 30,
        color: Colors.blue,
        fontStyle: FontStyle.italic,
        fontWeight: FontWeight.bold,
        decoration: TextDecoration.underline,
        decorationColor: Colors.red,
    ),
    "Hello, Flutter!",
),
```

![image-20260322213542917](D:\Majinliang\Documents\笔记\前端\Flutter\Inbox\image-20260322213542917.png)

### 行数控制

当超出 maxLines 时，可以使用 overflow 属性，指定对超出文本的行为 (如自动转为省略号)。

```dart
Text(
    style: TextStyle(color: Colors.blue, fontSize: 30),
    maxLines: 2,
    overflow: TextOverflow.ellipsis,
    "今天天气不错今天天气不...",	// 过长省略
),
```

![image-20260322213433230](D:\Majinliang\Documents\笔记\前端\Flutter\Inbox\image-20260322213433230.png)

### TextSpan

通过使用 Text.rich() 构造函数并传入 TextSpan 组件，可以实现对文本形式的单独设置。其中外层设置的是全局样式，内层设置的是相对样式。

```dart
Text.rich(
    TextSpan(
        style: TextStyle(
            fontSize: 30,
            color: Colors.red,
            fontWeight: FontWeight.bold,
            decoration: TextDecoration.underline,
            decorationColor: Colors.yellow,
            decorationThickness: 3,
        ),
        text: "Hello ",
        children: [
            TextSpan(
                style: TextStyle(color: Colors.green),
                text: "Flutter",
            ),
            TextSpan(text: "!"),
        ],
    ),
),
```

![image-20260322215137699](D:\Majinliang\Documents\笔记\前端\Flutter\Inbox\image-20260322215137699.png)

## Image

Image 组件是用于在用户界面中显示图片的核心组件。

**分类**

| 分类            | 作用说明                                                     |
| --------------- | ------------------------------------------------------------ |
| Image.asset()   | 加载项目资源目录 (assets) 中的图片。需要在 pubspec.yaml 文件中声明资源路径 |
| Image.network() | 直接从网络地址加载图片。                                     |
| Image.file()    | 加载设备本地存储中的图片文件。                               |
| Image.memory()  | 加载内存中的图片数据。                                       |

**常用属性**

| 分类           | 类型              | 作用说明                                             |
| -------------- | ----------------- | ---------------------------------------------------- |
| width / height | double            | 设置图片显示区域的宽度和高度                         |
| fit            | BoxFit            | 控制图片如何适应其显示区域，如拉伸、裁剪或保持原比例 |
| alignment      | AlignmentGeometry | 图片在其显示区域内的对齐方式                         |
| repeat         | ImageRepeat       | 当图片小于显示区域时，设置是否以及如何重复平铺图片   |

### Image.asset()

**步骤**

1. 配置 pubspec.yaml 文件，在 flutter.assets 配置项下配置要读取的图片名，或直接加载整个文件夹。配置 pubspec.yaml 时，要额外注意缩进问题。

```yaml
flutter:
  uses-material-design: true
  assets:
    - lib/images/
```

2. 在文件中添加 Image 组件，并在其中传入图片文件路径。要注意的是，由于配置文件修改过，而当前运行的程序没有这个配置，无法识别到新添加的路径，需要重新启动一次程序。

```dart
Container(
    alignment: Alignment.center,
    width: double.infinity,
    height: double.infinity,
    color: Colors.amber,
    child: Image.asset(
        "lib/images/github.png",
        width: 100,
        height: 100,
        // fit: BoxFit.contain,
        // fit: BoxFit.cover,
        // fit: BoxFit.fill,
        fit: BoxFit.fitHeight,
    ),
),
```

<img src="D:\Majinliang\Documents\笔记\前端\Flutter\Inbox\image-20260328210445472.png" alt="image-20260328210445472" style="zoom:33%;" />

### Image.network()

相比 `asset()`，`network()` 配置相对简单，不需要添加资源文件或设置资源路径，只需要传入资源地址，并保证网络畅通即可。

在 Web 环境下不需要关心网络权限，但是在如 安卓、IOS、鸿蒙 等环境都需要确保软件具有网络权限。

```dart
Container(
    alignment: Alignment.center,
    width: double.infinity,
    height: double.infinity,
    color: Colors.amber,
    child: Image.network(
        "https://inews.gtimg.com/news_ls/OSBSD0f6k8Md_QZ3W7JJunUuywoclkUuurTrwDgmYq-YwAA_870492/0",
        width: 300,
        height: 300,
        fit: BoxFit.fill,
    ),
),
```

<img src="D:\Majinliang\Documents\笔记\前端\Flutter\Inbox\image-20260328210850546.png" alt="image-20260328210850546" style="zoom:33%;" />

## TextField

TextField 是用来实现文本输入的核心组件，使用时需要控制其中的内容。使用 TextField 时，必须使用**有状态组件**，因为 TextField 需要管理自己的状态，而只有有状态组件才能管理自己的状态。

其核心属性如下：

| 属性        | 作用说明                                                 |
| ----------- | -------------------------------------------------------- |
| controller  | 文本编辑器控制器，用于获取、设置文档内容以及监听内容变化 |
| decoration  | 控制输入框的外观，如标签、提示文字、图标、边框等         |
| style       | 定义输入文本的样式                                       |
| maxLines    | 最大行数                                                 |
| onChanged   | 输入内容发生变化时执行的回调函数                         |
| onSubmitted | 用户提交输入时的回调函数                                 |

### UI 结构绘制

decoration 使用的类型为 **InputDecoration**。设定背景颜色时，需要同时设定 fillColor 和 filled，前者用于设定背景颜色，后者用于允许显示背景颜色。hinitText 用于设置输入框的提示文本。同时，通过 obscureText 属性可以控制是否显示文本框内实际内容。

在 decoration 中，通过其中的 border 属性，可以控制输入框的边框信息，如是否圆角、是否有边框等。通过 borderSide，可以控制是否显示输入框的边框；通过 borderRadius 可以控制输入框的圆角。

在 decoration 中，通过 contentPadding 属性，可以控制输入框中内容的 padding。

使用 **obscureText** 属性，可以设定是否显示实际内容，通常用于密码输入框。

```dart
Container(
    padding: EdgeInsets.all(20),
    color: Colors.white,
    child: Column(
        children: [
            TextField(
                decoration: InputDecoration(
                    contentPadding: EdgeInsets.only(left: 20),
                    border: OutlineInputBorder(
                        borderSide: BorderSide.none,
                        borderRadius: BorderRadius.circular(25),
                    ),
                    fillColor: const Color.fromARGB(255, 255, 246, 164),
                    filled: true,
                    hintText: "请输入账号",
                ),
            ),
            SizedBox(height: 10),
            TextField(
                obscureText: true,
                decoration: InputDecoration(
                    contentPadding: EdgeInsets.only(left: 20),
                    border: OutlineInputBorder(
                        borderSide: BorderSide.none,
                        borderRadius: BorderRadius.circular(25),
                    ),
                    fillColor: const Color.fromARGB(255, 255, 246, 164),
                    filled: true,
                    hintText: "请输入密码",
                ),
            ),
            SizedBox(height: 10),
            Container(
                height: 50,
                width: double.infinity,
                decoration: BoxDecoration(
                    color: Colors.black,
                    borderRadius: BorderRadius.circular(25),
                ),
                child: TextButton(
                    onPressed: () {},
                    child: Text("提交", style: TextStyle(color: Colors.white)),
                ),
            ),
        ],
    ),
),
```

效果如下：

<img src="D:\Majinliang\Documents\笔记\前端\Flutter\Inbox\image-20260330215645631.png" alt="image-20260330215645631" width=300px />

### 功能实现

**controller**

为获取 TextField 中的内容，需要为 TextField 的 controller 属性传入一个 **TextEditingController** 类型组件，以实现获取输入框内容的功能。

通常在有状态组件的对内类中声明控制器，并在相关的方法中通过该控制器获取组件中的内容并对组件进行操作。

```dart
class MainPage extends StatefulWidget { ... }

// 在内部类中定义控制器
class _MainPageState extends State<MainPage> {
    TextEditingController _phoneController = TextEditingController();
    TextEditingController _passwordController = TextEditingController();
    @override
    Widget build(BuildContext context) {
        TextField(
            controller: _phoneController,
        ),
        TextField(
            controller: _passwordController,
        ),
        Container(
            child: TextButton(
                onPressed: () {
                    print("登录-账号-${_phoneController.text}");
                    print("登录-密码-${_passwordController.text}");
                },
                child: Text("提交", style: TextStyle(color: Colors.white)),
            ),
        ),
    }
}
```

<img src="D:\Majinliang\Documents\笔记\前端\Flutter\Inbox\image-20260330221918383.png" alt="image-20260330221918383" width=300px />![image-20260330221940759](D:\Majinliang\Documents\笔记\前端\Flutter\Inbox\image-20260330221940759.png)

**onChanged 和 onSubmit**

onChanged 回调方法在输入框中内容发生变化时调用，onSubmit 方法在输入框中内容提交时进行调用。在浏览器中，通过在输入框中回车触发 onSubmit 函数；在手机端通过键盘上的提交按钮触发 onSubmit 函数。

```dart
TextField(
    onChanged: (value) {
        print("onChanged: $value");
    },
    onSubmitted: (value) {
        print("onSubmitted: $value");
    },
),
```

![image-20260330222457925](D:\Majinliang\Documents\笔记\前端\Flutter\Inbox\image-20260330222457925.png)

# 滚动组件

| 组件                  | 特点                                            | 使用场景                                     |
| --------------------- | ----------------------------------------------- | -------------------------------------------- |
| SingleChildScrollView | 让单个组件可以滚动，所有内容一次性全部渲染      | 长表单、设置页、内容量不固定但总量不多的页面 |
| ListView              | 线性列表，可以通过 builder 实现懒加载，性能优异 | 聊天记录、新闻、常见的单列滚动的数据列表     |
| GridView              | 网格布局列表，支持懒加载，可以固定列数          | 图片墙、商品网格、应用图标列表               |
| CustomScrollView      | 复杂布局方案，通过组合多个 Sliver 组件实现滚动  | 电商首页、社交 App 个人主页多个滚动紧密联动  |
| PageView              | 整页滚动效果，支持横向和纵向                    | 应用引导页、图片轮播页、书籍翻页             |

## SingleChildScrollView

SingleChildScrollView 组件用于包裹一个子组件，让单个子组件具备滚动能力。

通过 List.generate 方法可以快速生成一系列组件。

```dart
Column(
    children: List.generate(100, (index) {
        return Container(
            margin: EdgeInsets.only(top: 10),
            width: double.infinity,
            height: 100,
            alignment: Alignment.center,
            color: Colors.blue,
            child: Text(
                "我是第${index + 1}个",
                style: TextStyle(color: Colors.white, fontSize: 30),
            ),
        );
    }),
),
```

生成完后效果如下，可以看到提示 "Buttom overflowed by 10326 pixels"，即内容超出页面 (因内容无法滚动查看)：

<img src="D:\Majinliang\Documents\笔记\前端\Flutter\Inbox\image-20260331165853203.png" alt="image-20260331165853203" width=300px />

通过 SingleChildScrollView，可以使单个子组件具备滚动功能。同时可以设置 padding 内边距属性。

<img src="D:\Majinliang\Documents\笔记\前端\Flutter\Inbox\image-20260409164727582.png" alt="image-20260409164727582" width=300px />

### 滚动控制

通过 SingleChildScrollView 的 controller 属性，可以给组件绑定对应的 ScrollController 对象。使用 Controller 组件，可以控制 SingleChildScrollView 的滚动位置等。

**去底部 / 顶部的实现**



## ListView

ListView 组件是用于构建可滚动列表的核心部件，且提供了流畅的滚动体验。

该组件提供多种构造函数, 如默认构造函数、ListView.builder、ListView.separated。

除默认构造函数为非懒加载，其他构造方式都采用按需渲染的懒加载模式，只构建当前可见区域的列表项，极大提升长列表性能。

ListView 中同样存在 controller 属性，其接受值类型与 SingleChildSrcollView 相同，控制滚动方式也相同。

### 默认构造模式

默认构造函数会一次性构建所有选项，适用于静态数量有限数据一次性构建所有表项。

```dart
ListView(
    padding: EdgeInsets.all(10),
    children: List.generate(100, (index) {
        return Container(
            margin: EdgeInsets.only(top: 10),
            width: double.infinity,
            height: 80,
            alignment: Alignment.center,
            decoration: BoxDecoration(color: Colors.blue),
            child: Text(
                "第${index + 1}个",
                style: TextStyle(color: Colors.white, fontSize: 30),
            ),
        );
    }),
),
```

<img src="D:\Majinliang\Documents\笔记\前端\Flutter\Inbox\image-20260328111052112.png" alt="image-20260328111052112" style="zoom:50%;" />

### ListView.builder

builder 模式是处理长列表或动态数据的首选和推荐方式。

其接受一个 **itemBuilder 回调函数**来按需构建列表项，通过 **itemCount** 控制列表长度。回调函数需要传入两个参数，分别是构建 UI 上下文 BuilderContext 和 当前组件下标 index。每次构建新表项时，都会调用 itemBuilder。

优势：按需构建，不会在初始化时将所有列表项都创建，而是根据用户的滚动行为动态地创建和销毁列表项。

### ListView.separated

ListView.separated 模式相当于在 ListView.builder 的基础上，额外提供了构建分割线的能力。对于该构造方法，需要同时提供 itemBuilder、separatorBuilder、itemCount 三个属性。separatorBuilder 提供一个回调函数，用于构建分割线组件。

```dart
ListView.separated(
    itemBuilder: (BuildContext context, int i) {
        return Container(
            width: double.infinity,
            height: 80,
            alignment: Alignment.center,
            decoration: BoxDecoration(color: Colors.blue),
            child: Text(
                "第${i + 1}个",
                style: TextStyle(color: Colors.white, fontSize: 30),
            ),
        );
    },
    separatorBuilder: (BuildContext context, int i) {
        return Container(
            height: 10,
            width: double.infinity,
            color: Colors.pink,
        );
    },
    itemCount: 100,
),
```

<img src="D:\Majinliang\Documents\笔记\前端\Flutter\Inbox\image-20260328204414246.png" alt="image-20260328204414246" style="zoom:50%;" />

## GridView

用于创建二维可滚动网格布局的核心组件，可以通过 scrollDirection 属性设置滚动的方向。

GridView 提供多种构建方式，如：
- GridView.count() - 基于固定列数的网格布局 (最常用)
- GridView.entent() - 基于固定子项的最大宽度 / 高度的网格布局 (最常用)
- GridView.builder() - 懒加载策略，用于网格项数量巨大或动态生成的情况，需要接收 gridDelegate 布局委托属性。gridDelegate 可接收的属性类型如下：
	- `SliverGridDelegateWithFixedCrossAxisCount`：固定列数
	- `SliverGridDelegateWithFixedCrossAxisExtent`：最大宽度 
	- 在 GridView.builder 中，在 gridDelegate 中声明主轴间距和交叉轴间距。

同时 GridView 存在默认构造方式，但是写起来过于繁琐，通常不使用。

### GridView.count()

通过该方法，可以构造固定列数网络。GridView.count 以列数为优先。指定网格列数后，Flutter自动计算列的宽度，在空间内均匀排列。要注意的是，当滚动方向发生变化时，网格中组件的大小会随着横 / 纵空间大小到的不同而发生变化。

```dart
GridView.count(
    // 修改滚动方向为横向
    // 修改以后每个组件都会变大一点点, 因为固定空间放三个组件
    // 而竖向比横向空间更大
    scrollDirection: Axis.horizontal,
    padding: EdgeInsets.all(10),
    mainAxisSpacing: 10,
    crossAxisSpacing: 10,
    crossAxisCount: 3,
    children: List.generate(100, (index) {
        return Container(
            alignment: Alignment.center,
            decoration: BoxDecoration(color: Colors.blue),
            child: Text(
                "第${index + 1}个",
                style: TextStyle(color: Colors.white),
            ),
        );
    }),
),
```

<img src="D:\Majinliang\Documents\笔记\前端\Flutter\Inbox\image-20260327201259945.png" alt="image-20260327201259945" width=300px /><img src="D:\Majinliang\Documents\笔记\前端\Flutter\Inbox\image-20260327201242808.png" alt="image-20260327201242808" width=300px/>

### GridView.extent()

使用 GridView.extent 指定子项的最大宽度或高度。GridView.extent 通过 maxCrossAxisExtent 设置子项最大宽度 / 高度来计算横向或纵向有多少列。

```dart
GridView.extent(
    scrollDirection: Axis.vertical,
    // 由当前方向的最大宽度 / 高度决定该方向的组件数量
    maxCrossAxisExtent: 200,
    mainAxisSpacing: 10,
    crossAxisSpacing: 10,
    padding: EdgeInsets.all(10),
    children: List.generate(100, (index) {
        return Container(
            alignment: Alignment.center,
            decoration: BoxDecoration(color: Colors.blue),
            child: Text(
                "第${index + 1}个",
                style: TextStyle(color: Colors.white),
            ),
        );
    }),
),
```

<img src="D:\Majinliang\Documents\笔记\前端\Flutter\Inbox\image-20260327203232620.png" alt="image-20260327203232620" width=300px />

### GridView.builder()

使用 GridView.builder 实现动态长网络，该构造方式对组件懒加载，只渲染可见区域。使用时，需要接收布局委托 (通过列数 (类似 count ) 还是宽高 (类似 extent ) 进行网格限制)，同时类似 ListView，传入 itemBuilder 构造函数和 itemCount。

在 GridView.builder 中，在 gridDelegate 布局属性中声明主轴间距和交叉轴间距。同时，还可以使用 childAspectRatio 控制子组件宽高比。

```dart
GridView.builder(
    padding: EdgeInsets.all(10),
    // 布局委托, 此时为固定列数模式
    gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: 3,
        mainAxisSpacing: 10,
        crossAxisSpacing: 10,
        childAspectRatio: 2,
    ),
    itemCount: 100,
    itemBuilder: (BuildContext buildContext, int index) {
        return Container(
            alignment: Alignment.center,
            decoration: BoxDecoration(color: Colors.blue),
            child: Text(
                "第${index + 1}个",
                style: TextStyle(color: Colors.white),
            ),
        );
    },
),
```

<img src="D:\Majinliang\Documents\笔记\前端\Flutter\Inbox\image-20260327204512306.png" alt="image-20260327204512306" style="zoom:50%;" /><img src="D:\Majinliang\Documents\笔记\前端\Flutter\Inbox\image-20260327204849836.png" alt="image-20260327204849836" style="zoom:50%;" />

