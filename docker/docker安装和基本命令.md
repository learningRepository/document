### docker安装

此文档是基于Centos7.2操作。用户为root。
系统内核需要3.10或更高版本
```shell
//查看内核
$ uname -a  

//安装docker,此脚本会检查内核并安装
$ curl -fsSL https://get.docker.com/ | sh 

//启动服务
$ systemctl start docker 

$ docker --help

//查看版本
$ docker --version 

//查看系统信息
$ docker info
```


### 常用命令

```shell
//查找镜像
$ docker search nginx

//下载ubuntu镜像
$ docker pull ubuntu 

//推送到仓库
$ docker push beyondyinjl/ubuntu

//创建容器,没有启动
$ docker create  --name my nginx:latest   

//运行容器
$ docker run -d -p 80:80 --name webserver nginx

//进入容器 开启一个交互模式的终端
$ docker exec -it webserver /bin/bash

//在容器内执行后台任务，创建一个文件
$ sudo docker exec -d centos_d_container touch /etc/new_config_file 

//交互模式启动容器
$ docker run -it beyondyinjl/centos /bin/bash

//查看镜像
$ docker images

//删除镜像 -f表示强制删除
$ docker rmi -f 镜像id

//创建一个后台执行的容器, -d表示后台执行
$ docker run --name centos_d_container -d centos /bin/sh -c "while true; do echo hello world; sleep 1; done"

//获取容器日志 与tail -f 类似
$ docker logs centos_d_container -f

//来查看当前系统中正在运行的容器列表
$ docker ps    

//命令会列出所有的容器
$ docker ps -a  

//停止容器 start restart
$ docker stop registry (容器名或容器id)

//删除容器
$ docker rm registry (容器名或容器id)

//删除全部容器
$ docker rm `docker ps -a -q`

//查看容器的端口映射情况
$ docker port 容器名 

//获取容器/镜像的元数据
$ docker inspect 容器或镜像

//查看容器的进程信息
$ docker top 容器

```
