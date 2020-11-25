---
title: Docker-info
date: 2020-08-20 17:18:43
tags: docekr images
cover: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1597926782647&di=df9480fa85e9c5331f808d7254b352ac&imgtype=0&src=http%3A%2F%2Fpic1.zhimg.com%2Fv2-db080a21341c9aa0a7d508e39353531f_1200x500.jpg
categories:
  - DOCKER
---

#### 注册登录

https://hub.docker.com/

#### Docker快捷键

> 常用： start restart stop  images ps-a

##### 带有参数的使用

docker ps -a ：查看最近使用的容器id

docker rm 容器id:删除某个容器

docker images

docker rmi 镜像id:删除某个镜像

docker run :

docker run -d -p 9200:9200 -p 5601:5601 nshou/elasticsearch-kibana

> -d 后台运行，-p 内部端口/宿主机端口 容器id

docker exec -it /bash :进入容器

docker login -u xx -p xxx：登录

#### 配置加速源

1.阿里云：百度如何通过阿里云加速docker拉取和推送速度

2.DaoCloud ：大公司，国内的。网站：https://www.daocloud.io/mirror

> 加速url

```
Linux:curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io
```

原理：编辑   /etc/docker/daemon.json  这个文件夹

  ```
{"registry-mirrors": ["http://f1361db2.m.daocloud.io","https://vbw6t0eb.mirror.aliyuncs.com"]}
  ```

#### 查看当前docker配置文件



docker info :可以查看是否配置成功加速 版本信息等各种信息



#### 打包镜像源和推送到docker.io

1.docker pull xxx镜像:tag

2.docker images 查看镜像

3.docker run -d -p 8080:8081 xxx镜像:version

4.docker ps -a 查看是不是启动了，然后stop start restart 找找感觉

5.docker exec -it 镜像id bash :进入镜像进行修改：拉代码，查bug ,增加mysql实例等

6.docker commit -m "php71-daemon:xhprof-graphviz" -a "some" f69187b4375e "18260356308/php71-daemon:xhprof"

​	docker commit -m "提交log" -a "作者"  容器id "docker账户名/自定义镜像名：tag" 就会制作成一个新的image了

7.执行docker push xxx镜像的id：

> tips:
>
> 前提是得登录，还有  注意一个问题,给自己镜像命名的时候格式应该是: docker注册用户名/镜像名,比如我的docker用户名为 test123,那么我的镜像tag就为 test123/whalesay,不然是push不上去的





