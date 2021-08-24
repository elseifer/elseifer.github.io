# Getting-started
Golang 入门，以 Mac 环境演示。

## Install

在 [https://golang.org/dl/](https://golang.org/dl/) 找到合适的 go for Mac 安装包并下载，例如 go1.16.6.darwin-amd64.pkg。

双击该 pkg 包将默认将 go 安装在 `/usr/local/go/` 下，并把 `/usr/local/go/bin` 添加在环境变量 PATH 中（见 `/etc/paths.d/go` 文件），安装结束后在终端中键入 `go version` 来验证安装效果<sup>[1]</sup>：
```shell
> go version    
go version go1.16.3 darwin/amd64
```

## 环境变量

在 `.profile` 中设置下 GOPATH 和 GOBIN：
```
export GOPATH=$HOME/go
export GOBIN=$GOPATH/bin
```

目前 Go 已经不需要设置 GOROOT 环境变量<sup>[4]</sup><sup>[5]</sup>，`go` 命令所在目录的父目录默认视为 GOROOT，如 `/usr/local/go` 目录

同时 GOPATH 不能和 GOROOT 相同，否则遇到这样的报错：*$GOPATH must not be set to $GOROOT. For more details see: 'go help gopath'*

可以使用 `go env` 来查看 go 环境变量，例如 `go env GOROOT`

# 课外

## linux环境安装go

go 官网介绍了如何把 go 安装 `/usr/local/go` 目录<sup>[1]</sup>（该目录一般是 root 权限），但有时候需要在其他目录安装，例如 `/home/admin`（或者我们当前用户目录），这里介绍如何安装。

前提；
- 需要 admin 角色登录（以下示例演示在 /home/admin 目录）
- 依据 linux 机器 cpu 架构选择合适的安装包

### 安装

我们把 go 安装在隐藏目录 .go 下：
```
cd /home/admin
mkdir .go
```

在 .go 目录中下载并解压 go 安装包，文件默认解压到 `.go/go` 目录：
```
cd .go
wget https://dl.google.com/go/go1.14.13.linux-amd64.tar.gz
tar -zxvf go1.14.13.linux-amd64.tar.gz 
```

### 环境变量

`/home/admoin` 下创建用户的 go 目录，编辑 `/home/admin/.bash_profile`：

```
export GOPATH=/home/admin/go
export GOBIN=$GOPATH/bin
export PATH=$PATH:/home/admin/.go/go/bin
```

```
source .bash_profile
```

## Mac添加admin目录

1. 获取当期用户

```shell
> whoami
qingqin.cdd
```

2. 把 /homea/dmin 的权限交给当前用户

MacOS 遇到 rootless 问题需要关闭 SIP 保护<sup>[2]</sup>

```shell
cd /home
mkdir admin
sudo chown -R qingqin.cdd:staff admin
```

3. 允许显示 /home/admin

执行第一条命令，注释 `/homeauto_home -nobrowse,hidefromfinder` 这一行，执行第二条命令或者重启 Mac 电脑使修改生效<sup>[3]</sup>。
```shell
sudo vim /etc/auto_master 
sudo automount -vc 
```

## Mac环境变量
全局：
- /etc/paths.d
- /etc/paths
- /etc/profile
- /etc/bashrc

当前用户：
- .profile
- .bashrc
- .bash_profile
- .zshrc

## REF

1.[https://golang.org/doc/tutorial/getting-started](https://golang.org/doc/tutorial/getting-started)

2.[https://apple.stackexchange.com/questions/208478/how-do-i-disable-system-integrity-protection-sip-aka-rootless-on-macos-os-x](https://apple.stackexchange.com/questions/208478/how-do-i-disable-system-integrity-protection-sip-aka-rootless-on-macos-os-x)

3.[https://stackoverflow.com/questions/1362703/how-can-i-use-the-home-directory-on-mac-os-x](https://stackoverflow.com/questions/1362703/how-can-i-use-the-home-directory-on-mac-os-x)

4.[https://dave.cheney.net/2013/06/14/you-dont-need-to-set-goroot-really](https://dave.cheney.net/2013/06/14/you-dont-need-to-set-goroot-really)

5.[https://go-review.googlesource.com/c/go/+/42533/](https://go-review.googlesource.com/c/go/+/42533/)
