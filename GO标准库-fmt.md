[TOC]

# fmt

# 一、向外输出

标准库`fmt`通过类似于C语言中的`printf`、`scanf`函数来实现格式化输出和输出，语法格式和C语言相差无几，但比C语言来得更简单。

## 1、Print系列

```go
func Print(a ...interface{}) (n int, err error)
func Printf(format string, a ...interface{}) (n int, err error)
func Println(a ...interface{}) (n int, err error)
```

`Print`系列函数会将内容输出到系统的标准输出。`Print`函数直接输出内容，`Printf`函数支持占位符，格式化输出字符串，`Println`函数泽是在输出的内容结尾添加一个换行符。

## 2、Fprint系列

```go
func Fprint(w io.Writer, a ...interface{}) (n int, err error)
func Fprintf(w io.Writer, format string, a ...interface{}) (n int, err error)
func Fprintln(w io.Writer, a ...interface{}) (n int, err error)
```

`Fprint`系列函数会将内容输出到一个io.Writer接口类型的变量w中，我们通常用这个函数往文件中写入内容。

```go
// 向标准输出写入内容
fmt.Fprintln(os.Stdout, "向标准输出写入内容")
fileObj, err := os.OpenFile("./tmp.txt", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644)
if err != nil {
	fmt.Println("打开文件出错，err:", err)
	return
}
name := "Kyf"
// 向打开的文件句柄中写入内容
fmt.Fprintf(fileObj, "往文件中写入信息：%s", name)
```

## 3、Sprint系列

```go
func Sprint(a ...interface{}) string
func Sprintf(format string, a ...interface{}) string
func Sprintln(a ...interface{}) string
```

`Sprint`系列函数会把传入的数据生成并返回一个字符串。

```go
p := Person{Name: "Kfy", Sex: "男", Age: 26}
s := fmt.Sprintf("Name: %s, Sex: %s, Age: %d", p.Name, p.Sex, p.Age)
fmt.Println(s)
```

## 4、Errorf

```
func Errorf(format string, a ...interface{}) error
```

`Errorf`函数根据format参数生成格式化字符串并返回一个包含该字符串的错误。我们通常使用这种方式来自定义错误，例如：

```go
err := fmt.Errorf("自定义错误")
```

在Go1.13版本还为`fmt.Errorf`函数新加了一个`%w`占位符用来生成一个可以包裹Error的Wrapping Error。

```go
e := errors.New("原始错误")
w := fmt.Errorf("修饰了一个错误%w", e)
```

# 二、占位符

```go
p := Person{Name: "Kfy", Sex: "男", Age: 26}
m := map[string]int{"A": 1}
b := true
i := -255
c := 65
s := "ABC"
f := 123.45
```

## 1、通用占位符

| 占位符 | 说明               | 举例                   | 输出                     |
| :----- | :----------------- | ---------------------- | :----------------------- |
| %%     | 打印百分号%        | fmt.Printf("%%\n")     | %                        |
| %v     | 打印字面值         | fmt.Printf("%v\n", p)  | {Kfy 男 26}              |
| %+v    | 打印结构体的成员名 | fmt.Printf("%+v\n", p) | {Name:Kfy Sex:男 Age:26} |
| %#v    | 打印值的表达式     | fmt.Printf("%#v\n", m) | map[string]int{"A":1}    |
| %T     | 打印值的类型       | fmt.Printf("%T\n", m)  | map[string]int           |

## 2、布尔型占位符

| 占位符 | 说明       | 举例                  | 输出 |
| :----- | :--------- | --------------------- | :--- |
| %t     | 打印布尔值 | fmt.Printf("%t\n", b) | true |

## 3、整型占位符

| 占位符 | 说明                                           | 举例                   | 输出     |
| :----- | :--------------------------------------------- | ---------------------- | :------- |
| %b     | 打印二进制                                     | fmt.Printf("%b\n", i)  | 11111111 |
| %d     | 打印十进制                                     | fmt.Printf("%d\n", i)  | 255      |
| %o     | 打印八进制                                     | fmt.Printf("%o\n", i)  | 377      |
| %O     | 打印八进制，以0o打头                           | fmt.Printf("%O\n", i)  | 0o377    |
| %x     | 打印16进制，小写字母a-f                        | fmt.Printf("%x\n", i)  | ff       |
| %X     | 打印16进制，大写字母A-F                        | fmt.Printf("%X\n", i)  | FF       |
| %U     | 打印Unicode格式                                | fmt.Printf("%U\n", 65) | U+0041   |
| %q     | 打印单引号围绕的字符字面值，由Go语法安全地转义 | fmt.Printf("%q\n", 65) | 'A'      |
| %c     | 打印Unicode字符                                | fmt.Printf("%c\n", 65) | A        |

## 4、浮点型占位符

| 占位符 | 说明                                                         | 举例                  | 输出         |
| :----- | :----------------------------------------------------------- | --------------------- | :----------- |
| %f     | 打印浮点型数值。%9f，表示宽度9，默认精度6；%9.2f，表示宽度9，精度2。 | fmt.Printf("%f", f)   | 123.450000   |
| %F     | 打印浮点型数值，等价于%F                                     | fmt.Printf("%F", f)   | 123.450000   |
| %e     | 科学计数法，如-1234.456e+78                                  | fmt.Printf("%e\n", f) | 1.234500e+02 |
| %E     | 科学计数法，如-1234.456E+78                                  | fmt.Printf("%E\n", f) | 1.234500E+02 |
| %g     | 根据实际情况采用%e或%f格式（以获得更简洁、准确的输出）       | fmt.Printf("%g\n", f) | 123.45       |
| %G     | 根据实际情况采用%E或%F格式（以获得更简洁、准确的输出）       | fmt.Printf("%G\n", f) | 123.45       |

