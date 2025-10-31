
# 概述

**概述**
- IO流是一种用于 读取&输出 数据内容的工具

**分类**
- 按流的方向
	- 输入流: 用于读取
	- 输出流: 用于写出
- 按操作文件类型
	- 字节流: 可以操作所有类型文件
	- 字符流: 只能操作纯文本文件

### IO流体系

- 最顶层的IO流都是抽象类
- 字节流
	- `InputStream`
	- `OutputStream`
- 字符流
	- `Reader`
	- `Writer`

### 资源释放

##### 异常处理
- 为确保流对象一定被关闭, 进行异常处理时, 可以将流关闭方法的调用置于finally方法块中
- 通常情况下, 仍将异常作抛出处理, 因为在 Spring 框架中会对执行过程中出现的异常在控制层统一处理
- 被finally控制的语句一定会执行, 除非jvm退出

##### AutoClosable
- 通过实现接口 `AutoClosable` , 可以实现**特定情况**下, 自动释放资源
- **注意**: 只有实现了 `AutoClosable` 的流才能在 `try()` 创建对象
- JDK7 
```java
try(创建流对象1;创建流对象2) {
	可能出现异常代码;
} catch(...) {
    异常处理代码;
}
```
- JDK9
```java
创建流对象1;
创建流对象2;
try(流1;流2) {
	可能出现异常代码;
} catch(...) {
    异常处理代码;
}
```

### 字符集

##### 分类
- **ASCII**: 在ASCII中, 一个字节就可以存储一个英文字符, 共128个
- **GB2312字符集**: 1980年发布, 简体中文汉字编码国标, 共7445个字符, 其中6763个简体汉字, 只包含简体中文
- **BIG5字符集**: 台湾地区繁体中文标准字符集, 共13053个中文
- **GBK字符集**: 2000年发布, 收录21003个汉字, 包含GB13000中的所有中日韩汉字以及BIG5中的所有繁体汉字, Windows中文系统默认使用GBK字符集, 系统显示为ANSI字符集
- **Unicode字符集**: 国际标准字符集, 将世界各种语言的每个字符定义一个唯一编码, 以满足跨语言跨平台的文本信息转换

##### GBK
- 对于英文, GBK完全支持ASCII字符集, 一个英文字母按ASCII使用1字节存储
- 对于中文, 查询GBK后得到2字节数据, 使用2字节进行存储. 
- 为区分中文和英文, 汉字的两个字节, 高位字节的首位一定以1开头; 对于英文, 所有英文字母的编码一定是以0开头

##### Unicode
- Unicode存在多种编码方式
**UTF-16** : 使用2~4个字节保存信息
**UTF-32**: 固定使用4字节保存信息
**UTF-8**: 使用1~4个字节保存信息
- ASCII表中字符: 使用1个字节保存, 以0开头
- 欧美国家文字: 使用2个字节保存, 以11开头
- 中日韩, 东南亚, 中东等: 使用3字节保存, 以111开头
- 其他语言: 4字节保存

# 字节流

### FileOutputStream

**作用**
- 操作本地文件的字节输出流, 可以把程序中的数据保存到本地文件当中
- 通过字节数组, 可以实现一次写多个数据, 且可以控制写入字节数组部分数据

**步骤**
- 创建 FileOutputStream 对象
	- 构造器参数可以是URL字符串路径, 也可以是File文件对象
	- 如果文件不存在, 会创建新文件, 但是需要保证**父级路径**存在
	- 如果文件已经存在, 默认会**清空文件**再写入
	- 通过将append参数设置为 true, 可以实现追加写的效果
- 写数据
	- 写入txt时参数为整数, 但是本地文件实际显示的是该整数对应ASCII字符
- 释放资源
	- 每次使用完流, 都需要释放资源, 相当于解除了对资源的占用

**注意事项**
- 通过调用String的getBytes()方法, 可以直接获得一个字符串的字节数组
- 进行换行操作时, 每个操作系统对应的字符不同
	- windows: `\r\n`
	- linux: `\n`
	- Mac: `\r`
- 在windows操作系统中, java中对回车换行进行了优化, 即使不写完整的`\r\n` , 只写 `\r` 或者 `\n`, Java也会对换行进行补全, 但是**建议写全**

