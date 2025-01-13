## 1 := 与 =
:= 与下面的语句等价
```
var [变量] [类型]
[变量] = [值]
```

## 2 go语言编译
先配置gopath环境变量，在项目根目录进行初始化
```
go mod init [模块名]
```
随后
```
go run
或者
go build -o [文件名].exe
```

## 3 filepath.walk
walk func的标准定义：
```go
walkFn := func(path string, info os.FileInfo, err error) error
```