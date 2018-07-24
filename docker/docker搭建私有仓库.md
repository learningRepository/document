### 部署私有仓库

镜像在公有仓库dockerhub拉取推送非常慢，而且把项目的镜像放在公网很不安全，在局域网内搭建私有仓库安全快捷。   
docker registry server分为2个版本，第一版是python写成的，第二版是go写的。 本文档基于go版本，需要docker version 1.6以上。

```shell
//运行基于容器的registry.  registry:2镜像不存在时 会去dockerhub上面下载镜像并运行。--restart 设置为always，无论容器的退出代码是什么，Docker都会重启该容器， 设置为on-failure时 只有当容器的退出代码为非0时才会重启，如--restart=on-failure:5表示最多重启5次
$ docker run -d -p 5000:5000 --restart=always --name registry registry:2  

//默认的registry data是存储在容器docker volume中的，如果registry停止volume是会被删除的
//把data共享出来到宿主机目录/data下
$ docker run -d -p 5000:5000 --restart=always --name registry -v /data:/var/lib/registry registry:2  

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

### 客户端使用私有仓库

先安装好Docker 请看上文
```shell
//配置私有仓库地址
$ echo '{ "insecure-registries":["192.168.10.182:5000"] }' > /etc/docker/daemon.json

//重启生效
$ systemctl restart docker

//拉取私有仓库中的ubuntu镜像
$ docker pull 192.168.10.182:5000/ubuntu 

//也可以推送到私有仓库,比如本地有个centos 镜像
$ docker tag centos 192.168.10.182:5000/centos 
$ docker push 192.168.10.182:5000/centos 

//使用API查看
$ curl https://192.168.10.182:5000/v2/_catalog 
```


### 创建证书

如果私有仓库想放到外网访问，这样谁都可以访问，就不安全了，需要登录认证   
running a domain registry using TLS
```shell
//需要安装openssl
$ which openssl

//创建证书
$ openssl genrsa -out server.key 2048
$ openssl req -new -key server.key -out server.csr

You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:guangdong
Locality Name (eg, city) [Default City]:shenzhen
Organization Name (eg, company) [Default Company Ltd]:afmobi
Organizational Unit Name (eg, section) []:soft
Common Name (eg, your name or your server's hostname) []:registrydomain.com
Email Address []:jinliang.yin@afmobigroup.com
Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:

$ openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt

```

然后把证书安装到系统中.
```shell
$ cp server.crt /etc/pki/ca-trust/source/anchors/
$ update-ca-trust enable
$ update-ca-trust extract
```

然后在/etc/hosts中配置域名，重启docker进程
```shell
$ vim /etc/hosts
192.168.10.182 registrydomain.com
$ systemctl restart docker
```

然后使用证书启动容器。
```shell
$ mkdir certs
$ cp server.crt certs/
$ cp server.key certs/
$ docker run -d -p 5000:5000 --name registry --restart=always -v /data:/var/lib/registry -v $PWD/certs:/certs -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/server.crt -e REGISTRY_HTTP_TLS_KEY=/certs/server.key registry:2
```
在其他客户端中也进行安装证书和配置域名并重启docker进程后就可以使用了

用户名密码应用
```shell
//用户名：hello,密码:world 
$ sh -c "docker run --entrypoint htpasswd registry:2 -Bbn hello world > auth/htpasswd"

$ docker run -d -p 5000:5000 --restart=always --name registry   -v `pwd`/auth:/auth   -e "REGISTRY_AUTH=htpasswd"   -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm"   -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd   -v `pwd`/certs:/certs   -v /data:/var/lib/registry   -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/server.crt   -e REGISTRY_HTTP_TLS_KEY=/certs/server.key   registry:2

//登录 输入用户名密码hello:world
$ docker login registrydomain.com:5000

//登出
$ docker logout registrydomain.com:5000 
```

### 客户端用户名密码登录使用私有仓库

```shell
//server.crt 仓库生成的证书
$ cp server.crt /etc/pki/ca-trust/source/anchors/
$ update-ca-trust enable
$ update-ca-trust extract

//然后在/etc/hosts中配置域名，重启docker进程
$ vim /etc/hosts
192.168.10.182 registrydomain.com

$ systemctl restart docker

$ docker login registrydomain.com -u hello -p world
```

### 界面管理私有仓库
```shell
//登录成功后查BASIC_AUTH值         
$ cat /root/.docker/config.json

//启动界面
$ docker run -d -p 8080:8080 --name web --link registry \
       -e REGISTRY_URL=https://192.168.10.182:5000/v2 \
       -e REGISTRY_TRUST_ANY_SSL=true \
       -e REGISTRY_READONLY=false \
       -e REGISTRY_BASIC_AUTH="aGVsbG86d29ybGQ=" \
       -e REGISTRY_NAME=localhost:5000 hyper/docker-registry-web  
```
浏览器访问：http://192.168.10.182:8080/

可以界面上删除镜像，增加delete配置
```shell
//把配置文件拷贝出来
docker cp registry:/etc/docker/registry/config.yml config.yml 

$ vim config.yml
storage:
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
  delete:  
    enabled: true
    .
    .
    .
    
//指定配置文件启动  
$ docker run -d -p 5000:5000 --restart=always --name registry -v `pwd`/config.yml:/etc/docker/registry/config.yml  -v `pwd`/auth:/auth   -e "REGISTRY_AUTH=htpasswd"   -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm"   -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd   -v `pwd`/certs:/certs   -v /data:/var/lib/registry   -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/server.crt   -e REGISTRY_HTTP_TLS_KEY=/certs/server.key   registry:2
```
注意：不建议使用删除，镜像是以层的概念，每个镜像可能依赖相同的层，在这里删除不会物理删除
