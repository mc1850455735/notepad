## Math类

里面的所有方法都是静态方法, Math类是一个工具类

- int abs(int a) - 取绝对值
- double ceil(double a) - 向上取整
- double floor(double a) - 向下取整
- int round(float a) - 四舍五入
- int max(int a, int b) - 获取两个int值中的较大值
- double pow(double a, double b) - 返回a的b次方幂
- double random() - 返回double类型随机值, 范围为 \[ 0.0, 1.0 \)

#### abs() 
- 对数值取绝对值

###### absExact() 
- 对数值取绝对值, 对于过大的数, 报错

#### ceil()
- 向上取整(向数轴正方向进一)

#### floor() 
-  向下取整(往数轴负方向进一)

## System类

也是一个工具类，提供了一些与系统有关的方法。

|函数名|功能|
|---|---|
|`public static void exit(int status)`|终止当前运行的Java虚拟机|
|`public static long currentTimeMillis()`|返回当前系统的时间**毫秒值**形式|
|`public static void arraycopy(数据源数组, 起始索引, 目的地数组, 起始索引, 拷贝个数)`|数组拷贝|

计算机中的时间原点
1970年1月1日 00:00  (C语言生日)
北京时间原点 1970年1月1日 08:00 (东八区)

#### System.currentTimeMillis()

可以通过计算两次函数调用返回值的差值来获得语句执行的时间

#### System.arraycopy()

- 细节1: 如果数据源数组和目的地数组都是基本数据类型, 那么两者数据类型必须保持一致, 否则就会报错
- 细节2: 在拷贝的时候需要考虑数组的长度, 如果超出数组的长度也会报错
- 细节3: 如果数据源数组和目的地数组都是引用数据类型, 那么子类类型可以赋值给父类类型, 引用数据类型拷贝的是地址值

## Runtime类

表示当前虚拟机的运行环境，这个类里面的方法**不是静态的**

|方法名|说明|
|-|-|
|`public static Runtime getRuntime()`|获取系统的运行环境对象|
|`public void exit(int status)`|停止虚拟机|
|`public int availableProcessors()`|获取cpu线程数|
|`public long maxMemory()`|虚拟机JVM能从内存中获得的总大小(单位byte)|
|`public long totalMemory()`|虚拟机JVM已经获得的内存总大小(单位byte)|
|`public long freeMemory()`|虚拟机JVM还剩余的总内存大小(单位byte)|
|`public Process exec(String command)`|运行cmd命令|

Runtime的构造方法**被私有化**了；
- 通过私有static final变量创建了一个对象，命名为currentRuntime，代表着变量记录着的地址值是不能变化的。
- 通过public方法getRuntime返回currentRuntime来获取Runtime对象。
- 通过这个方法，使不管哪个类中，都只有同一个Runtime对象。
- Runtime代表着虚拟机的运行环境，在同一台电脑中只能有一个运行环境，所以Runtime只能有一个对象，代表当前虚拟机的运行环境。

## Object和Objects

### Object

- Object是Java中的顶级父类。所有的类都直接或间接的继承自Object类
- Object类中的方法可以被所有子类访问，所以我们要学习Object类和其中的方法。

**构造方法:** 
(所有类中都有一个隐藏的super()无参构造, 这个构造函数就是Object的空参构造)

|方法名|说明|
|-|-|
|`public Object()`|空参构造(没有一个属性是所有类的共性,所以是空参构造)|

**成员方法**:

|方法名|说明|
|-|-|
|`public String toString()`|返回对象的字符串表示形式|
|`public boolean equals(Object obj)`|比较两个对象是否相等|
|`protected Object clone(int a)`|对象克隆|

- Object中的toString方法将会返回对象地址值的字符串格式
- Object中的equals方法比较两个对象的地址值

**clone方法**
为了实现clone方法，需要重载clone方法，clone方法中的克隆默认是浅克隆
同时需要实现Cloneable接口
Cloneable接口表示一旦实现当前类的对象就可以被克隆
方法会帮我们创建一个对象，并把原对象中的代码拷贝过去。
如果想要让clone实现深克隆，需要重写clone方法
在重写过程中，可以调用父类中的clone方法来复制值类型的数据成员。

**clone实现细节**
1. 重写Object中的clone方法
2. 让Javabean类实现Cloneable接口
3. 创建原对象并调用clone方法

如果一个接口没有抽象方法
表示当前的接口是一个**标记性接口**

浅克隆和深克隆
**浅克隆**
对于基本数据类型，直接复制基本数据类型的值；对于引用数据类型，复制引用数据类型的地址。
**深克隆**
对于基本数据类型，同样直接复制基本数据类型的值；对于引用数据类型，会创建一个新的对象，并将原数据中对象的值给新的对象。
String类型虽然是引用数据类型，但是编译器会复用串池中已经存在的字符串。

第三方深克隆工具
1、导入第三方代码
2、编写代码, 把对象变成一个字符串
3、把字符串变为对象
4、打印对象

### Objects

|方法名|说明|
|-|-|
|public static boolean equals(Object a, Object b)|先做非空判断,比较两个对象|
|public static boolean isNull(Object obj)|判断对象是否为null,为null返回true,反之为true|
|public static boolean nonNull(Object obj)|判断对象是否为null后,与isNull相反|

equals方法
- 先做非空判断，如果不相同且不为null，再调用Object的equals方法

## BigInterger和BigDecimal

### BigInteger

##### BigInteger的初始化
java中，有四种整数类型：byte, short, int, long

如果要表达的整数的范围超出了long能表示的范围, 则应使用BigInteger类
BigInteger有上限, 但是上限非常大, 可以近似看作没有上限.

|构造方法名|说明|
|-|-|
|public BigInteger(int num, Random rnd)|获取随机大整数,范围`[0~2的num次方-1]`|
|public BigInteger(String val)|通过字符串获取指定的大整数|
|public BigInteger(String val, int radix)|获取指定进制的大整数|

|方法名|说明|
|-|-|
|public static BigInteger valueOf(long val)|静态方法获取BigInteger的对象,内部有优化|

**对象一旦创建, 内部记录的值不能发生变化**

在BigInteger内部, 对常用的数字(-16 ~ 16)进行了优化, 提前把 -16 ~ 16 先创建好BigInteger的对象, 如果多次获取, 则不会创建新的对象, 实现了优化

![[语言/Java/Inbox/Pasted image 20230712145608.png]]

##### BigInteger的常见方法
![[语言/Java/Inbox/Pasted image 20230712150713.png]]

















