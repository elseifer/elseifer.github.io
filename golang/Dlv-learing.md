# dlv leanring

dlv 全称 [delve](https://github.com/go-delve/delve) ，它是一款面向 golang 语言的调试工具。
![](https://raw.githubusercontent.com/go-delve/delve/master/assets/delve_horizontal.png)

以下以 Mac 环境为示例。

## 1.Install

1. 下载 dlv 代码并构建<sup>1</sup>
```shell
git clone https://github.com/go-delve/delve
cd delve
go install github.com/go-delve/delve/cmd/dlv
```
或者
```shell
go install github.com/go-delve/delve/cmd/dlv@latest
```

dlv 将安装在 GOBIN 目录，GOBIN 默认为 `$GOPATH/bin`，如 GOPATH 未设置则默认为 `$HOME/go/bin`，关于 [Go 环境变量](./Getting-started#环境变量)。

如果安装报错 `"https://proxy.golang.org/github.com/cosiner/argv/@v/v0.1.0.mod": dial tcp 172.217.160.113:443: i/o timeout` 设置下 GOPROXY 即可：

```shell
go env -w GOPROXY=https://goproxy.cn
```

2. 检测下 dvl 安装

在 GOBIN 目录下执行 dlv：

```shell
> ./dlv version
Delve Debugger
Version: 1.7.0
Build: $Id: e353a65161e6ed74952b96bbb62ebfc56090832b
```

3. 获取 dlv 帮助
```shell
./dlv -h
./dlv --help
```

4. 设置全局 dlv

因为 dlv 不在 `/usr/bin`、`/usr/local/bin` 下，所以每次运行 dlv 均要指定 GOBIN，我们设置全局 dlv 来代替

在 `~/.profile` 中添加一个别名：
```shell
alias dlv='$GOBIN/dlv'
```
再 source 以生效配置
```shell
source ~/.profile 
```

当然，也可以在 `/usr/local/bin` 目录下创建一个链接指向 `$GOBIN/dlv`：
```shell
ln -s $GOBIN/dlv /usr/local/bin/dlv
```

## 2.Usage

上面我们已经体验了 version 命令和 -h 选项，下面详细了解下 dlv 支持的选项和命令。

### Options

下面 Options 是全局级别的，通过 `dlv -h` 获取帮助，同时具体的 dlv command 也会有自己独有的 Options ，通过 `dlv [command] -h` 获取。

```
--accept-multiclient               Allows a headless server to accept multiple client connections.
--allow-non-terminal-interactive   Allows interactive sessions of Delve that don't have a terminal as stdin, stdout and stderr
--api-version int                  Selects API version when headless. New clients should use v2. Can be reset via RPCServer.SetApiVersion. See Documentation/api/json-rpc/README.md. (default 1)
--backend string                   Backend selection (see 'dlv help backend'). (default "default")
--build-flags string               Build flags, to be passed to the compiler. For example: --build-flags="-tags=integration -mod=vendor -cover -v"
--check-go-version                 Checks that the version of Go in use is compatible with Delve. (default true)
--disable-aslr                     Disables address space randomization
--headless                         Run debug server only, in headless mode.
-h, --help                             help for dlv
--init string                      Init file, executed by the terminal client.
-l, --listen string                    Debugging server listen address. (default "127.0.0.1:0")
--log                              Enable debugging server logging.
--log-dest string                  Writes logs to the specified file or file descriptor (see 'dlv help log').
--log-output string                Comma separated list of components that should produce debug output (see 'dlv help log')
--only-same-user                   Only connections from the same user that started this instance of Delve are allowed to connect. (default true)
-r, --redirect stringArray             Specifies redirect rules for target process (see 'dlv help redirect')
--wd string                        Working directory for running the program.
```

### Command

`dlv attach` - Attach to running process and begin debugging.

`dlv connect` - Connect to a headless debug server.

`dlv core` - Examine a core dump.

`dlv dap` - [EXPERIMENTAL] Starts a headless TCP server communicating via Debug Adaptor Protocol (DAP).

`dlv debug` - Compile and begin debugging main package in current directory, or the package specified.

`dlv exec` - Execute a precompiled binary, and begin a debug session.

`dlv replay` - Replays a rr trace.

`dlv run` - Deprecated command. Use 'debug' instead.

`dlv test` - Compile test binary and begin debugging program.

`dlv trace` - Compile and begin tracing program.

`dlv version` - Prints version.

`dlv log` - Help about logging flags

`dlv backend` - Help about the --backend flag

## 3.实操attach

attach 默认暂停程序的执行，可设置 `--continue` 选项让 attach 不阻塞程序执行。
> --continue   Continue the debugged process on start.

开启一个不暂停目标程序、无交互界面模式的 debug 会话，会话端口为 8181（未指定 `--listen` 时端口由系统分配）。
```shell
dlv attach 16591 --continue --headless --accept-multiclient --listen=:8181
```

**注意**：指定 `--continue` 时必须搭配 `--headless`

连接上已开启的 8181 会话。
```shell
dlv connect 127.0.0.1:8181
```

**注意**：在 connect 上后目标程序即可被暂停

### attach正在运行的程序

编写一个 demo 程序
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	i:=0
	for {
		i++
		time.Sleep(time.Duration(1)*time.Second)
		fmt.Println(i)
		if i > 1000 {
			break;
		}
	}
}
```

编译 demo.go 为可执行程序 demo.exe，这里的 .exe 并无实际意思
```shell
go build -v -o demo.exe demo.go
```

运行 demo.exe 并查看进程 PID，例如 16250
```shell
./demo.exe
```

使用 ps 查看进程 PID，第二列为 PID
```
ps -ef|grep -v grep |grep demo.exe
```

或者使用 gops 查看进程 PID<sup>4</sup>，第一列为 PID
```
gops |grep demo.exe
```

以非 headless 模式 attach 到 16250 进程上，并出现 dlv 用户交互界面：
```shell
>dlv attach 16250
Type 'help' for list of commands.
(dlv)
```

### 修改程序运行状态
在 dlv 交互界面可以以命令的方式调试程序：

1.新增一个断点  
```
break demo.go:14
```

2.查看全部断点  
```
bp
```
可以看到存在额外的断点，因为 dlv 默认对 `/usr/local/go/src/runtime/panic.go` 施加了断点。

3.继续程序直到下一个断点  
```
continueq
```

4.修改变量的值  
```
set i=0
```

5.打印变量的值  
```
print i
```

6.清除全部断点  
```
clearall
```
但它不会清除 dlv 默认施加的 panic 断点。

## 4.More

### Goland集成attach
对于 remote attatch，我们可以在 goland 中集成 gops <sup>3</sup> 工具。
gops 由 google 官方提供，用于查看和诊断正在运行的 go 程序进程<sup>4</sup>。

>gops is a command to list and diagnose Go processes currently running on your system.

首先安装 gops：
```shell
go get -t github.com/google/gops/
```

之后重启 goland，工具栏出现 **Run | Attach to Process**(`⌥⇧F5`)：
![](./images/goland-gops.jpg)

运行 `./demo.exe`，尝试下 `⌥⇧F5`：
![](./images/gops-attach.jpg)

# REF

1.[Delve Installation](https://github.com/go-delve/delve/tree/master/Documentation/installation)

2.[Using Dlv](https://github.com/go-delve/delve/blob/master/Documentation/usage/dlv.md)

3.[Attach to running Go processes with the debugger](https://www.jetbrains.com/help/go/attach-to-running-go-processes-with-debugger.html)

4.[google gops](https://github.com/google/gops/)