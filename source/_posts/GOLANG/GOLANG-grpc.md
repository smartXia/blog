---
title: GOLANG-grpc
categories:
  - GOLANG
date: 2021-11-17 10:28:40
tags:
  - golang
cover: /img/golang.jpeg
---

> grpc 官网：https://grpc.io/docs/languages/go/basics/ 中文翻译版本：http://doc.oschina.net/grpc?t=56831

1、下载protobuf的编译器protoc

地址：

1、`https://github.com/google/protobuf/releases`

```
window：
  下载: protoc-3.3.0-win32.zip
  解压，把bin目录下的protoc.exe复制到GOPATH/bin下，GOPATH/bin加入环境变量。
当然也可放在其他目录，需加入环境变量，能让系统找到protoc.exe

linux：
    下载：protoc-3.3.0-linux-x86_64.zip 或 protoc-3.3.0-linux-x86_32.zip
解压，把bin目录下的protoc复制到GOPATH/bin下，GOPATH/bin加入环境变量。
如果喜欢编译安装的，也可下载源码自行安装，最后将可执行文件加入环境变量。
```

2、获取protobuf的编译器插件`protoc-gen-go`

```
  进入GOPATH目录
  运行
> go get -u github.com/golang/protobuf/protoc-gen-go
  如果成功，会在GOPATH/bin下生成protoc-gen-go.exe文件
```

3、创建一个`test.proto`文件

```
//指定版本
//注意proto3与proto2的写法有些不同
syntax = "proto3";
 
//包名，通过protoc生成时go文件时
package test;
 
//手机类型
//枚举类型第一个字段必须为0
enum PhoneType {
    HOME = 0;
    WORK = 1;
}
 
//手机
message Phone {
    PhoneType type = 1;
    string number = 2;
}
 
//人
message Person {
    //后面的数字表示标识号
    int32 id = 1;
    string name = 2;
    //repeated表示可重复
    //可以有多个手机
    repeated Phone phones = 3;
}
 
//联系簿
message ContactBook {
    repeated Person persons = 1;
}
```

4、运行如下命令

```
> protoc --go_out=. *.proto
会生成一个test.pb.go的文件，具体的文件内容我就不截图了。
```

5、在go语言中使用protobuf

```
package main;
 
import (
    "github.com/golang/protobuf/proto"
    "protobuf/test"
    "io/ioutil"
    "os"
    "fmt"
)
 
func write() {
    p1 := &test.Person{
        Id:   1,
        Name: "小张",
        Phones: []*test.Phone{
            {test.PhoneType_HOME, "111111111"},
            {test.PhoneType_WORK, "222222222"},
        },
    };
    p2 := &test.Person{
        Id:   2,
        Name: "小王",
        Phones: []*test.Phone{
            {test.PhoneType_HOME, "333333333"},
            {test.PhoneType_WORK, "444444444"},
        },
    };
 
    //创建地址簿
    book := &test.ContactBook{};
    book.Persons = append(book.Persons, p1);
    book.Persons = append(book.Persons, p2);
 
    //编码数据
    data, _ := proto.Marshal(book);
    //把数据写入文件
    ioutil.WriteFile("./test.txt", data, os.ModePerm);
}
 
func read() {
    //读取文件数据
    data, _ := ioutil.ReadFile("./test.txt");
    book := &test.ContactBook{};
    //解码数据
    proto.Unmarshal(data, book);
    for _, v := range book.Persons {
        fmt.Println(v.Id, v.Name);
        for _, vv := range v.Phones {
            fmt.Println(vv.Type, vv.Number);
        }
    }
}
 
func main() {
    write();
    read();
}
```

![img](https://upload-images.jianshu.io/upload_images/5367714-2eb580ad8e2e7a93.png)

image.png

```
//go:generate protoc -I ../routeguide --go_out=plugins=grpc:../routeguide ../routeguide/route_guide.proto protoc
```

`-I` 参数：指定import路径，可以指定多个-I参数，编译时按顺序查找，不指定时默认查找当前目录

`--go_out` ：golang编译支持，支持以下参数
plugins=plugin1+plugin2 - 指定插件，目前只支持grpc，即：plugins=grpc
