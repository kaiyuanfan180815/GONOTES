[TOC]

# 写在前面

下载地址：https://golang.google.cn/dl/

选择Stable版本，官方分别提供了源码包，压缩包，可执行安装包三种，可根据自己的操作系统下载安装即可。非线上服务器应该基本上都是MacOS或者Windows，建议.pkg或者.msi，傻瓜式安装直接了当。

![image-20220322211242980](/Users/kaiyuanfan/Library/Application Support/typora-user-images/image-20220322211242980.png)

安装完之后，配置GOROOT、GOPATH、GOBIN三个环境变量，最后执行`go verion` 验证是否安装成功即可。

```shell
➜ kaiyuanfan@kaiyuanfandeMacBook-Pro go cat ~/.bash_profile
export GOROOT=/usr/local/go
export GOPATH=/Users/kaiyuanfan/go
export GOBIN=$GOROOT/bin
export PATH=$PATH:$GOBIN
➜ kaiyuanfan@kaiyuanfandeMacBook-Pro go go version
go version go1.17.7 darwin/amd64
```

# 一、环境变量

下面讲一讲GOROOT、GOPATH、GOBIN这三个环境变量的含义和用途。

## 1、GOROOT

包含Go语言的安装根目录的路径。对于MacOS而言，默认路径是/usr/local/go。

```shell
➜ kaiyuanfan@kaiyuanfandeMacBook-Pro go pwd
/usr/local/go
➜ kaiyuanfan@kaiyuanfandeMacBook-Pro go ls -lrt
total 384
-rw-r--r--    1 root  wheel      52  2 10 02:14 codereview.cfg
-rw-r--r--    1 root  wheel     397  2 10 02:14 SECURITY.md
-rw-r--r--    1 root  wheel    1480  2 10 02:14 README.md
-rw-r--r--    1 root  wheel    1303  2 10 02:14 PATENTS
-rw-r--r--    1 root  wheel    1479  2 10 02:14 LICENSE
-rw-r--r--    1 root  wheel  107225  2 10 02:14 CONTRIBUTORS
-rw-r--r--    1 root  wheel    1339  2 10 02:14 CONTRIBUTING.md
-rw-r--r--    1 root  wheel   55782  2 10 02:14 AUTHORS
drwxr-xr-x   14 root  wheel     448  2 10 02:14 misc
drwxr-xr-x    3 root  wheel      96  2 10 02:14 lib
drwxr-xr-x    6 root  wheel     192  2 10 02:14 doc
drwxr-xr-x   23 root  wheel     736  2 10 02:14 api
drwxr-xr-x   69 root  wheel    2208  2 10 02:14 src
drwxr-xr-x  347 root  wheel   11104  2 10 02:14 test
-rw-r--r--    1 root  wheel       8  2 10 02:14 VERSION
drwxr-xr-x    6 root  wheel     192  2 10 02:16 pkg
drwxr-xr-x    4 root  wheel     128  2 11 03:54 bin
```

## 2、GOPATH

包含若干个工作区目录的路径。

## 3、GOBIN

包含用于放置Go程序生成的可执行文件的路径。

# 二、代码组织

## 1、工作区

我们可以把GOPATH理解成Go语言的工作目录，GOPATH变量的值可以可以是一个目录的路径，也可以是多个目录的路径，每个目录都代表一个Go语言的工作区。包含src、pkg、bin三个子目录：

### 1.1、src

放置Go源码文件(.go)的目录。

### 1.2、pkg

放置库源码文件编译后生成的归档文件(.a)的目录。

<u>**补充说明：归档文件在Linux下就是扩展名是.a的文件，也就是archive文件，相当于C语言中的.obj，是程序编译后生成的静态库文件，可执行文件的生成就是通过链接此类文件。**</u>

### 1.3、bin

放置可执行文件的目录，仅在GOBIN变量未设置时生效。

## 2、代码包

Go语言的源码文件是以代码包为基本组织单位的。其中src目录下每一个子目录就是一个代码包。一个代码包中可以包含任意个以.go为扩展名的Go源码文件，这些源码文件都需要被声明属于一个代码包。比如说下面的a，b，c三个源码文件就构成了一个example代码包。

```go
// src/example/a.go
package example
func a(){
  // ......
}
// src/example/b.go
package example
func b(){
  // ......
}
// src/example/c.go
package example
func c(){
  // ......
}
```

## 3、源码文件

