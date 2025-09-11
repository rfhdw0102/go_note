## 文件
**文件的常用相关操作**

打开文件/关闭文件
os.Open(name string)
os.OpenFile(name string, flag int, perm FileMode)
close(file)

其中os.OpenFile()里的二个参数有以下几种类型
![...](file_img/File.png)

```go
        //os.Open()方式只能以读的方式打开文件
        file, err := os.Open("./main.go")
        file, err := os.OpenFile("file/aaa.txt", os.O_CREATE|os.O_RDWR, 0666)
        //文件一定要及时关闭
        defer close(file)
```
创建目录/获取当前相对目录
```go
	//创建目录
	err := os.Mkdir("D:/file", 0755)
	if err != nil {
		log.Println("创建目录失败...")
	}
	//获取相对目录
	dirFile, _ := os.Getwd()
	fmt.Println(dirFile)
```
判断文件是否存在/删除文件/修改文件名
```go
	if _, err := os.Stat("file/aaa.txt"); err == nil {
		// 文件存在
		fmt.Println("文件存在...")
	}
	//删除文件
	//在windows系统，文件必须被关闭之后才能被删除
	err = os.Remove("file/aaa.txt")
	if err != nil {
		fmt.Println("删除文件失败...")
	}
	//更改文件名
	//文件只有在被关闭之后才能被更改名字
	err = os.Rename("file/aaa.txt", "file/abc.txt")
	if err != nil {
		fmt.Println("更改文件名失败...")
	}
```
写入/读出内容
```go
        file, err := os.OpenFile("file/aaa.txt", os.O_CREATE|os.O_RDWR, 0666)
        defer close(file)
	//写入内容
	writer := bufio.NewWriter(file)
	newReader := bufio.NewReader(os.Stdin)
	for {
		readMsg, err := newReader.ReadString('\n')
		readMsg = strings.TrimSpace(readMsg)
		if readMsg == "quit" {
			break
		}
		if err != nil {
			fmt.Println("终端输入信息接收失败...")
			return
		}
		_, err = writer.WriteString(readMsg + "\n")
		if err != nil {
			fmt.Println("写入文件信息失败...")
			return
		}
	}
	//读出内容
	reader := bufio.NewReader(file)
	for {
		message, err := reader.ReadString('\n')
		if err == io.EOF {
			break
		} else if err != nil {
			fmt.Println("")
			return
		}
		fmt.Println(message)
	}
        //一定要关闭缓冲，不然文件中啥都没有
	writer.Flush()

```
