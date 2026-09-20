# 前言

在完成 LeetCode Hot 100 后，本文结合本人遇到的算法题中的实际使用场景，对 Java 中常用的容器类进行系统整理，并与 C++ 中对应的常见容器进行对比，以帮助在两种语言之间切换的读者更快完成选型。

## 基础类与包装类

注意，Java 中的泛型是基于对象实现的，所以泛型容器内只能存储类对象，而对于基础类型，应该在容器类泛型中使用其对应的包装类。部分基础类到包装类之间对应关系如下表所示：

| 基础类  | 包装类    |
| ------- | --------- |
| byte    | Byte      |
| short   | Short     |
| int     | Integer   |
| long    | Long      |
| float   | Float     |
| double  | Double    |
| char    | Character |
| boolean | Boolean   |

## 获取长度

在 Java 中，不同的场景下获取其长度的方式是不同的，使用时需要注意不要搞混。通常来说遵循以下规律：

| 类别                  | 对应方式      |
| --------------------- | ------------- |
| 数组 Array (原生数组) | arr.length;   |
| 字符串 String         | str.length(); |
| 集合 List/Set/Map/... | list.size();  |

# String 类

## String

在 Java 中，String 字符串类是一个被提前封装的类，而非基础类型。并且，Java 中的 String 是不可变类，也就是说，String 中的内容一旦创建不可被修改。String 可以使用 `+` 号进行字符串拼接，其拼接原理是使用左右两侧的字符串创建了一个新字符串，而不是在原字符串上进行操作，性能相对较差。如果需要频繁进行字符串拼接等操作，请使用 `StringBuilder` 或 `StringBuffer`。

需要注意的是，由于 Java 中的 String 类使用了常量池等技术优化处理速度，导致对同样的一串字符串，直接使用 `==` 比较其引用，有时为 `true` 有时为 `false`，所以比较时简易通通使用 `equals()` 方法进行比较，避免意外情况的出现。

### 常用API

| 操作名     | 对应方法         |
| ---------- | ---------------- |
| 获取长度   | `length()`       |
| 访问字符   | `charAt(index)`  |
| 获取字串   | `substring(l,r)` |
| 转字符数组 | `toCharArray()`  |

### 与C++比较

|              | Java String          | C++ String        |
| ------------ | -------------------- | ----------------- |
| 可变性       | 不可变               | 可变              |
| 拼接能力     | 差（通常借助其他类） | 好                |
| 字符访问方法 | `charAt()`           | 直接使用索引 `[]` |

## StringBuilder和StringBuffer

与 C++ 中直接拼接的方式不同，在 Java 中，如果需要实现字符串拼接，通常需要借助 StringBuilder 或 StringBuffer。

StringBuilder 和 StringBuffer 实现的功能是一样的，都是为了解决 String 频繁拼接、修改场景下的性能底下的问题。二者区别在于，StringBuilder 是非线程安全的，而 StringBuffer 是线程安全的。使用时，StringBuffer 由于需要保证线程安全，会使用同步锁等方式保证多线程环境下的数据一致性，因此速度相对 StringBuilder 较慢。**在刷算法题中，建议使用 StringBuilder 而不是 StringBuffer**。

### 常用API

| 操作名                    | 对应方法               |
| ------------------------- | ---------------------- |
| 追加（字符串、字符）      | `append(ch/str);`      |
| 获取长度                  | `length();`            |
| 反转字符串                | `reverse();`           |
| 转为 String               | `toString();`          |
| 删除指定位置字符          | `deleteCharAt(index);` |
| 删除区间字符`[start,end)` | `delete(start, end);`  |

# Map 接口

在 Java 中，Map 并不作为具体的实现，而是作为一种接口约束，指导不同的 Map 类型所需要实现的共性接口。在 Map 接口的实现类中，最常使用的主要有 HashMap、TreeMap、LinkedHashMap 三类。

## HashMap

HashMap 底层使用哈希表的方式实现了底层的键-值映射关系。由于 HashMap 使用了哈希映射的方式定位键的位置，其查找、插入、删除的时间复杂度是接近O(1)的。但 HashMap 是无序的，不保证遍历的顺序性。

### 常用API



### 与C++比较

Java 中的 HashMap 可以与 C++ 中的 unordered_map 对应理解。

|      |      |      |
| ---- | ---- | ---- |
|      |      |      |
|      |      |      |
|      |      |      |



## TreeMap



### 与C++比较

Java 中的 TreeMap 可以与 C++ 中的 map 对应理解。

## LinkedHashMap

