[TOC]

# 写在前面

王者巅峰赛已经1500分，索然无味。写一篇杂谈消遣下。背景是近期在生产上总是遇到合作机构传过来的对账文件不全的问题，每当有告警就要找合作机构重新推送，然而合作机构重新推送也真的就是重新推送，当文件大的时候，耗费时间较多。因此我就思考，文件拷贝在GO中是怎么实现，是否可以做到断点续传。

# 一、Read、Write方法

```go
func (f *File) Read(b []byte) (n int, err error)
func (f *File) Write(b []byte) (n int, err error)
```

# 二、ReadFile、WriteFile方法



# 三、Copy方法



# 四、关于断点续传的思考

