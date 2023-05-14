# fmt

## Print

Print系列函数会将内容输出到系统的标准输出，区别在于 Print 函数直接输出内容，Printf 函数支持格式化输出字符串，Println 函数会在输出内容的结尾添加一个换行符。

```go
func Print(a ...interface{}) (n int, err error)
func Printf(format string, a ...interface{}) (n int, err error)
func Println(a ...interface{}) (n int, err error)
```

## Fprint

Fprint 系列函数会将内容输出到一个 io.Writer 接口类型的变量 w 中，我们通常用这个函数往文件中写入内容。

```go
func Fprint(w io.Writer, a ...interface{}) (n int, err error)
func Fprintf(w io.Writer, format string, a ...interface{}) (n int, err error)
func Fprintln(w io.Writer, a ...interface{}) (n int, err error) 
```

举例：

```go
// 向标准输出写入内容
fmt.Fprintln(os.Stdout, "向标准输出写入内容")
fileObj, err := os.OpenFile("./xx.txt", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644)
if err != nil {
    fmt.Println("打开文件出错，err:", err)
    return
}
name := "枯藤"
// 向打开的文件句柄中写入内容
fmt.Fprintf(fileObj, "往文件中写如信息：%s", name)
```

## Sprint

Sprint 系列函数会把传入的数据生成并返回一个字符串。

```go
func Sprint(a ...interface{}) string
func Sprintf(format string, a ...interface{}) string
func Sprintln(a ...interface{}) string
```

举例：

```go
s1 := fmt.Sprint("枯藤")
name := "枯藤"
age := 18
s2 := fmt.Sprintf("name:%s,age:%d", name, age)
s3 := fmt.Sprintln("枯藤")
fmt.Println(s1, s2, s3)
```

## Errorf

Errorf 函数根据 format 参数生成格式化字符串并返回一个包含该字符串的错误。

```go
func Errorf(format string, a ...interface{}) error
```

## 格式化占位符

*printf 系列函数都支持 format 格式化参数，在这里我们按照占位符将被替换的变量类型划分，方便查询和记忆。

<span style='color:red;font-size:13px'>注：红色字体的比较常用</span>

### 通用占位符

| 占位符                             | 说明                               |
| :--------------------------------- | :--------------------------------- |
| <span style='color:red'>%v</span>  | 值的默认格式表示                   |
| %+v                                | 类似%v，但输出结构体时会添加字段名 |
| <span style='color:red'>%#v</span> | 值的Go语法表示                     |
| <span style='color:red'>%T</span>  | 打印值的类型                       |
| %%                                 | 百分号                             |

### 布尔型

| 占位符                            | 说明        |
| :-------------------------------- | :---------- |
| <span style='color:red'>%t</span> | true或false |

### 整型

| 占位符 | 说明                                                         |
| :----- | :----------------------------------------------------------- |
| %b     | 表示为二进制                                                 |
| %c     | 该值对应的unicode码值                                        |
| %d     | 表示为十进制                                                 |
| %o     | 表示为八进制                                                 |
| %x     | 表示为十六进制，使用a-f                                      |
| %X     | 表示为十六进制，使用A-F                                      |
| %U     | 表示为Unicode格式：U+1234，等价于”U+%04X”                    |
| %q     | 该值对应的单引号括起来的go语法字符字面值，必要时会采用安全的转义表示 |

### 浮点数与复数

| 占位符 | 说明                                                   |
| :----- | :----------------------------------------------------- |
| %b     | 无小数部分、二进制指数的科学计数法，如-123456p-78      |
| %e     | 科学计数法，如-1234.456e+78                            |
| %E     | 科学计数法，如-1234.456E+78                            |
| %f     | 有小数部分但无指数部分，如123.456                      |
| %F     | 等价于%f                                               |
| %g     | 根据实际情况采用%e或%f格式（以获得更简洁、准确的输出） |
| %G     | 根据实际情况采用%E或%F格式（以获得更简洁、准确的输出） |

### 字符串和 []byte

