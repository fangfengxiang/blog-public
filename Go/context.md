



### Context 作用

- 上下文信息传递 （request-scoped），比如处理 http 请求、在请求处理链路上传递信息
- 控制子 goroutine 的运行
- 超时控制的方法调用
- 可以取消的方法调用

以上，我总结为两类

(1) 跨函数甚至跨协程 传递上下文信息。解决请求过程变量作用域问题 

CLS(continuation local storage) or TLS(Thread local storage)

(2) 控制goroutine行为。解决goroutine协同问题

> 我们经常使用 Context 来取消一个 goroutine 的运行，这是 Context 最常用的场景之一，Context 也被称为 goroutine 生命周期范围（goroutine-scoped）的 Context，把 Context 传递给 goroutine。但是，goroutine 需要尝试检查 Context 的 Done 是否关闭了

### Context定义

![image-20210215143056082](https://tva1.sinaimg.cn/large/008eGmZEgy1gno7fjuncmj31i30u07gc.jpg)

- Deadline 

  返回这个 Context 被取消的截止日期。如果没有设置截止日期，ok 的值是 false。后续每次调用这个对象的 Deadline 方法时，都会返回和第一次调用相同的结果

- Done

  返回一个只读的 channel。在 Context 被取消时，此 Channel 会被 close，如果没被取消，可能会返回 nil。后续的 Done 调用总是返回相同的结果。当 Done 被 close 的时候，可以通过 ctx.Err 获取错误信息

- Err 

  返回取消的错误原因，即因为什么原因 Context 被取消

- Value

  返回此 ctx 中和指定的 key 相关联的 value

### Context类型

#### emptyCtx

![image-20210215145635803](https://tva1.sinaimg.cn/large/008eGmZEgy1gno868mrrej317v0u0wqf.jpg)

```
var ( 
   background = new(emptyCtx) 
   todo = new(emptyCtx)
)
//返回一个非 nil 的、空的 Context，没有任何值，不会被 cancel，不会超时，没有截止日期。
//一般用在主函数、初始化、测试以及创建根 Context 的时候。
func Background() Context { 
  return background
}
//返回一个非 nil 的、空的 Context，没有任何值，不会被 cancel，不会超时，没有截止日期。
//当你不清楚是否该用 Context，或者目前还不知道要传递一些什么上下文信息的时候，就可以使用这个方法。
func TODO() Context { 
  return todo
}
```

emptyCtx为整个context的根节点，我们通常使用Background来生成一个初始的context

#### valueCtx

```
func WithValue(parent Context, key, val interface{}) Context {
	if key == nil {
		panic("nil key")
	}
	if !reflectlite.TypeOf(key).Comparable() {
		panic("key is not comparable")
	}
	return &valueCtx{parent, key, val}
}

// A valueCtx carries a key-value pair. It implements Value for that key and
// delegates all other calls to the embedded Context.
type valueCtx struct {
	Context
	key, val interface{}
}
```

我们可以看到valueCtx是由结构体嵌套构成的。每次调用withValue的时候，将parentCtx，key，val 嵌套生成一个新的valueCtx。从而形成一条valueCtx父子链

```
func (c *valueCtx) Value(key interface{}) interface{} {
	if c.key == key {
		return c.val
	}
	return c.Context.Value(key)
}
```

获取当前valueCtx中的键值时，如果没有。则通过递归调用的方式，一直往上查找上层节点。如果一直查不到，最后会出现panic

![image-20210215211226173](https://tva1.sinaimg.cn/large/008eGmZEgy1gnoj1b71pgj31360iadiv.jpg)

#### cancelCtx

##### cancelCtx 使用示例

```
func TestCancelCtx(t *testing.T) {

	parCtx := context.Background()
	ctx, cancel := context.WithCancel(parCtx)

	go func(c context.Context) {
		select {
		case <-c.Done():
			t.Log("ctx is done")
		}
	}(ctx)
	t.Log("main process want to exit")

	cancel()
	time.Sleep(3 * time.Microsecond)
}
```

也就是 我们通过调用withCancel得到闭包函数来关闭这个context，传递这个cancelCtx到其他goruntinue

##### cancelCtx 定义

```
type canceler interface {
	cancel(removeFromParent bool, err error)
	Done() <-chan struct{}
}

// A cancelCtx can be canceled. When canceled, it also cancels any children
// that implement canceler.
type cancelCtx struct {
	Context

	mu       sync.Mutex            // protects following fields
	done     chan struct{}         // created lazily, closed by first cancel call
	children map[canceler]struct{} // set to nil by the first cancel call
	err      error                 // set to non-nil by the first cancel call
}
```

cancelCtx 是一个实现canceler接口的结构体，嵌套了一个Context类型变量。

```
mu => 操作加锁 保证并发安全
done => 用于接收取消信号
children => 父子上下文维护
err => 取消 。错误
```

##### WithCancel做了什么

```
//WithCancel 返回一个可以取消的context
func WithCancel(parent Context) (ctx Context, cancel CancelFunc) {
	c := newCancelCtx(parent)  //将传入的上下文包装成私有结构体 context.cancelCtx
	propagateCancel(parent, &c)  //会构建父子上下文之间的关联，当父上下文被取消时，子上下文也会被取消：
	return &c, func() { c.cancel(true, Canceled) }
}

//Canceled 取消的错误信息
var Canceled = errors.New("context canceled")

// newCancelCtx returns an initialized cancelCtx.
func newCancelCtx(parent Context) cancelCtx {
	return cancelCtx{Context: parent}
}
```

也就是withCancel的时候:

```
(1)将上一层的ctx包装成为cancelCtx  ->  类似valueCtx
(2)通过propagateCancel()函数会构建父子上下文之间的关联，如果父ctx也是可以取消的ctx，需要写入相关逻辑。
达到：当父上下文被取消时，子上下文也会被取消
(3)返回cancelCtx 和 绑定取消逻辑的 闭包函数
```

这里为什么不直接将cancel() 设置为包外可访问, 返回cancelCtx 。而是返回一个 CancelFun闭包函数呢？ 我的理解是这样的：

```
(1)cancel()这个函数的逻辑依赖于传参数 removeFromParent,Canceled
,避免将这个逻辑泄漏到调用层还需要做区分 -> 表象原因
(2)CancelFun 应该是withCancel所在协程控制。ctx的调用链存在时，ctx会被传播到多个协程到conxt树中。
避免包外到处串用ctx.cancel()。返回一个闭包，从设计的层面就拒绝了多处调用cancel() 
除非在代码中 明文传递 闭包变量 CancelFun
```

##### CancelFun 做了什么

```
// cancel closes c.done, cancels each of c's children, and, if
// removeFromParent is true, removes c from its parent's children.
func (c *cancelCtx) cancel(removeFromParent bool, err error) {
	if err == nil {
		panic("context: internal error: missing cancel error")
	}
	c.mu.Lock()
	if c.err != nil {
		c.mu.Unlock()
		return // already canceled
	}
	c.err = err
	if c.done == nil {
		c.done = closedchan
	} else {
		close(c.done)
	}
	for child := range c.children {
		// NOTE: acquiring the child's lock while holding parent's lock.
		child.cancel(false, err)
	}
	c.children = nil
	c.mu.Unlock()

	if removeFromParent {
		removeChild(c.Context, c)
	}
}

func (c *cancelCtx) Done() <-chan struct{} {
	c.mu.Lock()
	if c.done == nil {
		c.done = make(chan struct{})
	}
	d := c.done
	c.mu.Unlock()
	return d
}
```

也就是cancel的时候：

```
(1)调用close(done),让监听Done()函数的协程收到信号
(2)循环children节点，递归调用子context的cancel，并将children置空
(3)通过c.Context中将自己从父context的children属性中摘除 
//总结 
通过chan来传递关闭信息，父上下文关闭，需关闭子上下文。子上下文关闭，不影响父上下文，但需要将自身从父上下文的关联关系移除，防止重复取消
```

![image-20210215135606051](https://tva1.sinaimg.cn/large/008eGmZEgy1gno6fa2g3fj31bq0oyn24.jpg)

##### propagateCancel 做了什么

```
// propagateCancel arranges for child to be canceled when parent is.
func propagateCancel(parent Context, child canceler) {
	if parent.Done() == nil {
		return // parent is never canceled
	}
	if p, ok := parentCancelCtx(parent); ok {
		p.mu.Lock()
		if p.err != nil {
			// parent has already been canceled
			child.cancel(false, p.err)
		} else {
			if p.children == nil {
				p.children = make(map[canceler]struct{})
			}
			p.children[child] = struct{}{}
		}
		p.mu.Unlock()
	} else {
		go func() {
			select {
			case <-parent.Done():
				child.cancel(false, parent.Err())
			case <-child.Done():
			}
		}()
	}
}
```

如果读懂了Done() 和 cancel() ,那么propagateCancel 就一目了然了。

- cancalCtx 撤销信号在上下文树中传播

![image-20210216091208665](https://tva1.sinaimg.cn/large/008eGmZEgy1gnp4y6hbtyj31ge0u016b.jpg)

#### timerCtx

##### 使用示例

```
func TestTimeCtx(t *testing.T) {

	parCtx := context.Background()
	d := time.Now().Add(3 * time.Millisecond)
	ctx, cancel := context.WithDeadline(parCtx,d)

	go func(c context.Context) {
		select {
		case <-c.Done():
			t.Log("ctx is done:",c.Err())
		}
	}(ctx)
	t.Log("main process want to exit")

	cancel()
	time.Sleep(5 * time.Microsecond)
}
```

##### timerCtx 结构

```
type timerCtx struct {
	cancelCtx
	timer *time.Timer // Under cancelCtx.mu.

	deadline time.Time
}
```

##### WithDeadline

```
// Canceling this context releases resources associated with it, so code should
// call cancel as soon as the operations running in this Context complete.
func WithDeadline(parent Context, d time.Time) (Context, CancelFunc) {
	if cur, ok := parent.Deadline(); ok && cur.Before(d) {
		// The current deadline is already sooner than the new one.
		return WithCancel(parent)
	}
	c := &timerCtx{
		cancelCtx: newCancelCtx(parent),
		deadline:  d,
	}
	propagateCancel(parent, c)
	dur := time.Until(d)
	if dur <= 0 {
		c.cancel(true, DeadlineExceeded) // deadline has already passed
		return c, func() { c.cancel(false, Canceled) }
	}
	c.mu.Lock()
	defer c.mu.Unlock()
	if c.err == nil {
		c.timer = time.AfterFunc(dur, func() {
			c.cancel(true, DeadlineExceeded)
		})
	}
	return c, func() { c.cancel(true, Canceled) }
}
```

winthDeadline

```
(1)父context截止时间是否提前于当前截止时间，如果是的话，直接返回cancelCtx
(2)构造一个cancelCtx，把到期时间入参赋值给deadline，得到timeCtx
(3)通过propagateCancel()装载父子上下文信息
(4)获得到达deadline 需要的时长
(5)通过time.AfterFunc() 延迟调用c.cancel() 方法
(6)返回timerCtx，CancelFun
```

##### WithTimeOut

```
// WithTimeout returns WithDeadline(parent, time.Now().Add(timeout)).
//
// Canceling this context releases resources associated with it, so code should
// call cancel as soon as the operations running in this Context complete:
//
// 	func slowOperationWithTimeout(ctx context.Context) (Result, error) {
// 		ctx, cancel := context.WithTimeout(ctx, 100*time.Millisecond)
// 		defer cancel()  // releases resources if slowOperation completes before timeout elapses
// 		return slowOperation(ctx)
// 	}
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc) {
	return WithDeadline(parent, time.Now().Add(timeout))
}
```

不需要解释了

### Context总结

![image-20210215135436098](https://tva1.sinaimg.cn/large/008eGmZEgy1gno6ds3pajj31bu0pan2m.jpg)

#### 使用context时约定俗成的规则：

- 一般函数使用 Context 的时候，会把这个参数放在第一个参数的位置。

- 从来不把 nil 当做 Context 类型的参数值，可以使用 context.Background() 创建一个空的上下文对象，也不要使用 nil

- Context 只用来临时做函数之间的上下文透传，不能持久化 Context 或者把 Context 长久保存。把 Context 持久化到数据库、本地文件或者全局变量、缓存中都是错误的用法。

- key 的类型不应该是字符串类型或者其它内建类型，否则容易在包之间使用 Context 时候产生冲突。使用 WithValue 时，key 的类型应该是自己定义的类型。ps: 官网示例也是用string

- 常常使用 struct{}作为底层类型定义 key 的类型。对于 exported key 的静态类型，常常是接口或者指针。这样可以尽量减少内存分配

- 一般传递context 应该使用值传递而不是引用传递，特别是跨协程传递的时候。错误示例：gin框架的context处理

  

#### context目前的争议

- Context 包名导致使用的时候重复 ctx context.Context
- Context.WithValue 可以接受任何类型的值，非类型安全
- Context 漫天飞，函数污染 代码侵入性问题

针对以上问题的ISSUE上的讨论同展望：

- 把context 作为同channel同级的内置类型
- 减少使用context存值，或者将context包关于协程控制部分的功能独立出来
- goroutine-local storage 
- 支持函数重载 避免扩展第一参数为ctx时需要侵入已有代码
- 在go2中移除context 

### Reference

- [Go Concurrency Patterns: Context](https://blog.golang.org/context)
- [How to correctly use context.Context in Go 1.7](https://medium.com/@cep21/how-to-correctly-use-context-context-in-go-1-7-8f2c0fafdf39)

- [go2context 设想issue](https://github.com/golang/go/issues/28342)
- [Context isn’t for cancellation](https://dave.cheney.net/2017/08/20/context-isnt-for-cancellation)
- [context-should-go-away-go2](https://faiface.github.io/post/context-should-go-away-go2/)
- [Replace Context with goroutine-local storage](https://github.com/golang/go/issues/21355)
- [gls REMEAD](https://pkg.go.dev/github.com/jtolio/gls?readme=expanded#section-readme)

- [Go Context 飞雪无情](https://www.flysnow.org/2017/05/12/go-in-action-go-context.html)

###### 思考题

- “可撤销的”在context包中代表着什么？“撤销”一个Context值又意味着什么？
- 撤销信号是如何在上下文树中传播的？
- Context值在传达撤销信号的时候是广度优先的，还是深度优先的？其优势和劣势都是什么？
- 如何通过 Context 实现日志跟踪

##### 几段有趣的话

```
根据这句话：“A great mental model of using Context is that it should flow through your program. Imagine a river or running water. This generally means that you don’t want to store it somewhere like in a struct. Nor do you want to keep it around any more than strictly needed. Context should be an interface that is passed from function to function down your call stack, augmented as needed.”
感觉上Context设计上更像是一个运行时的动态概念，它更像是代表传统call stack 层层镶套外加分叉关系的那颗树。代表着运行时态的调用树。所以它最好就是只存在于函数体的闭包之中，大家口口相传，“传男不传女”。“因为我调用你，所以我才把这个传给你，你可以传给你的子孙，但不要告诉别人！”。所以最好不要把它保存起来让旁人有机会看得到或用到。

有人提到这种风格的入侵性，我能理解你的感觉。但以我以前玩Node.js中的cls (continuation local storage)踩过的那些坑来看，我宁愿两害相权取其轻。这种入侵性至少是可控的，显式的。同步编程的世界我们只需要TLS(Thread local storage)就好了，但对应的异步编程的世界里玩cls很难搞的。在我来看，Context显然是踩过那些坑的老鸟搞出来的。
```

```
But alas, Andrew Gerrand's typically diplomatic answer to the question of goroutine-local variables was:"
We wouldn't even be having this discussion if thread local storage wasn't useful. But every feature comes at a cost, and in my opinion the cost of threadlocals far outweighs their benefits. They're just not a good fit for Go."

但是，可惜，安德鲁·格朗（Andrew Gerrand）对goroutine-local变量问题的典型外交回答是：
如果线程本地存储没有用，我们甚至不会进行讨论。但是每个功能都是有代价的，在我看来，threadlocals的成本远远超过其收益。它们只是不适合Go。
```



