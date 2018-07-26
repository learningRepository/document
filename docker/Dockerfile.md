## Dockerfile

Dockerfile 使用基本的基于DSL语法的指令来构建Docker镜像， 之后使用docker build 命令基于dockerfile中的指令构建镜像。
```shell
   ############################################
   # version : nginx-withssl-v1
   # desc : 安装nginx-1.10.2.tar.gz
   ############################################
   # 设置继承镜像
   FROM centos
   
   # 下面是一些创建者的基本信息
   MAINTAINER liang "jinliang.yin@xxxx.com"
  
  # 设置环境变量
  ENV NGINX_VERSION 1.10.2 20180724
  
  RUN yum install epel-release -y && \
      yum install redis -y
  
  EXPOSE 6379
  ENTRYPOINT ["/usr/bin/redis-server"]
```
 
[dockerfile 编写建议规范](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)

以上为一个Dockerfile文件   
第一行为注释    
FROM表示以centos为基础镜像   
MAINTAINER表示作者和联系方式    
ENV表示设置环境变量NGINX_VERSION为1.10.2 20180724,因为dockerfiler的构建是有缓存的，设置时间变量后可以改变时间变量刷新缓存    
RUN表示执行的命令，也可以用数组形式表示如：RUN ["yum", "install", "redis", "-y"]
EXPOSE表示开放6379端口    

```shell
$ cd docker_file_path
$ sudo docker build  -t redis . 
//以上为执行dockerfile的命令

$  docker ps -l //查看端口映射情况
$  docker port cantain_id port_num //same as before
$  docker run -d -P --name cantain_name rep_name/image_name // 将EXPOSE中所有端口绑定到宿主机的49153～65535端口上
```
