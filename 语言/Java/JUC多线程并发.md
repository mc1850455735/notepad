
# 概念

**概念**
- 进程, 是程序的基本执行实体
- 线程, 是操作系统能够进行运算调度的最小单位, 他被包含在进程中, 是进程中的实际运作单位, 相互独立, 同时又可以同时运行
- 有了多线程, 就可以让程序同时做多件事情, 提高效率

**并发和并行**
- 并发 : 同一时刻, 有多个指令在单个CPU上**交替**执行
- 并行 : 同一时刻, 有多个指令在多个CPU上**同时**执行

### 实现方式

- 继承Thread类的方式进行实现
- 实现Runnable接口的方式进行实现
- 利用Callable接口和Future接口进行实现

##### 继承Thread

* 多线程的第一种启动方法  
	1. 自己定义一个类继承Thread  
	2. 重写run方法  
	3. 创建子类的对象并启动线程
- 使用 `thread.start()` 开启一个线程
- 使用 `setName` 为线程命名, 使用 `getName` 在线程中获取名称

*MyThread.java*
```java
public class MyThread extends Thread{  
    @Override  
    public void run() {  
        // 书写线程要执行的代码  
        for (int i = 0; i < 100; i++) {  
            System.out.println("Hello World!");  
        }  
    }  
}
```
*Main.java*
```java
public static void main(String[] args) {  
    MyThread t1 = new MyThread();  
    // 开启线程 
    t1.start();  
}
```

##### 实现Runnable

* 多线程的第二种启动方法  
	1. 自己定义一个类实现Runnable接口  
	2. 重写其中的run方法  
	3. 创建自己的类的对象  
	4. 创建一个Thread类的对象, 并开启线程
- 在使用实现Runnable接口的对象的name时, 不能直接调用`getName`, 因为该对象并不是一个线程对象
- 通过 `Thread.currentThread` 可以获取到当前代码块所在的线程, 调用该线程对象的 `getName` 方法, 可以间接获取线程名称

*Main.java*
```java
public static void main(String[] args) {  
    // 创建MyRun对象表示多线程要执行的任务  
    MyRun myRun = new MyRun();  
    // 创建线程对象  
    Thread t1 = new Thread(myRun);  
    Thread t2 = new Thread(myRun);  
    // 为线程设置名字  
    t1.setName("线程1");  
    t2.setName("线程2");  
    // 开启线程  
    t1.start();  
    t2.start();  
}
```

*MyRun.java*
```java
public class MyRun implements Runnable{  
    @Override  
    public void run() {  
        for (int i = 0; i < 10; i++) {  
            System.out.println(Thread.currentThread().getName() + ": Hello Thread");  
        }  
    }  
}
```


##### 利用Callable接口和Future接口

- 上述两种方法有一个共同缺点, 即无法获取线程运行的结果
 * 多线程的第三种实现方式  
 * 特点: 可以获取到多线程运行的结果  
	 1. 创建一个类MyCallable实现Callable接口  
	 2. 重写call(有返回值, 表示多线程运行的结果)  
	 3. 创建MyCallable的对象, 表示多线程要执行的任务  
	 4. 创建FutureTask的对象, 用来管理多线程运行的结果  
	 5. 创建Thread类的对象, 并启动  

*Main.java*
```java
public static void main(String[] args) throws ExecutionException, InterruptedException {  
    // 表示要执行的任务  
    MyCallable mc = new MyCallable();  
    // 管理运行的结果  
    FutureTask<Integer> ft = new FutureTask<>(mc);  
    // 创建线程的对象  
    Thread t1 = new Thread(ft);  
    // 启动线程  
    t1.start();  
    // 获取多线程运行结果  
    Integer res = ft.get();  
    System.out.println(res);  
}
```

*MyCallable.java*
```java
public class MyCallable implements Callable<Integer> {  
    @Override  
    public Integer call() throws Exception {  
        // 求1~100的和  
        int sum = 0;  
        for (int i = 0; i < 100; i++) {  
            sum += i;  
        }  
        return sum;  
    }  
}
```


##### 对比
- 继承Thread类
	- 优点 : 编程比较简单, 可以直接使用Thread类中的方法
	- 缺点 : 可扩展性较差, 不能再继承其他的类
- 实现Runnable接口 & 实现Callable接口
	- 优点 : 扩展性强, 实现该接口的同时还可以继承其他的类
	- 缺点 : 编程相对复杂, 不能直接使用Thread类中的方法

