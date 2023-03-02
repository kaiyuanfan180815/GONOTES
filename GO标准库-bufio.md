
# bufio

# 一、原理

实际上io操作的效率不低，造成io效率低下的原因往往是频繁地访问本地磁盘中的文件，因此`bufio`标准库利用缓存区，在读操作过程中，先把文件内容读到缓冲区（内存），之后程序就可以直接从缓冲区中读取数据，减少了程序对本地磁盘的访问次数，以提高读效率。而对于写操作，亦是同理。

![image-20220405154614820](/Users/kaiyuanfan/Library/Application Support/typora-user-images/image-20220405154614820.png)

* 读操作：

```go
func NewReader(rd io.Reader) *Reader
func (b *Reader) Read(p []byte) (n int, err error)
type Reader struct {
	buf          []byte
	rd           io.Reader
	r, w         int
	err          error
	lastByte     int
	lastRuneSize int
}
```

1. 当`buf`中有数据时，将`buf`的数据全部填入`p`中，并清空`buf`；
2. 当`buf`中没有数据，且`len(p)>len(buf)`时，即程序一次读取的内容大小比缓存区的大小还要大，程序可直接读取文件；
3. 当`buf`中没有数据，且`len(p)<len(buf)`时，即程序一次读取的内容大小比缓存区的大小还要小，先把文件中的内容读到`buf`中，再从`buf`填入`p`；
4. 若步骤3一次读取没有读完buf中的内容，则继续循环至`buf`所有数据都已填入过`p`中，最后清空`buf`。

附上源码：

```go
// Read reads data into p.
// It returns the number of bytes read into p.
// The bytes are taken from at most one Read on the underlying Reader,
// hence n may be less than len(p).
// To read exactly len(p) bytes, use io.ReadFull(b, p).
// At EOF, the count will be zero and err will be io.EOF.
func (b *Reader) Read(p []byte) (n int, err error) {
	n = len(p)
	if n == 0 {
		return 0, b.readErr()
	}
	if b.r == b.w {
		if b.err != nil {
			return 0, b.readErr()
		}
		if len(p) >= len(b.buf) {
			// Large read, empty buffer.
			// Read directly into p to avoid copy.
			n, b.err = b.rd.Read(p)
			if n < 0 {
				panic(errNegativeRead)
			}
			if n > 0 {
				b.lastByte = int(p[n-1])
				b.lastRuneSize = -1
			}
			return n, b.readErr()
		}
		// One read.
		// Do not use b.fill, which will loop.
		b.r = 0
		b.w = 0
		n, b.err = b.rd.Read(b.buf)
		if n < 0 {
			panic(errNegativeRead)
		}
		if n == 0 {
			return 0, b.readErr()
		}
		b.w += n
	}

	// copy as much as we can
	n = copy(p, b.buf[b.r:b.w])
	b.r += n
	b.lastByte = int(b.buf[b.r-1])
	b.lastRuneSize = -1
	return n, nil
}
```

* 写操作：

```go
func NewWriter(w io.Writer) *Writer
func (b *Writer) Write(p []byte) (nn int, err error)
type Writer struct {
	err error
	buf []byte
	n   int
	wr  io.Writer
}
```

1. 判断`buf`中剩余容量是否可以放下`p`，若能，直接将`p`填入`buf`；
2. 如果`buf`剩余容量无法放下`p`，且此时`buf`中没有数据，则直接将`p`写入文件；
3. 如果`buf`剩余容量无法放下p，且此时`buf`中有数据，则将`p`尽可能地填入`buf`，当`buf`满了，就把`buf`中的数据写入文件，并`buf`；
4. 此时`p`中还有数据，则进行上述步骤1~2；

附上源码：

```go
// Write writes the contents of p into the buffer.
// It returns the number of bytes written.
// If nn < len(p), it also returns an error explaining
// why the write is short.
func (b *Writer) Write(p []byte) (nn int, err error) {
	for len(p) > b.Available() && b.err == nil {
		var n int
		if b.Buffered() == 0 {
			// Large write, empty buffer.
			// Write directly from p to avoid copy.
			n, b.err = b.wr.Write(p)
		} else {
			n = copy(b.buf[b.n:], p)
			b.n += n
			b.Flush()
		}
		nn += n
		p = p[n:]
	}
	if b.err != nil {
		return nn, b.err
	}
	n := copy(b.buf[b.n:], p)
	b.n += n
	nn += n
	return nn, nil
}
```

