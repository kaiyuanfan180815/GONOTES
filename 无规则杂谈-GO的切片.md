[TOC]

# 写在前面

王者已混上50颗星，赛季目标达成。写一篇杂谈庆祝下。背景是近期跟随部门开发大佬的步伐在学Go，然后日常睡不着，在遨游极客时间评论区（对于评论区，我的评价是神仙打架）的时候，看到这样一个评论"切片在使用的时候需要注意些什么"，垫高枕头想了许久，把自己目前能想到的东西无规则地写一下。

# 一、处处受限的数组

关于数组，我们这里简单带过。

* 初始化

  ```go
  // 声明一个长度为10的整型数据，可以根据需要调整长度和类型
  var a [10]int
  a = [10]int{0,1,2,3,4}
  ```

* 相关操作

  ```go
  // 获取数组长度，即当前数组内的元素个数
  len(a)
  //获取数组容量，即数组能够存放的最多元素的个数
  cap(a)
  // 通过下标访问数组中的某个元素
  item := a[i]
  // 遍历数组中的元素
  for i := 0; i <= len(a); i++ {
    fmt.Printf("index=%d,value=%d\n", i, a[i])
  }
  
  for i,v := range a {
    fmt.Printf("index=%d,value=%d\n", i, v)
  }
  ```

* 缺点

  从初始化，获取数组长度以及容量可以看出，数组的缺点是声明时就分配内存空间，长度不可变，元素类型单一（当然，如果声明的数组类型为[10]interface{}的，可以当我没说。但非要说的话空接口也是一种类型）

# 二、支持动态扩容的切片

再聊聊切片。切片是引用类型，其底层是数组。

* 初始化

  ```go
  // 这里直接声明一个指向a数组的切片
  s := a[:]
  ```

