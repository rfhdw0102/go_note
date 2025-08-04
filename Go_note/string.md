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