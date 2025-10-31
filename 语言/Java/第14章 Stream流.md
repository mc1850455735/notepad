
**作用与使用**
- 结合Lambda表达式, 简化集合, 数组操作
- 使用步骤
	- 先得到一个Stream流, 并把数据放上去
	- 利用中间方法对流水线上的数据进行操作
	- 使用终结方法对流水线上的数据进行操作

# 获取Stream流
![[语言/Java/Inbox/Pasted image 20251017170440.png]]

**双列集合获取Stream流**
```java
hm.entrySet().stream().forEach(entry -> {
    System.out.println(entry.getKey() + " " + entry.getValue());
});
```

**零散数据整合成为Stream流**
- 要求这些零散数据的类型必须相同, 如果要传入数组作为可变参的数据, 则要求该数组必须是**引用数据类型**, 否则会把整个数组作为一个元素传入
```java
int a = 3, b = 5, c = 3;
Stream.of(a,b,c).forEach(System.out::println);
```

# 中间方法

**概述**
- 中间方法会返回新的Stream流, 原来的Stream流只能使用一次, 用完一次后该流就会关闭, 所以推荐使用**链式编程**
- 修改Stream流中的数据, 不会影响原来集合或数组中的数据
![[语言/Java/Inbox/Pasted image 20251017171644.png]]

**去重**
- 如果想要实现自定义类去重, 则自定义方法必须重写hashCode和equals方法
- 去重的逻辑是基于HashSet实现的, 所以需要重写这两个方法

**合并流**
- 当两个要合并的流类型不一致时, 会向他们的共同父类进行强制转换

**类型转换**
- 第一个参数类型: 流 中原本的数据类型
- 第二个参数类型: 要转成之后的数据类型
```java
list.stream().map(s -> {
    String[] arr = s.split("-");
    String intStr = arr[1];
    return Integer.parseInt(intStr);
}).forEach(System.out::println);
```

# 终结方法
![[语言/Java/Inbox/Pasted image 20251017175220.png]]

**toArray**
- 默认不加参数的情况, 返回值为Object类型的数组
- 传参情况下, 该函数的参数为Lambda表达式, 负责创建一个指定类型的数组
- Lambda表达式的参数接收当前流内元素的数量
- toArray方法的底层, 会依次得到流里面的每一个数据, 并将其加入数组中
- toArray方法的返回值是一个装着流里面所有数据的数组
```java
String[] array = list.stream().toArray(value -> new String[value]);
```

**collect**
- 通过 collect() 方法, 可以将流中的数据收集到 `List, Set, Map` 中

**收集到List**
```java
List<String> list1 = list.stream()
    .filter(s -> "男".equals(s.split("-")[1]))
    .collect(Collectors.toList());
```

**收集到Set**
- 收集到Set的过程中会实现去重
```java
Set<String> set1 = list.stream()
    .filter(s -> "男".equals(s.split("-")[1]))
    .collect(Collectors.toSet());
```

**收集到Map**
- 收集到Map时, 需要在 Collectors.toMap 的参数中指定键和值
- 如果想要收集到Map中, 则**键不能重复**, 否则代码会出现报错
- 需要第一个参数表示键的生成规则, 第二个参数表示值的生成规则, 都需要传入Lambda表达式
```java
Map<String, Integer> map1 = list.stream()
    .filter(s -> "男".equals(s.split("-")[1]))
    .collect(
        Collectors.toMap(
            s -> s.split("-")[0],
            s -> Integer.parseInt(s.split("-")[2]))
    );
```

# 方法引用