## 5、字符串占位符

| 占位符 | 说明                                       | 举例                  | 输出  |
| :----- | :----------------------------------------- | --------------------- | :---- |
| %s     | 打印字符串                                 | fmt.Printf("%s\n", s) | ABC   |
| %q     | 打印双引号围绕的字符串，由Go语法安全地转义 | fmt.Printf("%q\n", s) | "ABC" |

## 6、指针占位符

| 占位符 | 说明                                     | 举例                  | 输出         |
| :----- | :--------------------------------------- | --------------------- | :----------- |
| %p     | 打印十六进制，以0x开头，常用作取变量地址 | fmt.Printf("%p\n",&s) | 0xc000098210 |

## 7、其他

| 占位符 | 说明                                                         | 举例                                                         | 输出                           |
| :----- | :----------------------------------------------------------- | ------------------------------------------------------------ | :----------------------------- |
| +      | 总是输出数值的正负号；<br />对%+q会输出ASCII编码；           | fmt.Printf("%+d\n", i)<br />fmt.Printf("%+q\n", "中国")      | +255<br />"\u4e2d\u56fd"       |
| -      | 默认的右对齐输出切换为左对齐输出                             | fmt.Printf("%5d\n", i)<br />fmt.Printf("%-5d\n", i)          | 255<br />255                   |
| #      | 八进制前加0（%#o）;<br />十六进制前加0x（%#x）或0X（%#X）;<br />指针去掉0x前缀（%#p）; | fmt.Printf("%#o\n", i)<br />fmt.Printf("%#\n", i)<br />fmt.Printf("%#p\n",&s) | 0377<br />0xff<br />c000098210 |
| ' '    | 对数值，正数前加空格而负数前加负号；<br />对字符串采用%x或%X时（% x或% X）会给各打印的字节之间加空格； | fmt.Printf("% d\n", i)<br />fmt.Printf("% d\n", -i)          | 255<br />-255                  |
| 0      | 使用0而不是空格填充，对于数值类型会把填充的0放在正负号后面； | fmt.Printf("%05d\n", -i)                                     | -0255                          |

# 三、获取输入

标准库`fmt`下有`fmt.Scan`、`fmt.Scanf`、`fmt.Scanln`三个函数，可以在程序运行过程中从标准输入获取用户的输入。

## 1、Scan系列

```go
func Scan(a ...interface{}) (n int, err error)
func Scanf(format string, a ...interface{}) (n int, err error)
func Scanln(a ...interface{}) (n int, err error)
```

`Scan`函数从标准输入中扫描用户输入的数据，将以**空白符**分隔的数据分别存入指定的参数。**换行符视为空白符**。扫描直到参数足够就结束。

```go
func MyFmt() {
	var (
		name string
		sex  string
		age  int
	)
	fmt.Scan(&name, &sex, &age)
	fmt.Printf("name: %s, sex: %s, age: %d\n", name, sex, age)
}

// ➜ kaiyuanfan@kaiyuanfandeMacBook-Pro offcial_lib go run main.go
// Kyf 男 26
// name: Kyf, sex: 男, age: 26
```

`Scanf`函数从标准输入中**根据定义好的格式**扫描用户输入的数据，将数据分别存入指定的参数。扫描遇到换行符结束。

```go
	var (
		name string
		sex  string
		age  int
	)
	fmt.Scanf("name:%s sex:%s age:%d", &name, &sex, &age)
	fmt.Printf("name: %s, sex: %s, age: %d\n", name, sex, age)

// ➜ kaiyuanfan@kaiyuanfandeMacBook-Pro offcial_lib go run main.go
// name:Kyf sex:男 age:26
// name: Kyf, sex: 男, age: 26
```

`Scanln`函数从标准输入中扫描用户输入的数据，扫描遇到换行符结束。

```go
func MyFmt() {
	var (
		name string
		sex  string
		age  int
	)
	fmt.Scanln(&name, &sex, &age)
	fmt.Printf("name: %s, sex: %s, age: %d\n", name, sex, age)
}

// ➜ kaiyuanfan@kaiyuanfandeMacBook-Pro offcial_lib go run main.go
// Kyf 男 26
// name: Kyf, sex: 男, age: 26
```

## 2、Fscan系列

```go
func Fscan(r io.Reader, a ...interface{}) (n int, err error)
func Fscanln(r io.Reader, a ...interface{}) (n int, err error)
func Fscanf(r io.Reader, format string, a ...interface{}) (n int, err error)
```

这几个函数功能分别类似于`fmt.Scan`、`fmt.Scanf`、`fmt.Scanln`三个函数，只不过它们不是从标准输入中读取数据而是从`io.Reader`中读取数据。

## 3、Sscan系列

```go
func Sscan(str string, a ...interface{}) (n int, err error)
func Sscanln(str string, a ...interface{}) (n int, err error)
func Sscanf(str string, format string, a ...interface{}) (n int, err error)
```

这几个函数功能分别类似于`fmt.Scan`、`fmt.Scanf`、`fmt.Scanln`三个函数，只不过它们不是从标准输入中读取数据而是从指定字符串中读取数据。