### 3.1、命令源码文件

**如果一个源码文件声明属于main包，并且包含一个无参数声明且无结果声明的main函数，那么它就是命令源码文件。**命令源码文件是程序的运行入口，是每个可独立运行的程序必须拥有的。我们可以通过构建或安装，生成与其对应的可执行文件，一般会与该命令源码文件的直接父目录同名。

### 3.2、库源码文件

非命令源码文件以外的源码文件，此类文件被安装后，会生成相应的归档文件。

### 3.3、测试源码文件

以 _test.go 为后缀命名的源码文件，并且必须包含 Test 或者 Benchmark 名称前缀的函数

```go
// 名称以Test为前缀的函数，只能接受 *testing.T 的参数，这种测试函数是功能测试函数。
func TestXXX( t *testing.T) {

}
// 名称以Benchmark为前缀的函数，只能接受 *testing.B 的参数，这种测试函数是性能测试函数。
func BenchmarkXXX( b *testing.B) {

}
```

# 三、命令行

```
➜ kaiyuanfan@kaiyuanfandeMacBook-Pro utils go help      
Go is a tool for managing Go source code.

Usage:

        go <command> [arguments]

The commands are:

        bug         start a bug report
        build       compile packages and dependencies
        clean       remove object files and cached files
        doc         show documentation for package or symbol
        env         print Go environment information
        fix         update packages to use new APIs
        fmt         gofmt (reformat) package sources
        generate    generate Go files by processing source
        get         download and install packages and dependencies
        install     compile and install packages and dependencies
        list        list packages or modules
        mod         module maintenance
        run         compile and run Go program
        test        test packages
        tool        run specified go tool
        version     print Go version
        vet         report likely mistakes in packages

Use "go help <command>" for more information about a command.
```

其中和编译相关的是run、build、install这三个。

## 1、go run

```shell
go run
-n		# 查看构建的执行步骤但不执行构建
-x		# 查看构建的执行步骤且执行构建
-work	# 查看构建时生成的临时目录且于构建后保留
```

go run执行的过程就是首先创建一个临时目录。其次把命令源码文件依赖的库源码文件对应的归档文件清单写到该目录下的importcfg.link文件中。然后在临时目录下创建一个exe子目录，并通过把importcfg.link文件给出的归档文件编译并链接起来形成可执行文件。最后执行可执行文件。<u>**需要注意的是只有初次执行过程或源码文件有改动时候才会触发编译，否则直接链接已生成的归档文件生成可执行文件。**</u>

