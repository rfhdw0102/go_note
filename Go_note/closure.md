### 闭包
闭包是一个函数值，它捕获了其外部作用域中的变量。即使该函数在其原始作用域之外执行，这些被捕获的变量也会被保留。常用于回调、延迟执行和状态管理等场景。
闭包与匿名函数的区别：
匿名函数：没有名称的函数，例如 func() { ... }。
闭包：是一个函数值，它捕获了外部变量。
所有闭包都基于匿名函数实现，但并非所有匿名函数都是闭包
```go
//简单的闭包实现
func adder() func(int) int {
    sum := 0 // 被闭包捕获的变量,sum的值是会保存的
    return func(x int) int {
        sum += x // 可以访问并修改外部的sum
        return sum
    }
}
//两种方法效果是一样的
func sum(a int) int {
	sum := 0
	func(a int) {
		sum += a
	}(a)
	return sum
}
func main() {
    add := adder() // 创建一个闭包
    fmt.Println(add(1)) // 输出：1（sum = 0+1）
    fmt.Println(add(2)) // 输出：3（sum = 1+2）
    fmt.Println(add(3)) // 输出：6（sum = 3+3）
}
```
关键点：
捕获变量：闭包内部引用了外部的 sum 变量。
状态保持：每次调用闭包时，sum 的值会被保留并累积。