### 常见成员方法

**常见方法**
- `String getName()` : 返回此线程名称
	- 如果没有给线程设置名称, 则线程有默认名称
	- 格式: Thread-X ( X序号, 从0开始 )
- ``void setName(String name)`` : 设置当前线程名称(也可以在构造方法)
	- 如果需要给线程设置名字, 可以使用set方法, 也可以使用构造方法
	- 自己写的thread类需要在构造方法中调用父类的构造方法
- `static Thread currentThread()` : 获取当前线程的对象
	- JVM虚拟机启动后会自动启动多个线程
	- 其中, main方法线程的名称为 main, 其作用为调用main方法并执行其中的代码
- `static void sleep(long time)` : 让线程休眠指定的时间, 单位ms
- `void setPriority(int newPriority)` : 设置线程优先级
	- 默认线程优先级为5, 越大优先级越高
	- 在Java中, 优先级高的线程抢占到CPU的概率更大, 但不一定是优先级高的线程一定能获得CPU
- `final int getPriority()` : 获取线程优先级
- `final void setDaemon(boolean on)` : 设置为守护线程
	- 当其他的非守护线程结束后, 守护线程会陆续结束
- `public static void yield()` : 出让线程/礼让线程
	- 出让当前CPU的执行权, 使所有线程重新开始抢夺执行权
- `public final void join()` : 插入线程/插队线程
	- 将某个线程插入到当前线程之前

### 线程生命周期
![[语言/Java/Inbox/Pasted image 20250917172436.png]]


# 线程安全

- 多线程在卖票过程中出现的问题
	- 相同的票出现了多次
	- 出现了超出范围的票
- 这种问题即所谓的临界区域问题, 可以通过加锁的方法避免

### 同步代码块

- 在Java中, 这种加锁被称为**同步代码块**, 使用关键字 `synchronized`
	- 特点1 : 锁默认打开, 有一个线程进去了, 锁自动关闭
	- 特点2 : 里面代码全部执行完毕, 线程出来, 锁自动打开
- 对于同一个临界区, `synchronized` 的锁对象一定要是唯一的
	- 通常情况下, 可以使用该类的class对象, 该对象一定是唯一的

**代码示例**
```java
public class MyThread extends Thread{  
    static int ticket = 0;  
    @Override  
    public void run() {  
        while(true) {  
            synchronized (MyThread.class) {  
                if(ticket < 100) {  
                    try {  
                        sleep(100);  
                    } catch (InterruptedException e) {  
                        throw new RuntimeException(e);  
                    }  
                    ticket++;  
                    System.out.println(
                        getName() + 
                        "正在卖出第" + 
                        ticket + 
                        "张票"
                    );  
                } else {  
                    break;  
                }  
            }  
        }  
    }  
}
```

### 同步方法

- 如果想要将整个方法内的所有代码都加锁, 可以直接将 `synchronized` 关键字加在方法上, 这样的方法就被称为**同步方法**
- 同步方法的锁对象不能由自己指定
	- 非静态方法 : this
	- 静态方法 : 当前类的字节码文件对象
- 格式 : `修饰符 synchronized 返回值 方法名(参数) {...}`
- `Ctrl + Alt + M` : 抽取方法

**程序实例**
```java
int ticket = 0;
@Override  
public void run() {  
    while(true) {  
        if (sellTicket()) break;  
    }  
}  
private synchronized boolean sellTicket() {  
    if(ticket >= 100) {  
        return true;  
    } else {  
        ticket++;  
        System.out.println
        (Thread.currentThread().getName() + "@" + ticket);  
    }  
    return false;  
}
```

##### StringBuild和StringBuffer
- StringBuild是线程不安全的, 而StrinBuffer是线程安全的
- 原因就是StringBuffer中的方法添加了 `synchronized` 修饰符

### Lock锁

**概述**
- 虽然已有了同步代码块和同步方法, 但并不能直观决定加锁和释放锁的过程
- 为了清晰表达如何加锁和释放锁, JDK5提供了一个新的锁对象 `Lock`
- Lock实现提供比使用synchronized方法和语句更广泛的锁定操作
	- 手动上锁, 手动释放锁
	- `void lock()` : 获得锁
	- `void unlock()` : 释放锁