```shell
WORK=/var/folders/8q/zryq_4912dldhxbs69t_8d5c0000gn/T/go-build1522530012
mkdir -p $WORK/b001/
cat >$WORK/b001/importcfg << 'EOF' # internal
# import config
packagefile fmt=/usr/local/go/pkg/darwin_amd64/fmt.a
packagefile runtime=/usr/local/go/pkg/darwin_amd64/runtime.a
EOF
cd /Users/kaiyuanfan/go/src/demo
# 只有初次执行才会有compile编译库源码文件的过程
/usr/local/go/pkg/tool/darwin_amd64/compile -o $WORK/b001/_pkg_.a -trimpath "$WORK/b001=>" -p main -complete -buildid Mg56XKw2fN5Q14yc0w63/Mg56XKw2fN5Q14yc0w63 -dwarf=false -goversion go1.17.7 -D _/Users/kaiyuanfan/go/src/demo -importcfg $WORK/b001/importcfg -pack -c=4 ./helloworld.go
/usr/local/go/pkg/tool/darwin_amd64/buildid -w $WORK/b001/_pkg_.a # internal
cp $WORK/b001/_pkg_.a /Users/kaiyuanfan/Library/Caches/go-build/fe/fe1dd607ad9d499040579b8d3966c342c03b00aeef1dfc1d2d4ba0c601f0b716-d # internal
cat >$WORK/b001/importcfg.link << 'EOF' # internal
packagefile command-line-arguments=$WORK/b001/_pkg_.a
packagefile fmt=/usr/local/go/pkg/darwin_amd64/fmt.a
packagefile runtime=/usr/local/go/pkg/darwin_amd64/runtime.a
packagefile errors=/usr/local/go/pkg/darwin_amd64/errors.a
packagefile internal/fmtsort=/usr/local/go/pkg/darwin_amd64/internal/fmtsort.a
packagefile io=/usr/local/go/pkg/darwin_amd64/io.a
packagefile math=/usr/local/go/pkg/darwin_amd64/math.a
packagefile os=/usr/local/go/pkg/darwin_amd64/os.a
packagefile reflect=/usr/local/go/pkg/darwin_amd64/reflect.a
packagefile strconv=/usr/local/go/pkg/darwin_amd64/strconv.a
packagefile sync=/usr/local/go/pkg/darwin_amd64/sync.a
packagefile unicode/utf8=/usr/local/go/pkg/darwin_amd64/unicode/utf8.a
packagefile internal/abi=/usr/local/go/pkg/darwin_amd64/internal/abi.a
packagefile internal/bytealg=/usr/local/go/pkg/darwin_amd64/internal/bytealg.a
packagefile internal/cpu=/usr/local/go/pkg/darwin_amd64/internal/cpu.a
packagefile internal/goexperiment=/usr/local/go/pkg/darwin_amd64/internal/goexperiment.a
packagefile runtime/internal/atomic=/usr/local/go/pkg/darwin_amd64/runtime/internal/atomic.a
packagefile runtime/internal/math=/usr/local/go/pkg/darwin_amd64/runtime/internal/math.a
packagefile runtime/internal/sys=/usr/local/go/pkg/darwin_amd64/runtime/internal/sys.a
packagefile internal/reflectlite=/usr/local/go/pkg/darwin_amd64/internal/reflectlite.a
packagefile sort=/usr/local/go/pkg/darwin_amd64/sort.a
packagefile math/bits=/usr/local/go/pkg/darwin_amd64/math/bits.a
packagefile internal/itoa=/usr/local/go/pkg/darwin_amd64/internal/itoa.a
packagefile internal/oserror=/usr/local/go/pkg/darwin_amd64/internal/oserror.a
packagefile internal/poll=/usr/local/go/pkg/darwin_amd64/internal/poll.a
packagefile internal/syscall/execenv=/usr/local/go/pkg/darwin_amd64/internal/syscall/execenv.a
packagefile internal/syscall/unix=/usr/local/go/pkg/darwin_amd64/internal/syscall/unix.a
packagefile internal/testlog=/usr/local/go/pkg/darwin_amd64/internal/testlog.a
packagefile internal/unsafeheader=/usr/local/go/pkg/darwin_amd64/internal/unsafeheader.a
packagefile io/fs=/usr/local/go/pkg/darwin_amd64/io/fs.a
packagefile sync/atomic=/usr/local/go/pkg/darwin_amd64/sync/atomic.a
packagefile syscall=/usr/local/go/pkg/darwin_amd64/syscall.a
packagefile time=/usr/local/go/pkg/darwin_amd64/time.a
packagefile unicode=/usr/local/go/pkg/darwin_amd64/unicode.a
packagefile internal/race=/usr/local/go/pkg/darwin_amd64/internal/race.a
packagefile path=/usr/local/go/pkg/darwin_amd64/path.a
EOF
mkdir -p $WORK/b001/exe/
cd .
/usr/local/go/pkg/tool/darwin_amd64/link -o $WORK/b001/exe/helloworld -importcfg $WORK/b001/importcfg.link -s -w -buildmode=exe -buildid=dTQeoZQ9MLZ0CWEXW9l3/Mg56XKw2fN5Q14yc0w63/L5IE5kd5WRWUERZhD0YU/dTQeoZQ9MLZ0CWEXW9l3 -extld=clang $WORK/b001/_pkg_.a
$WORK/b001/exe/helloworld
helloworld!

➜ kaiyuanfan@kaiyuanfandeMacBook-Pro demo tree /var/folders/8q/zryq_4912dldhxbs69t_8d5c0000gn/T/go-build2371469465                                                                   
/var/folders/8q/zryq_4912dldhxbs69t_8d5c0000gn/T/go-build2371469465
└── b001
    ├── exe
    │   └── helloworld
    └── importcfg.link

2 directories, 2 files
```

## 2、go build

go build的执行过程和go run相差无几，就是最后一步go run是执行临时目录exe下的可执行文件，而go build是将可执行文件挪到构建的命令源码文件所在的目录中。同样的，如果源码文件没有发生变动，后续的执行过程将不再执行。

## 3、go install

go install的执行过程和go build相差无几，就是最后一步go install将可执行文件从构建的命令源码文件所在的目录中挪到了GOBIN下。