### 错误/恐慌
1.defer + recover捕获错误
defer 用于注册延迟执行的函数，这些函数会在当前函数返回前，无论正常返回还是 panic，按 **后进先出（LIFO** 的顺序执行。
defer机制：
延迟调用栈：每个 goroutine 栈帧维护一个defer链表，defer语句会将函数压入该链表。
参数绑定：defer语句执行时，不是延迟函数执行时，就会计算并保存参数值。
执行时机：当函数return、发生 panic 或执行到os.Exit()时，defer函数会被触发。

多个defer会按 LIFO 顺序执行，因此捕获 panic 的defer应最早注册。
```go
func test() {
	//利用defer+recover来捕获错误。defer后加上匿名函数的调用
    //只有在defer中调用recover才有用
	defer func() {
		//利用recover内置函数，可以捕获错误
		err := recover()
		//如果没有捕获错误，返回值为零值：nil
		if err != nil {
			fmt.Println("错误已被捕获！")
			fmt.Println("err是:", err)
		}
	}()
	//错误捕获提高程序健壮性
	num1 := 10
	num2 := 0
	result := num1 / num2
	fmt.Println(result)
}
```
2.自定义错误
errors.New() 
特性：只能创建静态错误信息，性能更高，适用于静态错误信息，更常用
```go
func test() error {
	num1 := 10
	num2 := 0
	if num2 == 0 {
		//抛出自定义错误，errors包里的New方法
		return errors.New("除数不能为0!")
	} else {
		result := num1 / num2
		fmt.Println(result)
		//没有错误，返回零值
		return nil
	}

}
func main(){
        err := test()
	if err != nil {
		fmt.Println("err:", err)
		panic(err) //常用于程序无法继续运行，可以利用内置函数panic
	}
	fmt.Println("上面的test测试代码执行成功！")
}
```
3.格式化错误
fmt.Errorf()
性质：支持格式化字符串，性能略低，适合动态生成错误
```go
if err != nil {
    //使用 % w 包装底层错误，支持错误链追踪
    return fmt.Errorf("database query failed: %w", err)
}
```