
Line39: `@SuppressWarnings("deprecation")` 
- 是一个注解, 表示代码中存在已过时的内容且我已知晓, 在接下来的输出中不要再提示我相关内容 (deprecation : 弃用, suppress : 抑制) . 通过此方法, 可以保持提示信息的整洁
- `@SuppressWarnings` 注解是一个用来抑制警告的注解, 可以用在`类, 方法, 局部变量` 中

Line40: `public class CuttingBoardBlock extends HorizontalBlock implements IWaterLoggable`
- `HorizonBlock` : 表示该方块就有水平方块的属性(在水平范围旋转), 在被放置时将根据用户的视角自动处理
- IWaterLoggable: 支持水流交互逻辑, 即方块可以被水穿过, 同时又可以与水共存 (即: 可以成为一个含水方块)

Line42: 表示该方块是 `可含水的`

### 方块属性

Line43: `protected static final VoxelShape SHAPE = Block.box(1.0D, 0.0D, 1.0D, 15.0D, 1.0D, 15.0D);`
- 定义该方块的碰撞体积, 是一个由 `x1, y1, z1` 到 `x2, y2, z2` 构成的扁平长方体

##### 构造函数
Line46: 
- 定义`CuttingBoardBlock`的材质、强度等
Line47:
```java
this.registerDefaultState(
	this.getStateDefinition()
	.any()
	.setValue(FACING, Direction.NORTH)
	.setValue(WATERLOGGED, false)
);
```
- 注册方块的**默认状态**, 分别是**朝向**和**是否含水**
- `getStateDefinition().any()` : 获得方块的所有可能状态组合, `any()`返回默认空白状态
- `registerDefaultState()` : 将生成的状态作为方块的默认状态

### 粒子效果

##### spawnCuttingParticle函数
Line50-59: `spawnCuttingParticles`
- 该函数的意图: 生成飞溅的粒子效果
- 参数: `World`, `BlockPos`, `ItemStack`, `count` -> `count` : 生成粒子的数量
- `Vector3d` : 3D矢量类型, 控制粒子飞溅的方向
- `new ItemParticleData(ParticleTypes.ITEM, stack)` : 生成**物品粒子**的相关数据, 粒子效果由stack物品决定
- 服务端与客户端的区分:
- 目的 : 对于使用菜板的用户, 直接在客户端生成粒子; 对于其他联机用户, 服务端发送给其他人粒子信息
- 服务端 => 在特定的位置以某种方向生成1个速度为0的粒子, 循环count次
	- 对于 `ItemParticle`, 由于已经设置了vector(初速度的方向和大小), 所以speed为0也无所谓
```java
((ServerWorld) worldIn).sendParticles(  
      new ItemParticleData(ParticleTypes.ITEM, stack),  
      pos.getX() + 0.5F,  
      pos.getY() + 0.1F,  
      pos.getZ() + 0.5F,  
      1, 
      vec3d.x,  
      vec3d.y + 0.05D,  
      vec3d.z,  
      0.0D  
);
```
- 客户端 => 在特定的位置以某种方向生成1个粒子, 循环count次
```java
worldIn.addParticle(  
      new ItemParticleData(ParticleTypes.ITEM, stack),  
      pos.getX() + 0.5F,  
      pos.getY() + 0.1F,  
      pos.getZ() + 0.5F,  
      vec3d.x,  
      vec3d.y + 0.05D,  
      vec3d.z  
);
```


### 方块形状

Line80, 85: 返回该方块的形状与碰撞体积

### use函数
use的逻辑如下: 
- 如果切菜板为空, 则试图放入
	- 如果副手不为空, 则处理副手
		- 如果当前为主手, 主手不持有方块且副手不持有装备, 则PASS
			- PASS不一定到副手, 也有可能触发物品的`向容器中放入`的方法
		- 若PASS到了副手, 副手此时持有装备, 则PASS
			- 主手放不进去, 副手也不能放进去, 则不处理
	- 如果最终被用来处理切菜板的手为空手, 那么说明也不做
	- 否则, 试图将物品放入, 且返回`SUCCESS`
