### 类声明与基本数据

##### 类声明
- 该类继承自`FDSyncedTileEntity`类
	- `FDSyncedTileEntity`为模组自定义的tile父类, 可以用于半自动完成服务端与客户端的同步操作(tile变化时进行同步)

##### 基础数据

**ItemStackHandler**
- 物品处理类, 用于管理tile中物品槽或物品堆栈的逻辑
- 通常用于实现容器类方块

**`LazyOptional<IItemHandler>`**
- 延迟加载可选能力包装器
- 可以用来
	- 避免空指针异常 (某些可能成为null的对象)
	- 延迟加载 (需要使用时再加载, 减少不必要的开销)
	- 自动失效 (持有方块无效时, 自动标记为无效状态)
	- 提供线程安全的访问方式
![[游戏开发/Minecraft/农夫乐事阅读笔记/tile/Inbox/deepseek_mermaid_20250704_5f6f76.png]]

**ResourceLocation**
- 用来匹配上一次加载成功的配方ID

**isItemCarvingBoard**
- 切菜板状态标志位, 用于

### 构造函数

**CuttingBoardTileEntity - 参数**
- `super(...)` - 向父类指明对应TileEntity的类型, 用于系统创建对应实体
	- `ModTileEntityTypes.CUTTING_BOARD_TILE.get()` - 从注册表中获取对应tile的类型
- `inventory = createHandler()` - 创建一个handler给予tile
	- 给切菜板实例内部创建
	- `createHandler()` - 创建一个物品处理器`ItemStackHandler`
	- `ItemStack` : 单个物品栈实例
	- `ItemStackHandler` : 物品堆处理器
- `inputHandler = LazyOptional.of(() -> inventory)`
	- 外部交互
	- 将物品栏通过`LazyOptional`包装为延迟可选对象
	- `LazyOptional`为Forge提供的容器类, 用于封装可能为null的对象
	- 使用`() -> inventory`, 确保只有在需要时才获取inventory实例
- `isItemCarvingBoard = false`
	- 初始化状态标志位`isItemCarvingBoard`为false
	- 该标志位的作用为






