+++
title = "Golang WSL2 CentOS8 开发环境配置指南"
date = "2023-01-11T22:32:58+08:00"
author = "theoriz0"
authorTwitter = "" #do not include @
cover = ""
tags = ["开发环境"]
keywords = ["WSL2", "CentOS8", "Go"]
description = "配置 WSL2 下 CentOS8 Go 语言开发环境。解决代理、内存、默认用户等问题。其他 Linux 发行版亦可参考。"
showFullContent = false
readingTime = false
hideComments = false
color = "" #color from the theme settings
+++

## 获取 WSL CentOS8

[mishamosher/CentOS-WSL](https://github.com/mishamosher/CentOS-WSL)

## 安装并启动 CentOS8

解压并运行```CentOS8.exe```。

Windows 命令行下运行：

```bash
## 列出所有版本，确认 CentOS8 已安装成功
wsl --list

## 如果有多个发行版，可以将默认版本设为 CentOS8
wsl -s CentOS8

## 启动。如果设置了默认版本，可以忽略
wsl -d CentOS8
```

## 设置 root 用户密码

```bash
passwd root
```

## 创建普通用户并加入sudo组

```bash
useradd [username]
passwd [username]
sed -i '/^root.*ALL=(ALL).*ALL/a\[username]\tALL=(ALL) \tALL' /etc/sudoers
```

## 替换 yum 源

```bash
mv /etc/yum.repos.d /etc/yum.repos.d.bak # 先备份原有的 Yum 源
mkdir /etc/yum.repos.d
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-vault-8.5.2111.repo
yum clean all && yum makecache
```

## 修改 WSL 默认用户

WSL 下：

```bash
vi /etc/wsl.conf
```

文件内容为：

```text
[user]
default=[username]
```

保存后退出WSL，并在 Windows 下运行：

```bash
wsl --terminate CentOS8
```

重新启动 WSL，可以发现用户已经是所设置用户而不是root。

## 获取 Windows 主机网络地址

WSL下：
```bash
cat /etc/resolv.conf |grep -oP '(?<=nameserver\ ).*'
```

尝试ping该地址。

如果不通，尝试在 Windows powershell 下放开 vEthernet (WSL) 这张网卡的防火墙，执行：

```bash
New-NetFirewallRule -DisplayName "WSL" -Direction Inbound -InterfaceAlias "vEthernet (WSL)" -Action Allow
```

## 配置 ```$HOME/.bashrc```
```bash
# .bashrc

# Source global definitions
if [ -f /etc/bashrc ]; then
        . /etc/bashrc
fi

# User specific environment
if ! [[ "$PATH" =~ "$HOME/.local/bin:$HOME/bin:" ]]
then
    PATH="$HOME/.local/bin:$HOME/bin:$PATH"
fi
export PATH

# Uncomment the following line if you don't like systemctl's auto-paging feature:
# export SYSTEMD_PAGER=

# User specific aliases and functions
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'

# Create workspace folder if not exist
if [ ! -d $HOME/workspace ]; then
    mkdir -p $HOME/workspace
fi

# Basic envs
export LANG="en_US.UTF-8" # 设置系统语言为 en_US.UTF-8，避免终端出现中文乱码
export PS1='[\u@dev \W]$ ' # 默认的 PS1 设置会展示全部的路径，为了防止过长，这里只展示："用户名@dev 最后的目录名"
export WORKSPACE="$HOME/workspace" # 设置工作目录

# Default entry folder
cd $WORKSPACE # 登录系统，默认进入 workspace 目录

# Go envs
export GOVERSION=go1.19.4 # Go 版本设置
export GO_INSTALL_DIR=/usr/local/go # Go 安装目录
export GOROOT=$GO_INSTALL_DIR/$GOVERSION # GOROOT 设置
export GOPATH=$WORKSPACE/golang # GOPATH 设置
export PATH=$GOROOT/bin:$GOPATH/bin:$PATH # 将 Go 语言自带的和通过 go install 安装的二进制文件加入到 PATH 路径中
export GO111MODULE="on" # 开启 Go moudles 特性
export GOPROXY=https://goproxy.cn,direct # 安装 Go 模块时，代理服务器设置
export GOPRIVATE=
export GOSUMDB=off # 关闭校验 Go 依赖包的哈希值

# proxy
alias proxy="source ~/proxy.sh"
proxy set
```

### 配置clash或其他代理软件

详细可参考：[WSL2配置代理](https://www.cnblogs.com/tuilk/p/16287472.html)

开启代理软件的【allow lan/局域网】功能。

WSL 下：

```bash
export ALL_PROXY="http://{ip}:{port}"
## 尝试访问google
curl google.com
```

如果不通，检查 Windows 下 clash 防火墙设置。

如果成功，可以继续配置 ```proxy.sh``` 方便后续使用 ,修改代理端口号为你代理软件的端口号。

创建 ```proxy.sh```：

```bash
vi ~/proxy.sh
```

内容为：

```bash
#!/bin/sh
hostip=$(cat /etc/resolv.conf | grep nameserver | awk '{ print $2 }')
wslip=$(hostname -I | awk '{print $1}')
port=7890

PROXY_HTTP="http://${hostip}:${port}"

set_proxy(){
  export http_proxy="${PROXY_HTTP}"
  export HTTP_PROXY="${PROXY_HTTP}"

  export https_proxy="${PROXY_HTTP}"
  export HTTPS_proxy="${PROXY_HTTP}"

  export ALL_PROXY="${PROXY_SOCKS5}"
  export all_proxy=${PROXY_SOCKS5}

  git config --global http.https://github.com.proxy ${PROXY_HTTP}
  git config --global https.https://github.com.proxy ${PROXY_HTTP}

  echo "Proxy has been opened."
}

unset_proxy(){
  unset http_proxy
  unset HTTP_PROXY
  unset https_proxy
  unset HTTPS_PROXY
  unset ALL_PROXY
  unset all_proxy
  git config --global --unset http.https://github.com.proxy
  git config --global --unset https.https://github.com.proxy

  echo "Proxy has been closed."
}

ip(){
  echo "WSL IP:" ${wslip}
  echo "Host IP:" ${hostip}
}

test_setting(){
  echo "Host IP:" ${hostip}
  echo "WSL IP:" ${wslip}
  echo "Try to connect to Google..."
  resp=$(curl -I -s --connect-timeout 5 -m 5 -w "%{http_code}" -o /dev/null www.google.com)
  if [ ${resp} = 200 ]; then
    echo "Proxy setup succeeded!"
  else
    echo "Proxy setup failed!"
  fi
}

if [ "$1" = "set" ]
then
  set_proxy

elif [ "$1" = "unset" ]
then
  unset_proxy

elif [ "$1" = "test" ]
then
  test_setting

elif [ "$1" = "ip" ]
then
  ip
else
  echo "Unsupported arguments."
fi
```

使```.bashrc```生效

```bash
source ~/.bashrc
```

如果 yum 失败，可尝试关闭代理。

```bash
proxy unset
```

## 安装常用依赖
```bash
sudo yum -y install git make autoconf automake cmake perl-CPAN libcurl-devel libtool gcc gcc-c++ glibc-headers zlib-devel git-lfs telnet lrzsz jq expat-devel openssl-devel openssl
```

## 安装go环境
（如果不需要 go 环境，可删去 .bashrc 的 GO 部分）

确认.bashrc的$GOVERSION，如果安装不同版本，修改之。

按照go官网下载go。[Go官网下载地址](https://go.dev/dl/)

解压:

```bash
rm -rf /usr/local/go && tar -C /usr/local/go/$GOVERSION -xzf go1.19.4.linux-amd64.tar.gz
```

详见 [Go官网安装指南](https://go.dev/doc/install)。

此时 go version应该能输出版本。

### 安装protoc

（最新版本在(github.com)[https://github.com/protocolbuffers/protobuf/releases)，可根据需要自行调整命令)

```bash
cd /tmp/
wget https://github.com/protocolbuffers/protobuf/releases/download/v21.9/protobuf-cpp-3.21.9.tar.gz
tar -xvzf protobuf-cpp-3.21.9.tar.gz
cd protobuf-3.21.9/
./autogen.sh
./configure
make
sudo make install
protoc --version
```

在执行 autogen.sh 脚本时，如果遇到以下错误：
```text
configure.ac:48: installing './install-sh'
configure.ac:109: error: required file  './ltmain.sh'  not found configure.ac:48: installing './missing'
benchmarks/Makefile.am: installing './depcomp'
conformance/Makefile.am:362: warning: shell python --version 2>&1: non-POSIX variable name
conformance/Makefile.am:362: (probably a GNU make extension)
conformance/Makefile.am:372: warning: shell python --version 2>&1: non-POSIX variable name
conformance/Makefile.am:372: (probably a GNU make extension)
parallel-tests: installing './test-driver'
autoreconf: automake failed with exit status: 1
```
可以通过以下命令配置 libtoolize，并再次运行 autogen.sh 来解决：

```bash
libtoolize --automake --copy --debug --force
./autogen.sh
```

## 安装配置 Docker

安装 docker desktop

检查 docker desktop 设置，确保：

- ```General - Use the WSL2 ...``` 选中
- ```Resource - WSL Integration：Enable integration...``` 选中；指定的 WSL Distro（CentOS8）开关打开。

如果报 ```Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock```，可运行以下语句设置权限：

```bash
sudo setfacl --modify user:<user name or ID>:rw /var/run/docker.sock
```

(来源：[StackOverflow](https://stackoverflow.com/questions/51342810/how-to-fix-dial-unix-var-run-docker-sock-connect-permission-denied-when-gro))

如果报```failed to solve with frontend dockerfile.v0: failed to create LLB definition```，需在 docker desktop 中配置:

- docker engine: ```"features": {"buildkit": false },```

并重新用```setfacl```设置权限

（来源：[CSDN](https://blog.csdn.net/qq_41240287/article/details/125236997)）

## 创建默认 K8S 环境

在 Docker Desktop 中设置：

- Kubernetes：勾选 Enable Kubernetes

在 WSL 中检查 K8S 服务情况：

```bash
# 关闭网络代理
proxy unset

kubectl version
kubectl cluster-info
```

从 pod 中访问 WSL 资源时要使用 WSL 的 ip，而不是 localhost。获取方法：
```bash
proxy WSL_ip
## 或者
hostname -I | awk '{print $1}'
```

## 其他

### 如果 WSL 占用内存过大
在 Windows %UserProfile%目录下修改/新建 ```.wslconfig```，内容为：

```text
[wsl2]
memory=4GB
```

注：goland WSL服务端 至少需要3.8GB内存

### 如果域名解析有问题
可以在/etc/resolv.conf添加
```text
# aliyun dns
nameserver 223.5.5.5
nameserver 223.6.6.6
```

### （不建议关闭）关闭自动创建 /etc/resolv.conf
不建议关闭，因为 proxy.sh 脚本通过读取该文件获取 WSL 下 Windows 的 host ip。

新建文件：/etc/wsl.conf，内容如下：
```text
[network]generateResolvConf = false
```