- 切菜板不为空时, 如果手中有物品, 则试图对切菜板中物品处理
	- 如果当前物品可以处理板中物品, 则返回`SUCCESS`, 否则`CONSUME`
- 切菜板不为空且为主手空手点击, 则试图移出切菜板内物品
	- 如果玩家不为创造模式, 那么尝试将物品加入背包
		- 如果加入背包失败, 那么将其掉落在世界中
	- 否则, 直接从切菜板中移除物品
	- 移除时, 播放对应声音

Line90: use
- use函数默认为方块的右键行为, 当手持物品右键方块时, 会优先调用方块的右键效果 (比如拿着食物右键菜板会把食物放入菜板而不是吃食物)
- 参数: `BlockState`, `World`, `BlockPos`, `PlayerEntity`, `Hand`, `BlockRayTraceResult` 
- `BlockRayTraceResult`是用于表示 射线检测与方块交互结果 的类, 当用户通过射线与方块交互时, 通过该类返回详细的信息

Line91: `TileEntity tileEntity = worldIn.getBlockEntity(pos);`
- 从世界指定位置获取该方块的 `tileEntity` 
- `tileEntity` , 即方块实体, 为附加在方块上的数据对象, 用于储存方块信息或者复杂逻辑
Line92-93: 如果该位置是切菜板的方块实体, 则通过强转获取切菜板的方块实体

##### 交互优先级
Line94-95: 这段代码用于处理右键切菜板时的交互优先级
- `player.getItemInHand(hand)` : 获取`hand`中的物品, 哪只手右键方块, 就获取那只手手中的物品
- `player.getOffhandItem()` : 获取副手手持的物品
Line99: 该段代码会由客户端和服务端分别执行一遍
```java
if (!offhandStack.isEmpty()) {  
	if (
		handIn.equals(Hand.MAIN_HAND) && 
		!ModTags.OFFHAND_EQUIPMENT.contains(offhandStack.getItem()) && 
		!(heldStack.getItem() instanceof BlockItem)) 	{  
		// Pass to off-hand if that item is placeable  
		return ActionResultType.PASS; 
  }  
  if (
	  handIn.equals(Hand.OFF_HAND) && 
	  ModTags.OFFHAND_EQUIPMENT.contains(offhandStack.getItem())) {  
      // Items in this tag should not be placed from the off-hand  
      return ActionResultType.PASS; 
   }  
}
```
- 当 当前副手物品不可使用 且 主手物品不属于BlockItem, 主手被PASS, 执行其他逻辑
- Forge中, 处理逻辑如下: 
	- 右键时, 由主手执行`Block#use`, 如果被`PASS`会接着执行其他逻辑
	- 如果其他逻辑被触发了, 就不会再由副手被触发
	- 如果最终轮到副手了, 当副手不为装备时, 将副手物品放入

##### 物品加入
Line106: 当前手如果是空手, 那么直接PASS
Line109: 
```java
else if (cuttingBoardEntity.addItem(player.abilities.instabuild ? heldStack.copy() : heldStack)) {  
    return ActionResultType.SUCCESS;  
}
```
- 尝试向切菜板方块实体中添加物品
- `player.abilities.instabuild` : 表示玩家是否有无限物品的能力 (比如创造模式)
-  `? heldStack.copy() : heldStack` : 如果是创造模式, 将当前物品堆复制一份传入, 从而不减少创造模式下物品的数量

##### 物品使用
Line112: 
```java
if (cuttingBoardEntity.processStoredItemUsingTool(heldStack, player)) {  
    spawnCuttingParticles(worldIn, pos, boardStack, 5);  
    return ActionResultType.SUCCESS;  
}
```
- 使用 `cuttingBoardEntity.processStoredItemUsingTool(heldStack, player)` 函数判断该工具是否可以用作切菜板, 如果可以, 则处理切菜板上物品并返回true, 否则返回false
- 该函数内部使用 `hurtAndBreak` 减少工具的耐久, 当创造模式时, 可以自动的不扣除工具的耐久
- 当成功使用工具后, 在切菜板上方, 创建切菜板内物品材质的喷射粒子效果
- 返回 `SUCCESS` , 表示执行成功, 游戏执行后续动画等, 且不继续向下传递行为