| 占位符                            | 说明                                                         |
| :-------------------------------- | :----------------------------------------------------------- |
| <span style='color:red'>%s</span> | 直接输出字符串或者[]byte                                     |
| %q                                | 该值对应的双引号括起来的go语法字符串字面值，必要时会采用安全的转义表示 |
| %x                                | 每个字节用两字符十六进制数表示（使用a-f                      |
| %X                                | 每个字节用两字符十六进制数表示（使用A-F）                    |

### 字符串和 []byte

| 占位符                            | 说明                                                         |
| :-------------------------------- | :----------------------------------------------------------- |
| <span style='color:red'>%s</span> | 直接输出字符串或者[]byte                                     |
| %q                                | 该值对应的双引号括起来的go语法字符串字面值，必要时会采用安全的转义表示 |
| %x                                | 每个字节用两字符十六进制数表示（使用a-f                      |
| %X                                | 每个字节用两字符十六进制数表示（使用A-F）                    |

### 宽度标识符

宽度通过一个紧跟在百分号后面的十进制数指定，如果未指定宽度，则表示值时除必需之外不作填充。精度通过（可选的）宽度后跟点号后跟的十进制数指定。如果未指定精度，会使用默认精度；如果点号后没有跟数字，表示精度为0。举例如下

| 占位符 | 说明               |
| :----- | :----------------- |
| %f     | 默认宽度，默认精度 |
| %9f    | 宽度9，默认精度    |
| %.2f   | 默认宽度，精度2    |
| %9.2f  | 宽度9，精度2       |
| %9.f   | 宽度9，精度0       |

### 其他 flag

| 占位符 | 说明                                                         |
| :----- | :----------------------------------------------------------- |
| ’+’    | 总是输出数值的正负号；对%q（%+q）会生成全部是ASCII字符的输出（通过转义）； |
| ’ ‘    | 对数值，正数前加空格而负数前加负号；对字符串采用%x或%X时（% x或% X）会给各打印的字节之间加空格 |
| ’-’    | 在输出右边填充空白而不是默认的左边（即从默认的右对齐切换为左对齐）； |
| ’#’    | 八进制数前加0（%#o），十六进制数前加0x（%#x）或0X（%#X），指针去掉前面的0x（%#p）对%q（%#q），对%U（%#U）会输出空格和单引号括起来的go字面值； |
| ‘0’    | 使用0而不是空格填充，对于数值类型会把填充的0放在正负号后面； |

# Scan

## fmt.Scan

```go
func Scan(a ...interface{}) (n int, err error)
```

- Scan 从标准输入扫描文本，读取由空白符分隔的值保存到传递给本函数的参数中，换行符视为空白符。
- 本函数返回成功扫描的数据个数和遇到的任何错误。如果读取的数据个数比提供的参数少，会返回一个错误报告原因。

## fmt.Scanf

```go
func Scanf(format string, a ...interface{}) (n int, err error)
```

- Scanf 从标准输入扫描文本，根据format参数指定的格式去读取由空白符分隔的值保存到传递给本函数的参数中。
- 本函数返回成功扫描的数据个数和遇到的任何错误。

## fmt.Scanln

```go
func Scanln(a ...interface{}) (n int, err error) 
```

- Scanln 类似 Scan，它在遇到换行时才停止扫描。最后一个数据后面必须有换行或者到达结束位置。
- 本函数返回成功扫描的数据个数和遇到的任何错误。

## bufio.NewReader

有时候我们想完整获取输入的内容，而输入的内容可能包含空格，这种情况下可以使用bufio包来实现。

举例：

```go
func bufioDemo() {
    reader := bufio.NewReader(os.Stdin) // 从标准输入生成读对象
    fmt.Print("请输入内容：")
    text, _ := reader.ReadString('\n') // 读到换行
    text = strings.TrimSpace(text)
    fmt.Printf("%#v\n", text)
}
```

## Fscan

```go
func Fscan(r io.Reader, a ...interface{}) (n int, err error)
func Fscanln(r io.Reader, a ...interface{}) (n int, err error)
func Fscanf(r io.Reader, format string, a ...interface{}) (n int, err error)
```

## Sscan

```go
func Sscan(str string, a ...interface{}) (n int, err error)
func Sscanln(str string, a ...interface{}) (n int, err error)
func Sscanf(str string, format string, a ...interface{}) (n int, err error)
```

