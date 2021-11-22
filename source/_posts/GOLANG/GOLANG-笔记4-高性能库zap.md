---
layout: golang-笔记4-高性能库zap
title: 高性能库zap gox 
date: 2021-04-29 17:31:41
tags: 
  - golang
  - zap
  - gox
  - module
cover: /img/golang.jpeg
categories:
  - GOLANG
---

## zap
高性能日志库分析
#### 参数配置

##GOX
交叉编译工具可以编译各种的环境
go get github.com/mitchellh/gox
gox -build-toolchain

直接运行gox。程序会一口气生成17个文件
横跨windows,linux,mac,freebsd,netbsd五大操作系统
#####固定平台
gox -osarch "windows/amd64 linux/amd64" 或
        gox -os "windows linux" -arch amd64

#### go mod init 命令
go.sum是一个模块版本内容的校验值，用来验证当前缓存的模块。go.sum包含了直接依赖和间接依赖的包的信息，比go.mod要多一些。
#### 查看依赖包
go list -m all
#### 模块配置文本格式化
go mod edit -fmt
#### Windows 下开启 GO111MODULE 的命令为：
set GO111MODULE=on 或者 set GO111MODULE=auto
#### 代理
GOPROXY=https://goproxy.cn,direct
#### 以索引整个 GOPATH
.Preferences -> Go -> GOPATH，勾选上 Index entire GOPATH
<hr>

#### 基础命令关于module的
go mod download
go mod download -json 参数会以JSON的格式打印下载的模块对象<br />
go mod tidy   
go mod tidy -v  可以将执行的信息
可以使用go mod tidy命令来清除它<br />
go mod vendor
go mod vendor -v会将添加到vendor中的模块打印到标准输出。
go mod graph<br />
打印模块依赖图