**示例**
- 写入1字节数据
```java
FileOutputStream fos = new FileOutputStream("javase_myio\\a.txt");
fos.write(97);
fos.close();
```
- 写入字节数组
```java
FileOutputStream fos = new FileOutputStream("javase_myio\\a.txt");  
fos.write(new byte[] {97, 98, 99, 100});  
fos.close();
```
- 写入字节数组部分数据
```java
FileOutputStream fos = new FileOutputStream("javase_myio\\a.txt");  
byte[] bytes = {97, 98, 99, 100};  
fos.write(bytes, 1, 3);  
fos.close();
```

### FileInputStream

**作用**
- 操作本地文件的字节输入流, 把本地文件中的数据读取到程序中
- 当文件中存在多个字符时, 使用循环读取的方式获取其中数据

**步骤**
- 创建字节输入流对象
	- 如果文件不存在, 直接报错
- 读数据
	- 使用 `read()` 方法, 一次读取一个字节, 返回值为int
	- 即使写入的数据为负数, 读取时也只会返回该字节的无符号整数int形式
	- 读取时, 向其中传入一个byte数组, 可以实现批量读取字节, 每次读取一个字节数组的数据, 每次会尽可能把数组装满, 返回值表示本次读取到了多少个字节数据
	- 读取到文件末尾, `read()` 方法会返回 -1, 不论单个读取还是批量读取
- 释放资源

**注意**
- `new String(byte[] bs, int start, int len)`, 将字节数组从0开始的len个字符转换成为字符串

**示例**
- 读取单个字符
```java
FileInputStream fis = new FileInputStream("javase_myio\\a.txt");
int b1 = fis.read();
System.out.println(b1);
fis.close();
```
- 循环读取
```java
FileInputStream fis = new FileInputStream("javase_myio\\a.txt");
int b1;
while((b1 = fis.read()) != -1) {
    System.out.print((char)b1);
}
fis.close();
```
- 批量读取
```java
byte[] mybuffer = new byte[8];  
int len = 0;  
while((len = fis.read(mybuffer)) != -1) {  
    System.out.println(new String(mybuffer, 0, len));  
}
```

# 编码与解码

### 乱码

**原因**
- 读取数据时未读完整个汉字
- 编码和解码时的方式不统一

**解决**
1. 不要用字节流读取文本文件
2. 编码解码时使用同一个码表, 同一个编码方式

### 编码
- Idea中默认使用UTF-8进行编码
- Java中, 使用 String 类中的 `getBytes()` 的方式, 可以对字符串进行默认方式的编码, 使用 `getBytes(String charsetName)`, 可以对字符串使用指定方式进行解码
```java
String str = "马金良abc";  
byte[] bytes1 = str.getBytes();    
byte[] bytes2 = str.getBytes("GBK");  
```

### 解码
- 使用String类中的无参构造方法, 可以使用默认方式进行解码; 使用带有指定charset参数的构造方法, 可以使用指定方式进行解码
```java
String result1 = new String(bytes1);   
String result2 = new String(bytes2, "GBK");  
```


# 字符流

**概述**
- 字符流的底层实际就是字节流, 相当于是使用了字符集的字节流
- 读取时, 按照使用的字符集的规则, 一次性读一个或多个字节
- 输出时, 底层会把数据按照指定的编码方式进行编码, 变成字节再写到文件中
- 通常用于对纯文本文件进行读写

### FileReader

##### 步骤
- 创建字符输入流对象
	- 读取文件对象不存在会直接报错
	- 创建对象时, 可以直接在第二个参数指定字符集
		- 底层是使用 `InputStreamReader` 转换流实现的
- 读取数据
	- 按字节进行读取, 遇到中文时, 一次性读多个字节, 读取后解码成为int类型的十进制并返回一个整数, 表示对应字符集上的数字
	- 读到文件末尾 read 方法返回 -1
- 释放资源

##### 示例
- **按单个字符读取**
```java
FileReader fileReader = new FileReader("javase_myio\\a.txt");
int ch;
while ((ch = fileReader.read()) != -1) {
    System.out.println((char) ch);
}
fileReader.close();
```
- **批量读取**
- 换行由两个字符组成, 使用时需要注意
- 有参的 `read(chars)` 方法把读取数据, 解码和强转**三步合并**在一起
```java
FileReader fileReader = new FileReader("javase_myio\\a.txt");
char[] chBuffer = new char[2];
int len;
while ((len = fileReader.read(chBuffer)) != -1) {
    System.out.println(new String(chBuffer, 0, len));
}
fileReader.close();
```

