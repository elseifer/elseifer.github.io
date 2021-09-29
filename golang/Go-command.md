# GO Command

记录一些常用 go 命令的用法。

## go get


## go install



## go build

go build -gcflags="all=-N -l" -o myApp

### build constraints


## go env


##   goimports 

`go get goimports`
goimports -w cmd/mosn/main/mosn_test.go


## go test

查看 test 支持的全部 flag：
go help testflag


go install ./pkg/...
go test -p 8 -count=1 -failfast -timeout 240s ./pkg/... -coverprofile cover.out


## go tool

go tool cover -html=cover.out -o cover.html

# REF

1.[https://golang.org/cmd/go/](https://golang.org/cmd/go/)

2.[https://go.dev/blog/cover](https://go.dev/blog/cover)

2.[https://dave.cheney.net/2013/10/12/how-to-use-conditional-compilation-with-the-go-build-tool](https://dave.cheney.net/2013/10/12/how-to-use-conditional-compilation-with-the-go-build-tool)

3.[https://dave.cheney.net/2014/09/28/using-build-to-switch-between-debug-and-release](https://dave.cheney.net/2014/09/28/using-build-to-switch-between-debug-and-release)
