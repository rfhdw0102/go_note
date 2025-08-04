### 基本数据和字符串之间的转换
基本数据类型转字符串，主要有两种方法
```go
        //方法一 利用Springf
	str1 := fmt.Sprintf("%d", 2)//整数
	str2 := fmt.Sprintf("%f", 3.14)//浮点
	str3 := fmt.Sprintf("%t", true)//布尔
	str4 := fmt.Sprintf("%c", 'c')//字符
	//方式2：用包strconv里的方法
	s1 := strconv.FormatInt((int64(s1)), 10)
	//参数：第一个参数必须转为int64类型，第二个参数指定字面值的进制形式为十进制
	s2 := strconv.FormatFloat(s2, 'f', 9, 64)
	//第二个参数:f(-ddd.dddd) 第三个参数：9，保留9位小数 第四个参数：表示这个小数是float64类型
	s3 := strconv.FormatBool(s3)
	//string转基本数据类型，返回值有两个，（value，err），一般不要err，直接忽略
	flag, _ := strconv.ParseBool(“true”)//字符串转布尔
	num1, _ := strconv.ParseInt(“66”, 10, 64)//字符串转整数，第二个参数是进制
	num2, _ := strconv.ParseFloat(“3.14”, 64)//字符串转浮点数
        //注意！！！！
        ParseFloat（）是固定返回float64类型的浮点数，里面的第三个参数并不是把字符串转换为float相应字节的浮点数，而是跟精度有关
        比如ParseFloat（“66”, 10, 32）把“66”转为float32，在转为float64，32只是控制精度的，有兴趣的可以去了解
        //其中字符串和整数之间的转换还有一种好用的方法
        //字符串转整数
	num1, _ := strconv.Atoi("666")
	//整数转字符串
	str1 := strconv.Itoa(88)

        //字符串和字符之间转换可以强转
        strr := string('A')
```
优先使用strconv包：在性能敏感的场景或者简单的类型转换中，strconv包的函数是首选，因为它们效率高且类型安全。
使用fmt.Sprintf()：当需要复杂的格式化输出（如日期、货币格式）时，fmt.Sprintf()是更合适的选择。
谨慎使用强制转换：仅在处理byte/[]byte和rune时使用强制转换，避免对数值类型进行强制转换，以免产生意外结果。
### 切片
Go里面特有的数据类型, 是对数组一个连续片段的引用，是一个引用类型, 提供了一个相关数组的动态窗口
数组有特定的用处，但是长度固定不可变，有些呆板，不常用，但切片更常用，他是建立咋数组之上的抽象，有更强大的能力和便捷性
```go
    //切片定义之后不能直接使用，需要让其引用到一个数组，或者make一个空间来使用
    //eg:var slice []int  不能直接使用(不能赋值)
    //声明/定义
    var intarr = [5]int{1, 2, 3, 4, 5}
    //1.切片定义在数组之上
    var slice []int = intarr[1:4] //简写为var slice = intarr{1:}
    var slice = intarr[0:len(intarr)] //var slice = intarr[:]
    varvar slice = intarr[0:3] //var slice = intarr[:3] 
    //2.利用make，make函数的三个参数（1。切片类型，2.切片长度，3.切片容量）
    //make函数底层创建一个数组，数组不可见
    slice := make([]int, 10, 20)//长度一定小于等于容量
    //3.直接指定具体数组，使用原理类似make的方式
    slice := []int{1, 4, 7}
```
切片的一些常用的方法
```go
    len(slice)//返回长度
    cap(slice)//返回容量
    //切片可以继续切片，也可追加元素
    newSlice := slice[1:5]//继续切片
    slice = append(slice, 88, 99)//追加元素
    //也可以通过切片追加切片
    slice = append(slice, slice1...)//要带...，意思就是将一个切片的所有元素展开，这是因为切片是 [ ]int 类型，而 append 要求接收的参数是 int 类型，如果直接传入 slice1 会报错
    //遍历
    //1.for循环
    for i := 0; i < len(slice); i++ {
	    fmt.Println(slice[i])
    }
    //2.for-range
    for _, value := range slice {
	    fmt.Print(value)
    }
```
**切片的扩容机制**
切片是一个结构体（了解）
```go
type slice struct {
    array unsafe.Pointer // 指向底层数组
    len   int            // 切片长度
    cap   int            // 切片容量
}
```
重点：
当切片追加元素时，如使用append函数，若len超过了cap，则会触发扩容
Go 1.18 之前：扩容策略为：
若新长度（len + 新增元素数）超过当前容量的 2 倍：直接扩容至新长度。 
小于 1024 时翻倍。
大于 1024 时每次增加 25%，直到满足新长度要求
扩容完成之后，会进行内存对齐，主要通过向上取整到 8 字节的倍数实现
Go 1.18+：
Go1.18不再以1024为临界点，而是设定了一个值为256的threshold，以256为临界点；超过256，不再是每次扩容1/4，而是每次增加（旧容量+3*256）/4；其中cap为预期容量，也就是说添加了数据之后需要的容量
当新切片需要的容量cap大于两倍扩容的容量，则直接按照新切片需要的容量扩容；
当原 slice 容量 < threshold 的时候，新 slice 容量变成原来的 2 倍；
当原 slice 容量 > threshold，进入一个循环，每次容量增加（旧容量+3*threshold）/4。
内存对齐：Go 1.18 引入了更精细的内存块分级，将内存划分为约 70 种不同规格，覆盖从 8 字节到 32KB 的范围。
小对象：8, 16, 24, 32, 48, 64, 80, ..., 512 字节
中等对象：576, 640, 704, 768, ..., 32768 字节
当计算出理论容量后，Go 会选择最接近且大于等于该容量的内存块规格作为实际容量。
对于内存大于32KB的称为大对象，会单独处理.