##### 原理
- 创建字符输入流对象后, 关联文件, 在内存中开辟一块长度为8192的缓冲区
	- 只有字符流存在缓冲区, 字节流没有缓冲区
- 每次读取字符时, 都从缓冲区中获取数据, 数据不足时尽可能填充缓冲区
- 缓冲区和文件中都不存在数据, 返回 -1
- 缓冲区获取到数据后即使清空文件, 也至少会保留缓冲区中的数据进行输入

### FileWriter

##### 步骤
- 创建对象
	- 参数是可以是URL路径, 也可以是File类对象
	- 如果文件不存在, 会创建新文件, 但是需要保证**父级路径**存在
	- 如果文件已经存在, 默认会**清空文件**再写入
	- 通过将append参数设置为 true, 可以实现追加写的效果
- 写数据
	- 如果write方法参数为整数, 实际写入文件上的是整数在**对应字符集**上对应的字符
- 释放资源
	- 每次使用完流, 都需要释放资源, 相当于解除了对资源的占用

##### 实例
- 一次写一个字符
```java
FileWriter fileWriter = new FileWriter("javase_myio\\a.txt");
fileWriter.write('你');
fileWriter.close();
```
- 一次写一个字符串
```java
FileWriter fileWriter = new FileWriter("javase_myio\\a.txt");
fileWriter.write("我和你");
fileWriter.close();
```
- 写字符数组
```java
FileWriter fileWriter = new FileWriter("javase_myio\\a.txt");
fileWriter.write(new char[] {'a', '你'});
fileWriter.close();
```

##### 原理
- 创建字符输入流对象后, 关联文件, 在内存中开辟一块长度为8192的缓冲区
- 写出数据前, 将要写出的字符串按照编码规则编码成为多个字节, 并将编码完成后的字符串写入缓冲区
- 当缓冲区装满 或 手动刷新(flush) 或 close时, 缓冲区中的数据加入到文件中

# 缓冲流

