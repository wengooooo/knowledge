# 打包带Icon应用程序



下载goversioninfo

```go
go get github.com/josephspurrier/goversioninfo/cmd/goversioninfo
```

进入goversioninfo文件夹

```go
cd %GOPATH%/src/vendor/github.com/josephspurrier/goversioninfo/cmd
```

编译goversioninfo.exe出来到gopath的bin文件夹下面，在cmd就可以直接输入goversioninfo.exe使用

```go
go build -o %GOPATH%\bin\goversioninfo.exe main.go
```

复制versioninfo.json文件到项目文件夹与main.go同级,如果是32位就复制32位

```
D:\gopath\pkg\mod\github.com\josephspurrier\goversioninfo@v1.3.0\testdata\example32\versioninfo.json
```

64位

```
D:\gopath\pkg\mod\github.com\josephspurrier\goversioninfo@v1.3.0\testdata\example64\versioninfo.json
```

在main.go文件第一行输入, 一定要第一行输入

```go
//go:generate goversioninfo -icon=icon_YOUR_GO_PROJECT.ico
```

运行之后会生成一个syso文件

```go
go generate
```

开始打包

```
go build main.go
```

如果出来还是没有icon,换个名字打包,可能存在缓存问题

```
go build -o aaa.exe main.go
go build -o bbb.exe main.go
```

https://medium.com/@vijay1.chauhan/create-executable-with-icon-in-golang-6f236995d8f6

https://github.com/josephspurrier/goversioninfo

https://blog.csdn.net/halo_hsuh/article/details/106654340