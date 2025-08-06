### 概述
Advancement, 即进度, 是用来追踪玩家成就, 引导玩家进度的组件. 类似于原版中的成就系统

**作用**
- 任务引导
- 奖励机制
- 条件触发
- 可视化提醒

**定义**
- 通常使用JSON文件, 在`data/[modid]/advancement`路径下定义
- 可以通过`requirements`或`parent`字段关联其他进度，形成依赖关系
- 可以使用代码(如`AdvancementProvider`)动态生成进度

**区别**
1. 可以添加全新的进度树
2. 可以设定独特的进度达成条件

### 源码

##### CuttingBoardTrigger

- 继承自 `AbstractCriterionTrigger` , 这个类是所有自定义触发器的基类(Criterion: 标准)
	- 该类的模板参数要传入的是 `CuttingBoardTrigger` 的内部类 `Instance`
	- `Instance` 继承自 `CriterionInstance` , 
- `ResourceLocation ID` 是触发器的唯一标识符

**重点**
核心方法为 trigger, 