- `BufferedInputStream`
- `BufferedOutputStream`
- `BufferedReader`
- `BufferedWriter

### 字节缓冲流

##### 概述
- 原理: 底层自带了长度为8192的缓冲区提高性能
- 缓冲流是一种高级流, 是对基本流做的一种**包装**, 显著提高数据的读写性能
- 缓冲流创建时, 需要关联基本流

##### 实例
- 字节缓冲流每次拷贝一个字节
```java
BufferedInputStream bis = new BufferedInputStream(new FileInputStream("javase_myio\\a.txt"));
BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("javase_myio\\b.txt"));
int b;
while ((b = bis.read()) != -1) {
    bos.write(b);
}
bos.close();
bis.close();
```

- 字节缓冲流每次拷贝一批字节
```java
BufferedInputStream bis = new BufferedInputStream(new FileInputStream("javase_myio\\a.txt"));
BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("javase_myio\\c.txt"));
byte[] buffer = new byte[1024];
int len;
while ((len = bis.read(buffer)) != -1) {
    bos.write(buffer, 0, len);
}
bos.close();
bis.close();
```

##### 原理
- 缓冲流中关联了基本流, 实际从硬盘中读取和写入数据的还是基本流, 基本流完成读取后交给缓冲流, 缓冲流维护一个默认大小为**8192**的缓冲区
- 缓冲输入流和缓冲输出流各会维护一个缓冲区, 二者不是同一个缓冲区
- 同时使用缓冲输入和缓冲输出时, 接收数据的变量会先从**读取缓冲区**中获取数据, 再将数据写入**输出缓冲区**中, 相当于变量只起到中间件的作用
- 由于这个从输入缓冲流到输出缓冲流的过程在内存中进行, 速度相对更快

### 字符缓冲流
- 也对基本流进行了包装
- 在字符缓冲流中, 缓冲区大小不再是8192字节, 而是8192个char字符的大小
- 对字符流提升不明显, 关键点是其两个特有方法

##### 特有方法

**字符缓冲输入流**
- `readLine()` : 读取一行数据, 遇到回车符结束, 如果没有可读数据则返回 **null**
- 不会把回车符读入内存, 读取一行后输出必须手动huan'ha
```java
BufferedReader br = new BufferedReader(new FileReader("javase_myio\\a.txt"));
String s;
while((s = br.readLine()) != null) {
    System.out.println(s);
}
br.close();
```

**字符输出流**
- `newLine()` : 跨平台的换行
```java
BufferedWriter bw = new BufferedWriter(new FileWriter("javase_myio\\b.txt"));
bw.write("你好");
bw.newLine();
bw.write("我是马金良");
bw.close();
```

# 转换流

**概述**
- 是字符流和字节流之间的桥梁
- `InputStreamReader`
- `OutputStreamWriter`

**作用**
- **指定字符集**进行读写, jdk11后使用 FileReader可以直接指定编码方式
- 字节流想要**使用字符流中的方法**

**使用**
- 输入转换流
```java
InputStreamReader isr = new InputStreamReader(new FileInputStream("javase_myio\\gbkfile.txt"), "GBK");  
int ch;  
while((ch = isr.read()) != -1) {  
    System.out.print((char) ch);
}  
isr.close();
```
- 输出转化流
```java
OutputStreamWriter osw = new OutputStreamWriter(new FileOutputStream("javase_myio\\b.txt"), "GBK");  
osw.write("你好你好");  
osw.close();
```

# 序列化与反序列化流

### 概述
- 又称为对象包装输出流, 把基本流包装成高级流
- 可以把Java中的对象写到本地文件中
- 使用序列化流保存对象时, 要求对象必须实现 **Serializable 接口**
- Serializable 接口内部不含有抽象方法, 是一种标记型接口

### 使用
- 写出
```java
Student stu = new Student("zhangsan", 23);  
ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("javase_myio\\a.txt"));  
oos.writeObject(stu);  
oos.close();
```
- 读取
```java
ObjectInputStream ois = new ObjectInputStream(new FileInputStream("javase_myio\\a.txt"));  
Student student = (Student) ois.readObject();  
System.out.println(student);
```

### 注意事项

##### 版本号
- 当一个类实现Serializable时, 会根据类的成员变量, 静态方法, 构造方法等信息, 计算得出long类型的序列号
- 写入时, 会将该long类型序列号同时写入文件中, 读入时按序列号进行校验
- 通过自定义版本号, 可以实现版本号的固定, 版本号是一个 `long` 类型变量, 使用 `private static final` 修饰, 且名称必须为 `serialVersionUID`
- 初始化时, 如果数据中不存在对应某变量的值, 则直接置为默认值
- 通过设置idea, 可以让idea自动生成版本号

##### transient
- 瞬态关键字
- 通过为变量设置该关键字, 可以在序列化过程中跳过该变量


# 打印流

**概述**
- 打印流不能读, 只能写
- 分为字节打印流 `PrintStream` 和字符打印流 `PrintWriter`
- 字节打印流即使开启自动刷新也没有效果, 因为底层不存在缓冲区
- 字符流底层存在缓冲区, 想要自动刷新需要开启
- System 中的 out 静态变量, 实际上就是一个**字节打印流** PrintStream, 该打印流默认指向控制台. 该流的名称即为**标准输出流**, 不能关闭, 在系统中唯一

**特点**
- 打印流只操作文件目的地, 不操作数据源
- 打印流**特有写出方法**可以实现数据的原样写出
- 打印流**特有写出方法**可以实现自动刷新和自动换行: 写出+换行+刷新

**方法**
- `write` : 常规方法, 写出指定字节
- `println` : 打印任意数据, 自动刷新, 自动换行
- `print` : 打印任意数据, 不换行
- `printf` : 带占位符的打印语句, 不换行

# 压缩流与解压缩流

### 解压
- 解压本质: 把每一个 ZipEntry 按照层级拷贝到本地另一个文件夹中
- ZipEntry使用完毕后, 需要进行 `closeEntry` 操作
```java
public static void unzip(File src, File dest) throws IOException {
    ZipInputStream zip = new ZipInputStream(new FileInputStream(src));
    ZipEntry zipEntry;
    while((zipEntry = zip.getNextEntry()) != null) {
        File file = new File(dest, zipEntry.getName());
        if(zipEntry.isDirectory()) {
            file.mkdirs();
        } else {
            FileOutputStream fos = new FileOutputStream(file);
            byte[] buffer = new byte[1024];
            int len = 0;
            while((len = zip.read(buffer)) != -1) {
                fos.write(buffer, 0, len);
            }
            fos.close();
        }
        zip.closeEntry();
    }
    zip.close();
}
```


### 压缩
- 压缩本质: 把每个文件/文件夹看作ZipEntry对象放到压缩包中







