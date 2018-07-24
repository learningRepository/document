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

//查看版本
$ docker --version 
```


### 常用命令

```shell
//查看镜像
$ docker images

//删除镜像 -f表示强制删除
$ docker rmi -f 镜像id

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

//现在测试下私有仓库
//下载ubuntu镜像
$ docker pull ubuntu 

//打tag 将ubuntu取名为192.168.10.182:5000/ubuntu 不写tag默认为latest
$ docker tag ubuntu 192.168.10.182:5000/ubuntu 

//推送到私有仓库
$ docker push 192.168.10.182:5000/ubuntu 

//查看
//可以直接在宿主机共享目录 查到ubuntu
$ ll /data/docker/registry/v2/repositories/

//使用API查看
$ curl https://192.168.10.182:5000/v2/_catalog 
```
