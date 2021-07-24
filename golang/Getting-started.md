# Getting-started
Mac 环境的 Goland 的入门

## Install

下载 Mac 下的 go 安装包，例如 go1.16.6.darwin-amd64.pkg，双击该 pkg 包将默认安装 go 在 `/usr/local/go/` 下，同时在环境变量 PATH 中添加 `/usr/local/go/bin`（详见 `/etc/paths.d/go` 文件），安装结束在终端中键入 `go version` 查看<sup>1</sup>：
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

目前，高版本 Go 已经不需要设置 GOROOT 环境变量<sup>[4]</sup><sup>5</sup>。

# 课外

## 添加admin目录

1. 获取当期用户

```shell
> whoami
qingqin.cdd
```

2. 把 /homea/dmin 的权限交给当前用户

```shell
cd /home
mkdir admin
sudo chown -R qingqin.cdd:staff admin
```

3. 允许显示 /home/admin

```shell
sudo vim /etc/auto_master 
注释 `/homeauto_home -nobrowse,hidefromfinder` 这一行

sudo automount -vc 
执行上面命令或者重启 Mac 电脑使修改生效
```

## Mac环境变量

/etc/paths.d
/etc/paths
/etc/profile
/etc/bashrc

.profile
.bashrc
.bash_profile


## REF

1.[https://golang.org/doc/tutorial/getting-started](https://golang.org/doc/tutorial/getting-started)

2.[https://apple.stackexchange.com/questions/208478/how-do-i-disable-system-integrity-protection-sip-aka-rootless-on-macos-os-x](https://apple.stackexchange.com/questions/208478/how-do-i-disable-system-integrity-protection-sip-aka-rootless-on-macos-os-x)

3.[https://stackoverflow.com/questions/1362703/how-can-i-use-the-home-directory-on-mac-os-x](https://stackoverflow.com/questions/1362703/how-can-i-use-the-home-directory-on-mac-os-x)

4.[https://dave.cheney.net/2013/06/14/you-dont-need-to-set-goroot-really](https://go-review.googlesource.com/c/go/+/42533/)

5.[https://go-review.googlesource.com/c/go/+/42533/](https://go-review.googlesource.com/c/go/+/42533/)