##### 移除物品
Line120:
```java
else if (handIn.equals(Hand.MAIN_HAND)) {  
    if (!player.isCreative()) {  
        if (!player.inventory.add(cuttingBoardEntity.removeItem())) {  
            InventoryHelper.dropItemStack(worldIn, pos.getX(), pos.getY(), pos.getZ(), cuttingBoardEntity.removeItem());  
        }  
    } else {  
        cuttingBoardEntity.removeItem();  
    }  
    worldIn.playSound(null, pos.getX(), pos.getY(), pos.getZ(), SoundEvents.WOOD_HIT, SoundCategory.BLOCKS, 0.25F, 0.5F);  
    return ActionResultType.SUCCESS;  
}
```
- 当切菜板不为空且当前为空手时, 则试图移除切菜板上的物品
- `player.isCreative()` : 判断是否为创造模式
- `cuttingBoardEntity.removeItem()` : 从对应方块实体中移除物品, 返回值为对应物品
- `player.inventory.add(item)` : 尝试向玩家背包中加入物品, 返回值为是否加入成功
- `InventoryHelper` : 是一个Mojang提供的物品相关工具类, 可以用于在世界中生成掉落物, 以及清空物品背包掉落到世界中等
	- `InventoryHelper.dropItemStack` : 将物品堆栈掉落到世界中
- 可能存在的问题: 空手右键时, 如果背包中没有空间, `removeItem()` 会在判断条件中执行一次, 在条件语句中再执行一次
	- 但是很难出现主手空手但背包中没有空间的情况


### onRemove
该函数为方块被破坏时的情况
- 什么情况下会触发`onRemove`
	- 玩家用破坏该方块或用其他方块替换此方块
	- 方块被活塞或其他方块推走(此时isMoving参数为true)
	- 自身调用 `setBlockState` 更新属性时 (如旋转, 激活等)

Line140: `public void onRemove(BlockState state, World worldIn, BlockPos pos, BlockState newState, boolean isMoving)`
- state : 原方块状态
- newState : 新方块状态
- isMoving : 是否由活塞推动引起
Line141 : ` if (state.getBlock() != newState.getBlock()) `
- 如果 `onRemove` 后新旧状态是同一个方块, 说明是状态的更新, 不应清除方块
Line143:
```java
if (tileEntity instanceof CuttingBoardTileEntity) {  
    InventoryHelper.dropItemStack(
	    worldIn, 
	    pos.getX(), 
	    pos.getY(), 
	    pos.getZ(), 
	    ((CuttingBoardTileEntity) tileEntity).getStoredItem()
	  );  
    worldIn.updateNeighbourForOutputSignal(pos, this);  
}
```
- `InventoryHelper.dropItemStack` : 在世界中对应坐标掉落切菜板中的物品
- `updateNeighbourForOutputSignal` : 提醒周围红石相关方块, 该方块即将发生变化, 请做好更新准备, 以免出现延迟
- 对于能和红石交互的方块 (红石比较器等) , 这个流程可以作为一个模板
Line148: 调用父类`onRemove`


### 设置方块属性

##### 玩家重生
Line153: `isPossibleToRespawnInThis`
- 是否能出生在该方块内部
- 用以获取玩家重生定位

##### 方块放置
- 获取当前位置的流体信息
- 设置方块状态
	- 朝向
	- 是否包含水
Line 158: 
```java
@Override
public BlockState getStateForPlacement(BlockItemUseContext context) {  
    FluidState fluid = 
	    context
		    .getLevel()
		    .getFluidState(context.getClickedPos());  
    return 
	    this.defaultBlockState()
			    .setValue(
				    FACING, 
				    context.getHorizontalDirection().getOpposite()
				  )  
          .setValue(
	          WATERLOGGED, 
	          fluid.getType() == Fluids.WATER
	        );  
}
```
- 用于覆盖可放置方块的原`getStateForPlacement`方法, 用于获取玩家放置时方块的状态。
- `context.getLevel().getFluidState(context.getClickedPos())` : 获取当前世界点击位置坐标的流体信息, 因为切肉板是可浸水方块
- 设置`FACING`属性, 即方块的朝向
	- `context.getHorizontalDirection()` : 玩家面对的方向(水平方向)
	- `.getOpposite()` : 为使方块面对玩家, 方块的面向应该是玩家面向的反向
	- 可作为有方向方块的模板使用
