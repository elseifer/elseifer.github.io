# golang cover
采集go可执行文件的集成覆盖率，基本原理：`go test -c -covermode=count -coverpkg ./...`

## 编译一个测试二进制
完整的编译命令如下：
```sh
go test \
-c -covermode=count -coverpkg=./... \
-ldflags "-B 0x$(shell head -c20 /dev/urandom|od -An -tx1|tr -d ' \n') -X ${PROJECT_NAME}/cmd/demo/main.Version=${MAJOR_VERSION}"  \
-v -o demod \
-count=1 ${PROJECT_NAME}/cmd/demo/main \
```

对于 `ldflags` 参数，`go build` 和 `go test` 支持的方式有些差异的，
这里留意 `-ldflags "-X {PROJECT_NAME}/cmd/demo/main.Version=${MAJOR_VERSION}"` 使用了相对于 `$GOPATH/src` 的路径，
而 `go build -ldflags "-X main.Version=${MAJOR_VERSION}"` 使用了 import path，可以参考 [5]、[6] 文档。

添加 test flag 指明覆盖率报告，启动测试二进制：
```sh
/home/admin/demo/bin/demod  -test.coverprofile cover.out -test.outputdir=/home/admin/demo start -c /home/admin/demo/conf/demo_config.json > out.log 2>&1 &
```

注意为了能够 `go test` 打印 cover.out，不能采用 kill proc 的方式，必须让 demo 进程正常退出，需要实现一个优雅关闭 demo 进程的 api，这一步不做详细介绍。

## html格式覆盖率报告

cover.out 的可视化效果很差

1. 移动 cover.out 到 demo 源代码目录: `mv cover.out $srcPath`
2. 转为 html 格式<sup>[2]</sup>: `go tool cover -html=cover.out -o cover.html` 

同理如果使用 gocover-cobertura 等第三方 xml 格式的覆盖率报告，也是需要源代码目录。
与普通执行 `go test -coverpak ./... -coverprofile cover.out` 得出的 covert.out 不同，这里 covert.out 不是在 demo 源码下生成的，所以 cover.out 的每行是一个go文件相对 `$GOPATH/src` 的路径，需要我们用 sed 修改每行，将路径修改为相对 `$GOPATH/src/${PROJECT_NAME}` 的路径。

# REF
1.[https://www.elastic.co/cn/blog/code-coverage-for-your-golang-system-tests](https://www.elastic.co/cn/blog/code-coverage-for-your-golang-system-tests)

2.[https://go.dev/blog/cover](https://go.dev/blog/cover)

3.[https://stackoverflow.com/questions/31413281/golang-coverprofile-output-format](https://stackoverflow.com/questions/31413281/golang-coverprofile-output-format)

4.[https://pkg.go.dev/cmd/cover](https://pkg.go.dev/cmd/cover)

5.[go test` does not honor -ldflags](https://groups.google.com/g/golang-nuts/c/jxrxhcYywUU)

6.[how-to-set-package-variable-using-ldflags-x-in-golang-build](https://stackoverflow.com/questions/47509272/how-to-set-package-variable-using-ldflags-x-in-golang-build)