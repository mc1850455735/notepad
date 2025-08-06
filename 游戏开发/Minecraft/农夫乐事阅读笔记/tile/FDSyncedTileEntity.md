该类为Farmer Delighted的自定义基类TileEntity, 内置了一些方法, 用于实现自动同步功能

### 初始化

- 参数
	- `TileEntityType<?> tileEntityTypeIn` : 方块实体类型, 用于表示当前的TileEntity属于哪个特定的TileEntity

### getUpdatePacket

- 用于将当前的`TileEntity`同步到客户端
	- 因为Minecraft中服务端TileEntity的变化不会自动同步到客户端
	- 如果想要客户端看到服务端中相应方块的变化, 就需要手动同步
	- 通过发送同步包进行同步
- `SUpdateTileEntityPacket` 是一种类型, 表示服务端发送给客户端的数据更新包
- 客户端收到包后, 会通过调用`handleUpdateTag`方法同步包中内容
- `new SUpdateTileEntityPacket(worldPosition, 1, getUpdateTag())`
	- worldPosition : 要同步TileEntity的坐标
	- 1 : 标识符, 一般是1
	- `getUpdateTag()` : 获取当前存储的NBT数据
- 客户端与服务端的同步通常分为三步
	- 通过 `getUpdateTag` 获取当前 tile 的 nbt
	- 通过 `getUpdatePacket` 向客户端发送网络包
	- 通过 `onDataPacket` , 客户端处理网络包


### getUpdateTag

内部代码为 `return save(new CompoundNBT());`
将方块中存储的NBT标签打包返回

### onDataPacket
客户端处理接收到的网络包
- 参数
	- `NetworkManager net` : 代表这次传输所用的网络连接, 一般不会用到, 除非调试或者是高级用途 (可忽略)
	- `SUpdateTileEntityPacket pkt` : 服务器传来的更新数据包

### save和load
- save : 将TileEntity数据打包成为NBT数据包
- load : 将NBT数据包加载成为TileEntity数据

### inventoryChanged
- 作用: 当tile中物品栏发生变化时, 提醒世界更新, 保证服务端与客户端的数据同步与视觉刷新
- `super.setChanged()` : 调用基类方法, 标记TileEntity状态已改变, 需要保存到磁盘
- `level.sendBlockUpdated(getBlockPos(), getBlockState(), getBlockState(), Constants.BlockFlags.BLOCK_UPDATE)` : 通知客户端刷新
	- 既: 当方块的TileEntity发生变化时, 即使方块的state没有变化, 也需要刷新状态
	- 该函数参数分别对应: 方块位置, 原状态, 新状态, 通知客户端进行视觉更新


### 使用

- 当tile中的物品栏发生变化时, 手动调用`inventoryChanged`函数
	- 该函数先调用 `super.setChanged()` , 将脏数据存储到磁盘中
	- 再 `level.sendBlockUpdated` , 将服务端改动同步到客户端
	- `sendBlockUpdated` 函数会触发同步流程, 向客户端发送数据
- `sendBlockUpdated` 会调用 `getUpdatePacket` 函数, 使其向客户端发送一个同步数据包
	- `getUpdatePacket` 中会调用 `getUpdateTag` , 将 tile 中的数据打包为 nbt
- 客户端收到 `getUpdatePacket` 发送的数据包, 会将其同步到本地
- 自此,  通过调用`inventoryChanged`完成了tile变化时服务端与客户端的同步






