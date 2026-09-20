
# 异常分类

在 Java 中，所有的异常都继承自 `java.lang.Throwable` 类

## Error 与 Exception

### Error（错误）

通常是 **JVM 无法处理的严重问题**。
- 例如：`StackOverflowError`（栈溢出）、`OutOfMemoryError`（内存溢出）。
- **策略**：程序通常无法从 Error 中恢复，不建议用 `try-catch` 去捕获，只能通过优化系统配置或代码逻辑来解决。

### Exception（异常）

程序本身可以处理的异常。分为 **受检异常**和**非受检异常**。

## 受检异常 与 非受检异常

### 受检异常 (Checked Exception)

- **定义**：除了 `RuntimeException` 及其子类以外的异常。
- **特点**：编译器会**强制要求你处理**。要么 `try-catch` 捕获，要么在方法签名上用 `throws` 抛出。
- **典型例子**：`IOException`、`SQLException`、`ClassNotFoundException`。

### 非受检异常 (Unchecked Exception / RuntimeException)

- **定义**：`RuntimeException` 及其子类。
- **特点**：编译器不会检查。通常是由于代码逻辑漏洞导致的，应该通过编写健壮的代码来规避。
- **典型例子**：`NullPointerException`（空指针）、`ArrayIndexOutOfBoundsException`（数组越界）、`ArithmeticException`（除数为 0）。

## finally 块的执行细节

1. **一定会执行**：不管有没有捕获到异常，`finally` 块都会执行（除非 JVM 退出，如调用了 `System.exit(0)`）。
2. **`return` 的覆盖**：如果 `try` 块中有 `return` 语句，`finally` 也会在 `return` **执行之后、方法返回之前**执行。如果 `finally` 里也有 `return`，它会覆盖 `try` 里的返回值。

## 异常处理的“最佳实践”

- **不要忽略异常**：绝对不要写空的 `catch` 块。如果不知道怎么处理，至少把堆栈打印出来或者抛给上层。
- **尽量捕获具体异常**：不要直接捕获 `Exception`。这会隐藏掉你本应发现的其他潜在问题。
- **使用 try-with-resources**：在处理流（如 `FileInputStream`）时，Java 7 引入的这种语法可以**确保资源自动关闭**，避免手动在 `finally` 里写冗长的关闭代码。



