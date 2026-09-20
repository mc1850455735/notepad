
# go关键字

通过在方法前添加 `go` 关键字，即可将该方法的执行转为 goroutine 执行，也就是并发执行，如：
```go
func task() {
    fmt.Println("...")
}
func main() {
    go task()
}
```

同时需要注意的是，在 go 中，main() 函数本身也运行在 goroutine 中。

由于这个特性，在 main 函数中声明的 goroutine，如果想要保证执行，则需要确保 goroutine 在执行完毕前，main goroutine 没有退出。

可以使用 sync 包中的工具保证并发。

# WaitGroup

`sync.WaitGroup` 的作用是：等待一组 goroutine 全部完成后，再继续往下执行。可以把 WaitGroup 想象成一个计数器。

WaitGroup 中有三个关键的函数，分别是：
- `wg.Add()`：向 WaitGroup 中添加一个计数；
- `wg.Done()`：向 WaitGroup 中减少一个计数；
- `wg.Wait()`：等待 WaitGroup 中的所有计数全部完成。

在实际使用时，通常与 `defer` 关键字配合使用。
defer 关键字的作用是：在函数退出前，执行 defer 标注的代码。

```go
func task(wg *sync.WaitGroup) {
    defer wg.Done()
    fmt.Println("Working...")
}
func main() {
    var wg sync.WaitGroup
    wg.Add(2)
    go task(&wg)
    go task(&wg)
    wg.Wait()
}
```

需要注意的是，进行 WaitGroup 的传入时，需要传入引用而不是传入副本。因为函数需要对传入对象中的属性进行操作。

结合循环、goroutine 和 WaitGroup，可以批量对 goroutine 进行创建、等待和销毁：
```go
func main() {
    var wg sync.WaitGroup
    for i := range 5 {  
        wg.Add(1)  
        go func() {  
            defer wg.Done()  
            fmt.Printf("%d is running\n", i)  
        }()  
    }  
    wg.Wait()
}
```

# Mutex

Mutex 用来解决 Go 里多个 goroutine 同时修改共享数据的问题。

当不使用 Mutex 锁时，多个 goroutine 同时试图修改同一个共享数据，就会造成典型的并发修改问题，从而使得最终结果与构想中的结果不一致。

于是，可以使用 Mutex 作为并发锁，通过对共享变量加锁的方式，将共享变量数据的修改串行化。

对于一个原本可能出现问题的代码：
```go
func mutex_test() {
    var wg sync.WaitGroup
    count := 0

    for range 1000 {
        wg.Add(1)
        go func() {
            defer wg.Done()
            count++
        }()
    }
    wg.Wait()

    fmt.Println(count)
}
```

加入 Mutex，避免多个协程之间的数据竞争。同时，可以使用 defer，保证不管函数以什么方式结束，最终锁都可以被释放：

```go
func mutex_test() {
    ...
    var mu sync.Mutex
    ...
    go func() {
        mu.Lock()
        defer mu.Unlock()
        count++
    }
    ...
}
```

# Channel

channel 是 goroutine 之间相互传递数据的一种机制。同时，普通 channel 默认会阻塞等待另一方。通过 `<-` 关键字，可以做到从 Channel 中取数据，或者将数据推入 Channel。

Channel 的简单使用如下：
```go
func channel_test() {
	channel := make(chan int)
	go func() {
		channel <- 123
	}()

	intNum := <-channel
	fmt.Println(intNum)
}
```

通过 `for value := range ch`，会不断读取 channel，直到它被关闭。

## 有缓冲 Channel

有缓冲 Channel 中有一定的容量，在 Channel 未满时，生产者可以不断地向 Channel 中发送消息；同理，在 Channel 未空时，消费者可以不断地从 Channel 中取出消息。



## 无缓冲 Channel

无缓冲 Channel 可以看作只有一个缓冲区域的有缓冲 Channel。对于无缓冲 Channel，其创建方式如下：
```go
channel := make(chan int)
```

无缓冲 Channel 中的数据，发送后必须等待一个接收者进行接收。
# select



# context

