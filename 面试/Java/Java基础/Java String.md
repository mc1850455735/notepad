
## 为什么 String 不可变

### 原因

String 的底层实现是一个 final 类型 char / byte ( Java 9 之后 )，final 类型不可变。

### 依据

- **字符串常量池（String Pool）的需要**：只有当 String 不可变时，多个变量才能**安全地共享**同一个缓存实例。如果 String 可变，一个变量改了值，其他引用该字符串的变量都会受影响。
- **安全性**：String 经常作为 HashMap 的 **Key**。如果 String 是可变的，其 `hashCode` 就会随内容改变，导致无法在 Map 中找回原来的数据。此外，网络连接参数、文件路径通常也是 String，不可变性能**防止这些关键参数被篡改**。
- **性能优化**：由于 String 不可变，它的 `hash` 值在创建时就可以计算并缓存起来。在集合类中频繁使用时，**效率极高**。

## String 拼接

- 对于两个**字符串常量的拼接**，在编译器阶段就会被编译器直接优化，不会产生多余对象。
- 使用 `+` 进行 String 拼接时，实际上是调用了 StringBuilder，但是每次拼接都会创建一个新的 StringBuilder 对象，所以连续拼接 String 时建议**显式创建** StringBuilder 进行拼接。

## String、StringBuilder、StringBuffer 区别

**可变性**
- **`String`**：底层是用 `final` 修饰的字符数组，Java 9 以后改为 `byte[]` 以节省空间。它是**不可变**的，每次对 String 的修改实际上都是创建了一个新的对象。
- **`StringBuilder` 与 `StringBuffer`**：都是可变字符序列，可以在原有对象基础上进行修改，不会产生大量临时对象。

**线程安全性**
- **`String`**：因为不可变，所以天生线程安全。
- **`StringBuffer`**：对方法加了 `synchronized` 锁，是**线程安全**的，但**性能较低**。
- **`StringBuilder`**：**非线程安全**，**性能最高**。在单线程环境下首选 `StringBuilder`。