- 设置`WATERLOGGED`属性, 即方块是否浸水
	- 如果这个方块所处位置的液体为水, 则设置为浸水方块

##### 形状更新
**updateShape函数**
- 检测相邻方块发生变化时, 当前方块是否要更新状态
- 处理水浸方块中水的更新逻辑
- 检查方块是否能存活, 否则替换为空气
Line165: `BlockState updateShape(BlockState stateIn, Direction facing, BlockState facingState, IWorld worldIn, BlockPos currentPos, BlockPos facingPos)`
- `stateIn` : 当前方块状态
- `facing` : 面对方向
- `facingState` : 面对方块的状态
- `worldIn` : 所处世界
- `currentPos` : 当前方块位置
- `facingPos` : 面对方块的位置
Line166:
```java
if (stateIn.getValue(WATERLOGGED)) {  
    worldIn
	    .getLiquidTicks()
	    .scheduleTick(currentPos, Fluids.WATER, 
								    Fluids.WATER.getTickDelay(worldIn)
								    ); 
}
```
- 决定是否更新液体状态, 如果该方块为浸水方块, 那么更新液体状态
	- `worldIn.getLiquidTicks()` : 获取当时世界的流体更新队列
		- `.scheduleTick()` :  安排一个流体更新任务
	- `Fluids.WATER.getTickDelay(worldIn)` : 获取当前世界中指定液体(在这里是`WATER`的流动延迟)
Line169:
```java
return facing == 
		Direction.DOWN && 
		!stateIn.canSurvive(worldIn, currentPos)
		? Blocks.AIR.defaultBlockState()  
		: super.updateShape(stateIn, facing, facingState, worldIn, currentPos, facingPos);
```
- 返回是否更新方块状态
	- 如果下方方块状态发生变化且切菜板不可放在当前位置
		- 返回`Blocks.AIR`的`BlockState`
		- 否则, 调用父类的`updateShape`函数
- 返回`Blocks.AIR`时, 会将该方块替换为空气方块, 并由系统处理产生相应的掉落物
	- 由于使用了`loot_table`重置了方块掉落的战利品, 所以在代码中看不出修改掉落物的部分。
**对应loot_table**
```json
{  
  "type": "minecraft:block",  
  "pools": [  
    {  
      "name": "pool1",  
      "rolls": 1,  
      "entries": [  
        {  
          "type": "minecraft:item",  
          "name": "farmersdelight:cutting_board"  
        }  
      ],  
      "conditions": [  
        {  
          "condition": "minecraft:survives_explosion"  
        }  
      ]  
    }  
  ]  
}
```
- 该json表明, 当方块不是被爆炸摧毁时, 掉落对应的方块

##### 可存活
- 该函数用来判断某方块能否在某个位置"生存"
Line175:
`canSurvive(BlockState state, IWorldReader worldIn, BlockPos pos)`
- state : 方块状态
- worldIn : 世界状态
- pos : 试图放置方块的位置
Line176:
```java
BlockPos floorPos = pos.below();  
return 
	canSupportRigidBlock(worldIn, floorPos) || 
	canSupportCenter(worldIn, floorPos, Direction.UP);
```
- `pos.below();` : 获取当前方块位置下方的位置
- `canSupportRigidBlock` : 可以完整支持上方的方块
	- 比如石头, 原木等
- `canSupportCenter` : 可以支持中心位置的方块
	- `Direction.UP` : 检查该方块上方是否可以中心支撑对应方块
	- 比如栅栏, 铁栅栏等

##### 创建方块状态
Line192:
- 向方块状态中加入`FACING` , `WATERLOGGED` 两项
	- `FACING` 为父类中的状态
	- `WATERLOGGED` 为自己创建的状态
```java
protected void createBlockStateDefinition(final StateContainer.Builder<Block, BlockState> builder) {  
    super.createBlockStateDefinition(builder);  // 如果父类加了属性，这一步是必须的
    builder.add(FACING, WATERLOGGED);  
}
```