- Lock接口不能直接实例化, 通常使用它的实现类`ReentrantLock`来实例化
- 使用时注意一定最终要释放锁对象, 通常使用try-finally保证执行
```java
public class MyThread extends Thread {  
    static int ticket = 0;  
    static Lock lock = new ReentrantLock();  
    @Override  
    public void run() {  
        while (true) {  
            lock.lock();  
            try {  
                if (ticket == 100) {  
                    break;  
                } else {  
                    Thread.sleep(10);  
                    ticket++;  
                    System.out.println
                    (getName() + "在卖第" + ticket + "张票");  
                }  
            } catch (Exception e) {  
                throw new RuntimeException(e);  
            } finally {  
                lock.unlock();  
            }  
        }  
    }  
}
```

##### 死锁
- 死锁就是锁的相互嵌套导致了程序执行流程的相互依赖
- 写代码时避免出现锁的相互嵌套, 是避免出现死锁的最佳方式

### 生产者消费者模型

**常用方法**
- `void wait()` : 当前线程等待, 直到被其他线程唤醒
- `void notify()` : 随机唤醒单个线程
- `void notifyAll()` : 唤醒所有线程
- 进行等待和唤醒时, 需要和当前线程与锁进行绑定

*Desl.java*
```java
public class Desk{  
    // 0: 无面条 1: 有面条  
    public static int foodFlag = 0;  
    public static int count = 10;  
    public final static Object lock = new Object();  
}
```

*Producer.java*
```java
public class Producer extends Thread{  
    @Override  
    public void run() {  
        while (true) {  
            synchronized (Desk.lock) {  
                try {  
                    if(Desk.count <= 0) {  
                        System.out.println("做累了");  
                        break;                    }  
                    if (Desk.foodFlag == 1) {  
                        Desk.lock.wait();  
                    } else {  
                        Desk.foodFlag = 1;  
                        System.out.println("厨师做了一碗面");  
                        Desk.lock.notifyAll();  
                    }  
                } catch (InterruptedException e) {  
                    throw new RuntimeException(e);  
                }  
            }  
        }  
    }  
}
```

*Consumer.java*
```java
public class Consumer extends Thread {  
    @Override  
    public void run() {  
        while (true) {  
            synchronized (Desk.lock) {  
                try {  
                    if (Desk.count <= 0) {  
                        System.out.println("吃饱了");  
                        break;                    }  
                    if (Desk.foodFlag == 0) {  
                        Desk.lock.wait();  
                    } else {  
                        Desk.count--;  
                        System.out.println("食客吃了一碗面");  
                        Desk.foodFlag = 0;  
                        Desk.lock.notifyAll();  
                    }  
                } catch (InterruptedException e) {  
                    throw new RuntimeException(e);  
                }  
            }  
        }  
    }  
}
```

##### 阻塞队列

**概述**
- put数据时 : 放不进去, 会进行阻塞
- take数据时 : 取不到数据, 会进行阻塞
- 生产者和消费者必须使用同一个阻塞队列
- 使用时不需要加锁, 因为put和take函数本身就会使用lock锁对代码加锁
- 虽然外部不加锁对两线程操作与展示顺序有影响, 但实际上对数据安全性没有影响

**相关接口**
- Iterable : 迭代器
- Collection : 单列集合
- Queue : 队列
- BlockingQueue : 阻塞队列接口

**实现类**
- ArrayBlockingQueue : 底层是数组, 有界
- LinkedBlockingQueue : 底层是链表, 无界, 但不是真正的无界, 其界限为int的最大值

*Producer.java*
```java
public class Producer extends Thread{  
    ArrayBlockingQueue<String> blockingQueue;  
    public Producer(ArrayBlockingQueue<String> blockingQueue) {  
        this.blockingQueue = blockingQueue;  
    }  
  
    @Override  
    public void run() {  
        while (true) {  
            try {  
                blockingQueue.put("面条");  
                System.out.println("厨师放了一碗" + "面条");  
            }catch (InterruptedException e) {  
                throw new RuntimeException(e);  
            }  
        }  
    }  
}
```

*Consumer.java*
```java
public class Consumer extends Thread {  
    ArrayBlockingQueue<String> blockingQueue;  
    public Consumer(ArrayBlockingQueue<String> blockingQueue) {  
        this.blockingQueue = blockingQueue;  
    }  
  
    @Override  
    public void run() {  
        while(true) {  
            try {  
                String food = blockingQueue.take();  
                System.out.println("获得了一碗" + food);  
            } catch (InterruptedException e) {  
                throw new RuntimeException(e);  
            }  
        }  
    }  
}
```

### 线程的状态