### map
map基础知识：
```go
        //声明map
        //方式1：
	var a map[int]string
	//如果只声明map，是没有分配空间的，必须通过make初始化才会分配空间
	a = make(map[int]string, 10)
	//键值对是无序的，key不能重复，不然会覆盖value，但value是可以重复的
	//make函数的第二个参数size可以省略，默认分配一个内存
	//方式2：
	b := make(map[int]string, 10)
	//方式3：
	c := map[int]string{
		000: "张三",
		111: "李四", //注意最后也要加 “,”
	}
	//增删查改
	//增加
	b[00000] = "wwww"
	//修改
	b[00000] = "dddd"
	//删除，利用内置函数delete
	delete(b, 11111) //delete的第二个参数是key，如果key存在，删除这个键值对，若不存在，不做任何操作也不报错
	//清空
	//（麻烦）如果我们要删除map中的所有的key，没有一个专门的方法一次删除，可以遍历一下key，逐个删除
	//或者map = make（），make一个新的，让老的成为垃圾被go回收
	//查找
	value, flag := b[00000] //如果b[00000]存在，值返回value，flag为true
	//若不存在b[00000]，value为空，flag为false，不会报错
```
**map扩容机制**
1.map的底层是哈希表,采用链地址法解决哈希冲突，核心结构：
```go
type hmap struct {
    count     int       // 元素数量
    flags     uint8
    B         uint8     // 桶数量为2^B
    noverflow uint16    // 溢出桶的近似数量
    hash0     uint32    // 哈希种子
    
    buckets    unsafe.Pointer  // 桶数组，长度为2^B
    oldbuckets unsafe.Pointer  // 扩容时的旧桶数组
    nevacuate  uintptr         // 扩容进度标记
    // ...
}
```
2.触发扩容的条件
当 map 满足以下两个条件之一时，会触发扩容：
负载因子过高：当元素数量（count）超过桶数量（2^B）的 6.5 倍时,即 count > 6.5 * 2^B。
负载因子 = 元素数量 / 桶数量。
默认负载因子为 6.5，这是在空间和时间效率之间的平衡。
溢出桶过多：当溢出桶数量过多时（具体条件较复杂，大致为桶数量较小时溢出桶过多）。
3.扩容类型
Go 的 map 扩容分为两种类型：
等量扩容：
触发条件：溢出桶过多，但元素数量未超过负载因子限制。
扩容方式：桶数量不变（2^B 不变），重新排列键值对，减少溢出桶。
目的：减少内存碎片，提高访问效率。
翻倍扩容：
触发条件：元素数量超过负载因子限制。
扩容方式：桶数量翻倍:2^B → 2^(B+1)，重新哈希所有元素。
目的：降低负载因子，减少哈希冲突。
4.Go 的 map 扩容采用 **渐进式（incremental** 方式，而非一次性完成，以避免长时间阻塞：
创建新桶：扩容时，创建新的桶数组（翻倍或等量）。
旧桶标记：将旧桶数组保存到 oldbuckets 字段。
逐步迁移：在后续的 map 操作（如插入、删除、查找）中，每次处理 1-2 个旧桶，将其元素迁移到新桶。
完成迁移：所有旧桶迁移完成后，释放旧桶内存。
```go
func main() {
    m := make(map[int]int, 2) // 初始桶数量为2^1=2
    // 添加元素触发扩容
    for i := 0; i < 20; i++ {
        m[i] = i
        fmt.Printf("添加元素 %d: 元素数=%d, 桶数估计=2^%d\n", i, len(m), bucketCount(len(m)))
    }
}
// 估算桶数量对应的B值
func bucketCount(n int) uint8 {
    B := uint8(0)
    for 1 << B < uintptr(n/6.5) {
        B++
    }
    return B
}
//结果
添加元素 0: 元素数=1, 桶数估计=2^1
添加元素 1: 元素数=2, 桶数估计=2^1
添加元素 2: 元素数=3, 桶数估计=2^1  // 触发扩容（3 > 2*6.5/2=6.5/2≈3.25）
添加元素 3: 元素数=4, 桶数估计=2^2
...
添加元素 14: 元素数=15, 桶数估计=2^2 // 触发扩容（15 > 4*6.5=26）
添加元素 15: 元素数=16, 桶数估计=2^3
```
5.关键点总结：
1）.负载因子平衡：
负载因子 6.5 是时间和空间的平衡点，过高会增加冲突，过低会浪费内存。
2）.渐进式扩容优势：
避免一次性扩容的长时间停顿，保证 map 操作的响应性。
3）.并发限制：
map 不是并发安全的，扩容期间若多个 goroutine 同时操作，可能导致 panic。
4）.预分配优化：
若预先知道 map 的大致元素数量，可通过 make(map[T]U, size) 预分配，减少扩容次数。
6.与切片扩容的对比
| 特性 | map 扩容 | 切片扩容 |
| :---: | :---: | :---:|
| 触发条件 | 负载因子 > 6.5 或溢出桶过多 | 元素数量超过容量 |
| 扩容方式 | 翻倍或等量，渐进式迁移 | 直接分配新数组，一次性复制 |
| 内存管理 | 更复杂（涉及旧桶逐步释放） | 简单（旧数组丢弃） |
| 并发安全 | 非并发安全，需手动同步 | 非并发安全，需手动同步 |	
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