##### 检测流体
Line198: 
```java
@Override  
public FluidState getFluidState(BlockState state) {  
    return state.getValue(WATERLOGGED) ? 
    Fluids.WATER.getSource(false) : 
    super.getFluidState(state);  
}
```
- `Fluids.WATER.getSource(false)` : 返回一个不在下落的水源
	- `getSource(isFalling)` : 返回液体状态, 参数代表是否在下落
- super.getFluidState(state) : 调用父类的获取流体的逻辑, 通常是`FluidState.EMPTY`


##### 红石比较器
Line203: `hasAnalogOutputSignal`
- 是否可以通过红石比较器输出信号
Line208:
```java
public int getAnalogOutputSignal(BlockState blockState, World worldIn, BlockPos pos) {  
    TileEntity tileEntity = worldIn.getBlockEntity(pos);  
    if (tileEntity instanceof CuttingBoardTileEntity) {  
        return 
        !((CuttingBoardTileEntity) tileEntity).isEmpty() ? 15 : 0;  
    }  
    return 0;  
}
```
- 切菜板中有东西, 红石强度为15, 否则就为0

##### TileEntity
Line217: 返回`true`, 说明该方法存在`TileEntity`
Line223: 给予系统该方块的`TileEntity`
```java
public TileEntity createTileEntity(BlockState state, IBlockReader world) {  
    return ModTileEntityTypes.CUTTING_BOARD_TILE.get().create();  
}
```
- 返回`CUTTING_BOARD_TILE`

### 潜行工具
- 对于切菜板, 潜行时, 可以将手中的工具放入到切菜板中
- 对于潜行状态的判定, 是通过监听器来实现的

Line227:
- `Mod.EventBusSubscriber` : 使该函数成为事件监听器
- `modid = FarmersDelight.MODID` : 该监听器所属的mod
- `bus = Mod.EventBusSubscriber.Bus.FORGE` : 该监听器挂载到Forge总线中
**作用**


Line228: `public static class ToolCarvingEvent` : 方块类中定义的内置类, 用于声明放置物品事件
Line231:
```java
@SubscribeEvent  
@SuppressWarnings("unused")  
public static void onSneakPlaceTool(PlayerInteractEvent.RightClickBlock event) {
```
- `@SubscribeEvent` : 设置该函数为监听事件
	- `PlayerInteractEvent.RightClickBlock` : 监听的是玩家在方块上右键点击时触发的事件
Line238:
```java
if (player.isSecondaryUseActive() && 
		!heldStack.isEmpty() && 
		tileEntity instanceof CuttingBoardTileEntity)
```
- `player.isSecondaryUseActive` : 是否正在进行辅助使用操作
- `!heldStack.isEmpty()` : 手中持有物品
- `tileEntity instanceof CuttingBoardTileEntity` : 当前位置的实体属于切菜板实体
Line239:
```java
if (heldStack.getItem() instanceof TieredItem ||  
        heldStack.getItem() instanceof TridentItem ||  
        heldStack.getItem() instanceof ShearsItem)
```
- 手中物品属于 以上三类时, 继续逻辑
Line242:
```java
boolean success = ((CuttingBoardTileEntity) tileEntity)
	.carveToolOnBoard(
		player.abilities.instabuild ? heldStack.copy() : heldStack
	);
```
- 调用`CuttingBoardTileEntity`的`carveToolOnBoard`函数, 将工具嵌到切菜板上
- 返回嵌入是否成功
Line244:
```java
world.playSound(
	null, pos.getX(), pos.getY(), pos.getZ(), 
	SoundEvents.WOOD_PLACE, SoundCategory.BLOCKS, 1.0F, 0.8F);  
event.setCanceled(true);  
event.setCancellationResult(ActionResultType.SUCCESS);
```
- 播放音乐
- 对于原事件(即鼠标右键该方块的事件), 阻止该事件原本的逻辑
	- 如果潜行时右击切菜板, 会阻止普通右键切菜板的逻辑
- 对于被阻止的事件, 设置其返回值为`SUCCESS`, 表示被成功处理