# 二、读文件

```go
func NewReaderSize(rd io.Reader, size int) *Reader
// 相当于NewReader(rd, 4096)
func NewReader(rd io.Reader) *Reader
func (b *Reader) Read(p []byte) (n int, err error)
func (b *Reader) ReadBytes(delim byte) (line []byte, err error)
func (b *Reader) ReadString(delim byte) (string, error)
// Peek返回缓存的一个切片，该切片引用缓存中前n个字节的数据。
// 该操作不会将数据读出，只是引用，引用的数据在下一次读取操作之前是有效的。如果切片长度小于n，则返回一个错误信息说明原因。
// 如果n大于缓存的总大小，则返回ErrBufferFull。
func (b *Reader) Peek(n int) ([]byte, error)
// Buffered 返回缓存中未读取的数据的长度。
func (b *Reader) Buffered() int
```

`bufio`标准库通过`NewReader`函数接受一个`io.Reader`接口类型的对象，返回一个`Reader`类型的指针，而`io.Reader`接口包含了以下的方法

```go
type Reader interface {
	Read(p []byte) (n int, err error)
}
```

`Read`方法也是老朋友了，我们通过`os`标准库中的`File`对象就实现了`Read`方法，也就是说`File`类型是`io.Reader`接口的实现，因此我们就可以通过把`os.File`对象传入`NewReader`函数来获取`Reader`类型的指针，再操作后续的文件的读操作。

`Reader`对象除了实现了`Read`方法以外，还提供了`ReadLine`，`ReadBytes`，`ReadString`方法来按需读取文件。

具体可以看下面的示例代码：

```go
func MyBufio() {
	path, _ := os.Getwd()
	path += "/myBufio"
	os.Chdir(path)
	f, err := os.OpenFile("tmp.txt", os.O_CREATE|os.O_RDWR|os.O_APPEND, fs.ModePerm)
	if err != nil {
		fmt.Printf("文件打开失败, %v\n", err)
		return
	}
	defer f.Close()
	reader := bufio.NewReader(f)
	bs := make([]byte, 3)
	for {
		n, err := reader.Read(bs)
		// str, err := reader.ReadString('\n')
		if err == io.EOF {
			fmt.Println("读取文件完成")
			break
		} else if err != nil {
			fmt.Printf("读取文件失败, %v\n", err)
			return
		}
		fmt.Print(string(bs[:n]))
		// fmt.Print(str)
	}
}
```

# 三、写文件

```go
func NewWriterSize(wr io.Writer, size int) *Writer
// NewWriter相当于NewWriterSize(wr, 4096)
func NewWriter(wr io.Writer) *Writer
// WriteString功能同Write，只不过写入的是字符串
func (b *Writer) WriteString(s string) (int, error)
// WriteRune向b写入r的UTF-8编码，返回r的编码长度
func (b *Writer) WriteRune(r rune) (size int, err error)
// Flush将缓存中的数据提交到底层的io.Writer中
func (b *Writer) Flush() error
// Available返回缓存中未使用的空间的长度
func (b *Writer) Available() int
// Buffered返回缓存中未提交的数据的长度
func (b *Writer) Buffered() int
// Reset将b的底层Writer重新指定为w，同时丢弃缓存中的所有数据，复位所有标记和错误信息。相当于创建了一个新的bufio.Writer
func (b *Writer) Reset(w io.Writer)
```

和读操作类似，`bufio`标准库通过`NewWriter`函数接受一个`io.Writer`接口类型的对象，返回一个`Writer`类型的指针，而`io.Writer`接口包含了以下的方法

```go
type Writer interface {
	Write(p []byte) (n int, err error)
}
```

同样，我们可以把`os.File`对象传入`NewWriter`函数来获取`Writer`类型的指针，再操作后续的文件的写操作。

具体可以看下面的示例代码：

```go
func MyBufio() {
	path, _ := os.Getwd()
	path += "/myBufio"
	os.Chdir(path)

	f1, err := os.OpenFile("tmp1.txt", os.O_CREATE|os.O_WRONLY|os.O_APPEND, fs.ModePerm)
	if err != nil {
		fmt.Printf("打开文件失败, %v\n", err)
	}
	writer := bufio.NewWriter(f1)
	str := "Helloworld!"
	writer.WriteString(str + "\n")
	writer.Flush()
}
```

