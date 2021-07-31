# Code Reading

继续对 [Dlv 设计结构](./Dlv-design.md) 进行探索，这次主要以代码分析为主，delve 代码阅读以 commitId aaed14ff 为止，以 Mac 环境为主要讨论场景。

本文引用的英文说明和 delve 代码较多，如果没有特别注释则默认来自 [https://github.com/go-delve/delve](https://github.com/go-delve/delve) 和 Architecture of Delve slides<sup>1</sup>。

## Target Layer

### pkg/proc/native
>controls target process using OS API calls,supports:
> - Windows
> - Linux
> - MacOS

### pkg/proc/core
>reads linux_amd64 core files



### pkg/proc/gdbserial
>used to connect to
> - debugserver on macOS(default setup on macOS)
> - lldb-server
> - Mozilla RR(a time travel debugger backend, only works on linux/amd64)

**疑问**：dlv attach 是如何选择到合适的 debug server？

对于这个疑问，这里不逐行解释代码，而是在代码中添加注释的方式来浏览下代码。

`pkg/proc/gdbserial/gdbserver.go:551` 行：
```go
// LLDBAttach starts an instance of lldb-server and connects to it, asking
// it to attach to the specified pid.
// Path is path to the target's executable, path only needs to be specified
// for some stubs that do not provide an automated way of determining it
// (for example debugserver).
func LLDBAttach(pid int, path string, debugInfoDirs []string) (*proc.Target, error) {
    //attach 进程不支持 windows 操作系统
	if runtime.GOOS == "windows" {
		return nil, ErrUnsupportedOS
	}
	//如果 go 进程运行在 Rosetta 下则不支持
	if err := macutil.CheckRosetta(); err != nil {
		return nil, err
	}

	var (
		isDebugserver bool
		process       *exec.Cmd
		listener      net.Listener
		port          string
		err           error
	)
	//获取 debugserver 程序的路径
    if debugserverExecutable := getDebugServerAbsolutePath(); debugserverExecutable != "" {
        isDebugserver = true
		//获取本地监听，当address的port为空或者0时，随机选择端口
        listener, err = net.Listen("tcp", "127.0.0.1:0")
        if err != nil {
            return nil, err
        }
		//debugserv 进程参数 -R --attach
		// -R 让 debugserv 进程来主动连接该地址
        args := []string{"-R", fmt.Sprintf("127.0.0.1:%d", listener.Addr().(*net.TCPAddr).Port), "--attach=" + strconv.Itoa(pid)}
        if canUnmaskSignals(debugserverExecutable) {
            args = append(args, "--unmask-signals")
        }
		//先打印进程参数，然后开启 debugserv 进程，这里说明 debugserv 进程的父进程是 dlv attach
		//例如 /Library/Developer/CommandLineTools/Library/PrivateFrameworks/LLDB.framework/Versions/A/Resources/debugserver -R 127.0.0.1:59828 --attach=50879
        process = commandLogger(debugserverExecutable, args...)
    } else {
		//获取不到则使用 lldb-server
        if _, err = exec.LookPath("lldb-server"); err != nil {
            return nil, &ErrBackendUnavailable{}
        }
		//lldb-server 要去监听端口
        port = unusedPort()
		//开启 lldb-server 进程
        process = commandLogger("lldb-server", "gdbserver", "--attach", strconv.Itoa(pid), port)
    }
    process.Stdout = os.Stdout
	process.Stderr = os.Stderr
	process.SysProcAttr = sysProcAttr(false)

	if err = process.Start(); err != nil {
		return nil, err
	}
    
	//dlv 定义的 Process 结构，
	p := newProcess(process.Process)
	p.conn.isDebugserver = isDebugserver

	var tgt *proc.Target
	if listener != nil {
		//dlv attach 本地监听 等待 debugserv 进程来主动连接
		tgt, err = p.Listen(listener, path, pid, debugInfoDirs, proc.StopAttached)
	} else {
		//dlv attach 主动去连接 lldb-server 的端口
		tgt, err = p.Dial(port, path, pid, debugInfoDirs, proc.StopAttached)
	}
	//返回被 debug 的进程
	return tgt, err
}
```

`LLDBAttach` 方法由 `debugger.go` 中调用。如果追溯调用的源头，那就到了 `cmd/dlv/main.go:24` 行，这个调用流程会在[ DLV 启动流程](./Dlv-startup-process.md)中详细介绍。

`service/debugger/debugger.go:144` 行：
```golang
// New creates a new Debugger. ProcessArgs specify the commandline arguments for the
// new process.
func New(config *Config, processArgs []string) (*Debugger, error) {
	logger := logflags.DebuggerLogger()
	d := &Debugger{
		config:      config,
		processArgs: processArgs,
		log:         logger,
	}

	// Create the process by either attaching or launching.
	switch {
	//有传递进程 PID 则进入该分支
	case d.config.AttachPid > 0:
		d.log.Infof("attaching to pid %d", d.config.AttachPid)
		path := ""
		if len(d.processArgs) > 0 {
			path = d.processArgs[0]
		}
		//调用 debugger.Attach
		p, err := d.Attach(d.config.AttachPid, path)
		if err != nil {
			err = go11DecodeErrorCheck(err)
			return nil, attachErrorMessage(d.config.AttachPid, err)
		}
		d.target = p
	//这里不详细展开后面的 case 情况了
```

`service/debugger/debugger.go:342` 行：
```go
// Attach will attach to the process specified by 'pid'.
func (d *Debugger) Attach(pid int, path string) (*proc.Target, error) {
	switch d.config.Backend {
	case "native":
	    //由编译器来决定
		return native.Attach(pid, d.config.DebugInfoDirectories)
	case "lldb":
		//选择 lldb，调用 gdbserver.LLDBAttach 方法
		return betterGdbserialLaunchError(gdbserial.LLDBAttach(pid, path, d.config.DebugInfoDirectories))
	case "default":
		//操作系统为 macos 时，调用 gdbserver.LLDBAttach 方法
		if runtime.GOOS == "darwin" {
			return betterGdbserialLaunchError(gdbserial.LLDBAttach(pid, path, d.config.DebugInfoDirectories))
		}
		//不是 macos，由编译器来决定
		return native.Attach(pid, d.config.DebugInfoDirectories)
	default:
		return nil, fmt.Errorf("unknown backend %q", d.config.Backend)
	}
}
```
`native.Attach(pid, d.config.DebugInfoDirectories)` 在 3 个文件中定义了：
- pkg/proc/native/nonative_darwin.go
- pkg/proc/native/proc_linux.go
- pkg/proc/native/proc_windows.go

对此 Golang 编译器会依据操作系统类型来选择它们中的其中一个进行编译，例如 `nonative_darwin.go` 的头部声明如下内容：
```go
//+build darwin,!macnative

package native
```

如果感兴趣可以阅读 [Build constraints](https://golang.org/cmd/go/#hdr-Build_constraints)


**疑问**：判断选择使用 Mozilla RR 的逻辑在哪里？

## Symbolic Layer

- mostly pkg/proc
- support code in pkg/dwarf and stdlib debug/dwarf



# REF

1.[Architecture of Delve](https://speakerdeck.com/aarzilli/internal-architecture-of-delve)

2.[https://golang.org/cmd/go/](https://github.com/go-delve/delve)