* 相关操作

  ```go
  // 获取切片长度，结果是底层数组的元素个数。
  len(s)
  //获取数组容量，结果是底层数组能够存放的最多元素的个数
  cap(s)
  
  // 假设s := a[start:end],那么len(s) = end - start, cap(s) = len(a) - start
  
  // 通过下标访问数组中的某个元素
  item := s[i]
  // 遍历数组中的元素
  for i := 0; i <= len(s); i++ {
    fmt.Printf("index=%d,value=%d\n", i, s[i])
  }
  for i,v := range s {
    fmt.Printf("index=%d,value=%d\n", i, v)
  }
  
  // 添加单个元素
  s = append(s, 5)
  // 添加多个元素
  s = append(s, 6，7)
  // 添加其他切片里的元素
  s = append(s, []int{8, 9}...)
  ```

  回到正题，"切片在使用的时候需要注意些什么"，我想到的是以下三点：

  * 扩容

    从不少的Go教学视频和极客专栏中看到，切片的扩容遵循以下3个原则：

    1. 通过append添加元素时，当元素个数超过底层数组的容量时，返回指向一个新的底层数组的切片。其长度为添加元素后的切片元素个数，而容量为原来的数组的两倍。
    2. 通过append添加元素时，当元素个数没有超过底层数组的容量时，返回原来的切片。其长度为添加元素后的切片个数，而容量不变。
    3. 通过append添加元素时，当元素个数已经超过1024，后续发生扩容时，容量不再翻倍，而是1.25倍，若扩一次不够，就再扩1.25倍，以此类推。

    **那么，真的如此吗？我们看看以下代码：**

    ```go
    s := a[:]
    
    // []int,0xc00001c150,[0 1 2 3 4],len=5,cap=5
    fmt.Printf("%T,%[1]p,%[1]v,len=%d,cap=%d\n", s, len(s), cap(s))
    s = append(s, 5)
    
    // []int,0xc00001a0f0,[0 1 2 3 4 5],len=6,cap=10
    fmt.Printf("%T,%[1]p,%[1]v,len=%d,cap=%d\n", s, len(s), cap(s))
    s = append(s, 6, 7)
    
    // []int,0xc00001a0f0,[0 1 2 3 4 5 6 7],len=8,cap=10
    fmt.Printf("%T,%[1]p,%[1]v,len=%d,cap=%d\n", s, len(s), cap(s))
    s = append(s, []int{8, 9}...)
    
    // []int,0xc00001a0f0,[0 1 2 3 4 5 6 7 8 9],len=10,cap=10
    fmt.Printf("%T,%[1]p,%[1]v,len=%d,cap=%d\n", s, len(s), cap(s))
    
    s1 := make([]int, 0)
    // 1015
    for i <= 10; i <= 1024; i++ {
      s1 = s1.append(i)
    }
    s = append(s, s1...)
    // ...,len=1025,cap=1184
    fmt.Printf("%T,%[1]p,%[1]v,len=%d,cap=%d\n", s, len(s), cap(s))
    ```

    从代码中看出，最后一次将切片s1的元素添加至切片s中之后，扩容切片的容量并非为640\*2=1280或是640\*1.25\*1.25\*1.25=1250，而是1184，那么1184是怎么来的呢？

    ```go
    // $GOROOT/src/runtime/slice.go
    func growslice(et *_type, old slice, cap int) slice {
    ...
            newcap := old.cap
            doublecap := newcap + newcap
            if cap > doublecap {
                    newcap = cap
            } else {
                    if old.cap < 1024 {
                            newcap = doublecap
                    } else {
                            for 0 < newcap && newcap < cap {
                                    newcap += newcap / 4
                            }
                            if newcap <= 0 {
                                    newcap = cap
                            }
                    }
            }
    ...
    }
    ```

    首先看看slice.go中的扩容函数growslice有上述这么一段代码，从这段代码中，我们可以看到上述说的新切片元素个数小于1024时扩容时容量翻倍，大于1024时翻1.25倍似乎没啥问题。但就在扩容函数growslice的下面还有这么一段代码：

    ```go
    // $GOROOT/src/runtime/slice.go
    func growslice(et *_type, old slice, cap int) slice {
    ...
            var overflow bool
            var lenmem, newlenmem, capmem uintptr
            switch {
            case et.size == 1:
                    lenmem = uintptr(old.len)
                    newlenmem = uintptr(cap)
                    capmem = roundupsize(uintptr(newcap))
                    overflow = uintptr(newcap) > maxAlloc
                    newcap = int(capmem)
            case et.size == sys.PtrSize:
                    lenmem = uintptr(old.len) * sys.PtrSize
                    newlenmem = uintptr(cap) * sys.PtrSize
                    capmem = roundupsize(uintptr(newcap) * sys.PtrSize)
                    overflow = uintptr(newcap) > maxAlloc/sys.PtrSize
                    newcap = int(capmem / sys.PtrSize)
            case isPowerOfTwo(et.size):
                    var shift uintptr
                    if sys.PtrSize == 8 {
                            // Mask shift for better code generation.
                            shift = uintptr(sys.Ctz64(uint64(et.size))) & 63
                    } else {
                            shift = uintptr(sys.Ctz32(uint32(et.size))) & 31
                    }
                    lenmem = uintptr(old.len) << shift
                    newlenmem = uintptr(cap) << shift
                    capmem = roundupsize(uintptr(newcap) << shift)
                    overflow = uintptr(newcap) > (maxAlloc >> shift)
                    newcap = int(capmem >> shift)
            default:
                    lenmem = uintptr(old.len) * et.size
                    newlenmem = uintptr(cap) * et.size
                    capmem, overflow = math.MulUintptr(et.size, uintptr(newcap))
                    capmem = roundupsize(capmem)
                    newcap = int(capmem / et.size)
            }
            if overflow || capmem > maxAlloc {
                    panic(errorString("growslice: cap out of range"))
            }
    ...
    }
    ```

    从这段代码中，我们可以看出，翻倍或是翻1.25倍只是用于计算一个预估容量，这个容量并非是新切片的最终容量，新切片的最终容量还要再经过一层计算。

    回到我们的疑问中来，切片s（当前长度为10，容量为10），添加切片s1中的1015个元素后，新切片的容量为1184是怎么计算的。

    ```go
    // 首先在第一段代码中，走的是下面这个分支，得到的newcap是1025
    ...
    				// old.cap = 10，cap = 1025
            newcap := old.cap
            doublecap := newcap + newcap
            if cap > doublecap {
                    newcap = cap
            }
    ...
    // 其次在第二段代码中，et.size = 8，是指切片中元素类型占用的字节数大小，这里元素的类型是int，因为我的操作系统是64位的，所以int相当于是int64，占8个字节。我们也可以通过反射确认下
            var i int = 1
    				fmt.Println(reflect.TypeOf(i).Size()) // 8
    				fmt.Println(sys.PtrSize) // 8 
    // 因此在第二段代码中，走的是以下的逻辑分支
            case et.size == sys.PtrSize:
            lenmem = uintptr(old.len) * sys.PtrSize
            newlenmem = uintptr(cap) * sys.PtrSize
            capmem = roundupsize(uintptr(newcap) * sys.PtrSize)
            overflow = uintptr(newcap) > maxAlloc/sys.PtrSize
            newcap = int(capmem / sys.PtrSize)
    ```

    确认了逻辑分支后，我们再看看分支中的计算逻辑，这里我们主要关注capmen和newcap的运算，涉及到roundupsize这个函数。我们从msize.go中找到了它的具体实现，并从sizeclass.go中找到了其实现时所用到的数据结构，从stubs.go中找到了实现用到的函数。

    ```go
    // $GOROOT/src/runtime/msize.go
    func roundupsize(size uintptr) uintptr {
            if size < _MaxSmallSize {
                    if size <= smallSizeMax-8 {
                            return uintptr(class_to_size[size_to_class8[divRoundUp(size, smallSizeDiv)]])
                    } else {
                            return uintptr(class_to_size[size_to_class128[divRoundUp(size-smallSizeMax, largeSizeDiv)]])
                    }
            }
            if size+_PageSize < size {
                    return size
            }
            return alignUp(size, _PageSize)
    }
    
    // $GOROOT/src/runtime/sizeclass.go
    // 定义了一组常量和数组
    
    // $GOROOT/src/runtime/stubs.go
    func divRoundUp(n, a uintptr) uintptr {
            return (n + a - 1) / a
    }
    ```

    我们代入到上述一系列代码中计算下。

    ```go
    // uintptr(newcap) * sys.PtrSize = uint(1025) * 8 = 8200，在roundupsize中满足size < _MaxSmallSize且size > smallSizeMax-8
    // return uintptr(class_to_size[size_to_class128[divRoundUp(size-smallSizeMax, largeSizeDiv)]])
    // 代入计算后可得到 capmem = 9472
    capmem = roundupsize(uintptr(newcap) * sys.PtrSize)
    // 代入计算后可得到 newcap = 1184
    newcap = int(capmem / sys.PtrSize)
    ```

    除了上述涉及到的代码外，还有其他的逻辑分支。在msize.go源码文件中，第一行就是"Malloc small size classes."，而roundupsize函数的注释则是"Returns size of the memory block that mallocgc will allocate if you ask for the size."，意为根据申请的空间大小来分配内存。

    以上可以看出，扩容并非想象中的翻多少倍那么简单，它与**操作系统位数、元素类型、追加个数**等都有关系。

    实际上我们用到的时候也不必过于关心底层的实现，但我们需要关注的是**"如果能够确定切片的容量范围，切片初始化时应该分配足够的容量空间。在append追加操作时，就不用再考虑扩容带来的性能损耗问题"。**

    ```go
    package SliceTest
    
    import "testing"
    
    func BenchmarkAppendFixCap(b *testing.B) {
    	for i := 0; i < b.N; i++ {
    		a := make([]int, 0, 1000)
    		for i := 0; i < 1000; i++ {
    			a = append(a, i)
    		}
    	}
    }
    
    func BenchmarkAppend(b *testing.B) {
    	for i := 0; i < b.N; i++ {
    		a := make([]int, 0)
    		for i := 0; i < 1000; i++ {
    			a = append(a, i)
    		}
    	}
    }
    ```

    通过以上的测试，也可以看出创建切片时候指定容量和不指定容量二者之间的优劣。

    ```bash
    ➜ kaiyuanfan@kaiyuanfandeMacBook-Pro slice_test go test -bench=. -benchmem
    goos: darwin
    goarch: amd64
    pkg: SliceTest
    cpu: Intel(R) Core(TM) i7-1068NG7 CPU @ 2.30GHz
    BenchmarkAppendFixCap-8          1917866               626.9 ns/op             0 B/op          0 allocs/op
    BenchmarkAppend-8                 336177              3440 ns/op           16376 B/op         11 allocs/op
    PASS
    ok      SliceTest       3.585s
    ```

  * 缩容

    ```go
    // 删除切片最后一个元素
    // s = []int{0,1,2,3,4,5,6,7,8,9}
    s := s[:len(s)-1]
    // []int,0xc00001a0f0,[0 1 2 3 4 5 6 7 8],len=9,cap=10
    fmt.Printf("%T,%[1]p,%[1]v,len=%d,cap=%d\n", s, len(s), cap(s))
    ```

    缩容的时候需要注意的是，上述代码虽然把切片末尾的元素删除了，但切片仍指向的是同一个底层数组，容量上没有发生变化，会形成资源浪费。应该重新生成新的切片，新的底层数组，通过copy操作把旧切片中的元素添加至新切片中，并把旧切片置为nil，让其回收。

    ```go
    s1 := make([]int, 9)
    copy(s1, s[:len(s)-1])
    s = nil
    // []int,0xc00001a140,[0 1 2 3 4 5 6 7 8],len=9,cap=9
    fmt.Printf("%T,%[1]p,%[1]v,len=%d,cap=%d\n", s1, len(s1), cap(s1))
    ```

  * 多切片

    同一底层数组有多个切片引用时，每一个切片的操作都会对其他切片造成影响。

    ```go
    s11 := a[:]
    // []int,0xc00001c120,[0 1 2 3 4],len=5,cap=5
    fmt.Printf("%T,%[1]p,%[1]v,len=%d,cap=%d\n", s11, len(s11), cap(s11))
    s12 := a[1:]
    // []int,0xc00001c128,[1 2 3 4],len=4,cap=4
    fmt.Printf("%T,%[1]p,%[1]v,len=%d,cap=%d\n", s12, len(s12), cap(s12))
    s11[4] = 5
    // []int,0xc00001c120,[0 1 2 3 5],len=5,cap=5
    fmt.Printf("%T,%[1]p,%[1]v,len=%d,cap=%d\n", s11, len(s11), cap(s11))
    // []int,0xc00001c128,[1 2 3 5],len=4,cap=4
    fmt.Printf("%T,%[1]p,%[1]v,len=%d,cap=%d\n", s12, len(s12), cap(s12))
    ```

* 优点

  相比数组，切片可变长，在底层数组长度超过容量时会自动扩容。

* 缺点

  元素类型仍单一，扩容时会分配多出来的内存空间。缩容时若不是通过重新生成底层数组再copy的方式，会浪费内存存空间。
