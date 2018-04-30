goroutine
===


协程 
- 轻量级线程
- 非抢占式多任务处理，由协程主动交出控制权
- 编译器，解释器，虚拟机层面的。
- 用户级线程，也就是通过编译器或解释器调度器切换协程，而不是操作系统切换
- 多个协程可能在一个线程上执行，也可能在多个线程上执行

协程 宕机
- 由于协程是非抢占式的，所以如果没有机会让出控制权就会照成宕机
- 协程 一般只有两个操作会 让出控制权，一是 IO操作，二是主动让出控制权 使用 runtime.Gosched()

go run -race xxx.go 来检测数据访问冲突

>子程序是协程的特例

协程可能切换的点
- IO 操作，select
- 函数调用（有时）
- channel
- 等待锁
- runtime.Gosched()
- 上面只是参考，不能保证一定切换，也不能保证在其他地方不切换

channel
- channel
- buffered channel
- range
- channel 理论基础CSP模型
close() 关闭channel


> 不要通过共享内存来通信，通过通信来共享内存

channel 如何等待任务结束
- 通信，加一个用于发送结束标志的通道结束符
- 使用sync.WaitGroup 

用select 来调度 非阻塞式获取
```
select {
case n := <-c1 :
case n := <-c2 :
}
```
nil channel 会被阻塞，不会被调度
 如果外层加for 死循环就有 调度器的意思

传统的同步机制(在go中尽量少用，因为有CSP模型的select那些)
- waitGroup
- Mutex  互斥量
Mutex.Lock()  Mutex.Unlock()
- Cond

