go简介
===



go 很特别
- 没有对象，没有继承，没有泛型，没有try/catch
- 有接口，函数式编程，CSP并发模型(goroutine+channel)
- 语法简单，但实现思想不同


面向接口
- 结构体
- duck typing的概念
- 组合的思想

函数式编程
- 闭包的概念

工程化
- 资源管理，错误处理
- 测试和文档
- 性能调优

并发编程
- goroutine 和 channel
- 调度器


### 变量定义
```
var a int = 123
var a = 123
a := 123

var a,b int = 123,321
var a,b = 123,'hello'

尽量使用：= 但是：= 只能在函数内部使用 不能在函数外部
```
### 内建变量类型
- bool
- string
- (u)int 	, (u)int8
- uintptr
- byte 8 位
- rune(32位) 字符类型
- float32 float64
- complex64 complex128  complex64 的实部和虚部都是float32

### 强制类型转换
go中的类型转换，只有强制类型转换，没有隐式类型装换

### 常量的定义
```
const a,b = 3,4
const a,b int = 3,4
值得一提的是const 变量如果不指定类型，可以作为各种类型使用，这点是var 不行的

枚举类型 go语言 通过一个const 来实现枚举类型
const(
	cpp = iota
	-
	java
	php
)
```

### 条件语句
- if的条件不需要括号
- if条件里是可以赋值的，赋值的变量 作用域就在这个if语句块里

### switch
- switch 会自动break，除非使用fallthrough
- switch 可以没有表达式，在case后加条件就行了

### 循环
- for 条件里不需要括号
- for 的条件里可以省略初始化条件，结束条件，递增表达式
- go里没有while，如果要用while可以用 for代替
```
for d>xxx {

}
```
- 死循环 (并发编程中常用)
```
for {

}
```

### 函数定义
```
func foo(a,b int, c string) int {

}

其实个人觉得最好还是这样写
func foo(a int, b int, c string) int {

} 这样能更清晰地看出有多少个形参
```
- 允许返回多个参数
```
func foo(a int, b int, c string) (int, string) {
	return a, b
}
```
- 允许为返回值命名
```
对调用者而言没有区别 
个人感觉没什么用？
func foo(a int, b int, c string) (q,r int) {
	q = a;
	r = b;
	return 
}
```
- 可变参数列表
```
func foo(number ...int) int {
	return number[i]
}
```
- 没有默认参数，可选参数
- 没有函数重载
- 函数可以作为参数

### 指针
```
var a int = 2
var pa *int = &a
*pa = 3
```
- go的指针不能运算

### 值传递 还是 引用传递
- go 语言只有值传递 一种方式
- 如果你要实现引用传递，只需要传指针

b8:27:eb:fe:40:53