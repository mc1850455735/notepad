
## IO类
目前使用的IO类型和对象都是用来操纵char类型的

**三种IO类型**
|头文件|类型|
|-|-|
|iostream|从流中读写数据|
|fstream|从文件中读写数据|
|sstream|从字符串中读写数据|
加上w, 就是对于宽字符类型的操作

**关系**
- ifstream和istringstream都是继承自istream的
- ofstream和ostringstream都是继承自ostream的
- 可以像使用cout和cin一样使用这种类

### 不能进行赋值或拷贝

### 条件状态

**strm是一种IO类型, 比如ifstream等**
- strm::iostate -> 表达条件状态
- strm::badbit -> 流已崩溃
- strm::failbit -> IO操作失败
- strm::eofbit -> 流到达文件结束
- strm::goodbit -> 指出流未处于错误状态

假定有流s
- s.eof() -> eof位是否置位, 是则返回true
- s.fail() -> 同上
- s.bad() -> 同上
- s.good() -> 流s处于有效状态
- s.clear() -> 将流中所有条件状态复位
- s.setstatus(flags) -> 根据所给的strm::iostate类型的flag, 复位对应条件位
- s.rdstatus() -> 返回当前条件状态

在使用cin之前, 应检查cin的状态, 在cin无错误的情况下进行操作

##### 查询流的状态
- 通过iostate可以用来表示流的完整状态, 其作为一个位集合来使用.
- IO库定义了4个iostate类型的constexpr值
- 可以与位运算符一起用来一次性检测或设置多个位标志

**各种标志位**
当对应事件发生时, 置位1
- badbit -> 不可恢复的系统级错误
- failbit -> 可恢复的普通输入输出错误
- eofbit -> 到达文件尾, failbit和eofbit都会被置位
- goodbit -> 流未发生错误
- 当badbit被置位时, 查询failbit的返回结果也将会是true
- fail()和good()可以表示总体的正确错误, 而eof()和bad()表示特定的错误

##### 管理流的状态
- 通过rdstate()获取流的状态
- 通过clear()清除流的状态
- 通过setstate()设置/恢复流的状态

### 管理输出缓冲

- 每个输出流都管理一个缓冲区, 用来保存程序读写的数据
	- 文本可能立即被打印, 也有可能被系统保存在缓冲区中
	- 通过这种操作, 可以减少调用写操作的次数, 提升系统效率
- 缓冲区刷新原因
	- 程序正常结束
	- 缓冲区满
	- 可以使用如endl来显式刷新缓冲区
	- 每次输出操作后, 可以用**操作符unitbuf**设置流内部状态, 以清空缓冲区
		- 默认情况下, cerr设置unitbuff, 因此写到cerr的内容都是立即刷新的
	- 一个输出流可能**关联**到另一个输出流. 这种情况下, 读写被关联的流时, 关联的流的缓冲区会被刷新
		- 默认情况下, cin和cerr都关联到cout, 因此读cin和写cerr都会导致cout缓冲区被刷新

##### 刷新缓冲区

**刷新输出缓冲区**
- 操作符
- endl: 换行并刷新缓冲区
- ends: 输出一个空字符, 然后刷新缓冲区
- flush: 只刷新缓冲区

**unitbuf操纵符**
- 使用unitbuf操作符, 告诉流接下来每次写操作后都进行一次flush操作
- 使用nonunitbuf重置流

**注意**
调试程序时要确保输出字符串的刷新

##### 关联输入和输出流
- 当输入流被关联到输出流时, 任何试图从输入流读取数据的操作都会先刷新输出流.
	- 标准库将cout和cin关联到一起

**tie函数**
- tie函数有两个重载的版本
- 一个版本不带参数, 返回指向输出流的指针
	- 如果未关联到流, 则返回空指针
- 另个版本接收一个指向ostream的指针, 将自己关联到此ostream
	- 既可以将istream关联到ostream, 也可以将ostream关联到ostream
- 每个流最多可以关联到一个ostream, 但是一个ostream可以被多个流关联

## 文件输入输出

- 头文件fstream提供了三种类来实现文件的输入输出
	- ifstream: 从给定文件读数据
	- ofstream: 向给定文件写数据
	- fstream: 既能读也能写
- 与iostream相同, 可以使用<<和>>来读写文件流, 也可以使用getline从ifstream中读数据

fstream中还添加了一些新的操作来管理与流关联的文件
|操作名|效果|
|-|-|
|fstream fstrm|创建一个未绑定的文件流
|fstream fstrm(s)|创建一个fstream,并打开一个名为s的文件
|fstream fstrm(s, mode)|以mode模式打开s文件
|fstrm.open(s)|打开名为s的文件, 并将其与fstrm绑定
|fstrm.close()|关闭与fstrm绑定的文件
|fstrm.is_open()|判断与fstrm关联的文件是否成功打开
- s既可以是string类型, 也可以是C风格字符串指针
- 这些构造函数都是explict类型的(抑制构造函数的隐式转换)
- 默认的文件模式mode依赖于fstream的类型

### 使用文件流对象
- 创建文件流时提供文件名参数, 相当于创建后直接使用open函数
- C++11中, 文件名既可以是string, 也可以是C风格字符串
	- 旧版本标准库只允许C风格字符串

##### 使用fstream替代iostream&






