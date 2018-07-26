## Docker Compose

   Docker Compose 定义一组要启动的容器，以及容器运行是的属性，这组容器叫做服务。 Docker Compose可以用一个YAML文件定义并启动服务。
Docker Compose File有两个版本：version 1现在已不支持,建议使用version 2。 

安装docker-compose
```shell
$ curl -L https://github.com/docker/compose/releases/download/1.11.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
$ chmod +x /usr/local/bin/docker-compose
$ docker-compose --version

//列出容器
$ docker-compose ps 

//version 1现在已不支持,建议使用version 2
//下面使用docker-compose 以服务方式 启动私有仓库和界面管理
$ vim docker-compose.yml
version: '2'
services:
  registry:
    restart: always
    image: registry:2
    ports:
      - 5000:5000
    environment:
      REGISTRY_HTTP_TLS_CERTIFICATE: /certs/server.crt
      REGISTRY_HTTP_TLS_KEY: /certs/server.key
      REGISTRY_AUTH: htpasswd
      REGISTRY_AUTH_HTPASSWD_PATH: /auth/htpasswd
      REGISTRY_AUTH_HTPASSWD_REALM: Registry Realm
    volumes:
      - ./config.yml:/etc/docker/registry/config.yml
      - ./auth:/auth
      - ./certs:/certs
      - /data:/var/lib/registry
    networks:
      - registry-net
  web:
    restart: always 
    image: hyper/docker-registry-web
    ports:
      - 8080:8080
    environment:
      REGISTRY_URL: https://registry:5000/v2
      REGISTRY_TRUST_ANY_SSL: "true"
      REGISTRY_READONLY: "false"
      REGISTRY_BASIC_AUTH: aGVsbG86d29ybGQ=
      REGISTRY_NAME: registry:5000
    networks:
      - registry-net
networks:
  registry-net:


//后台启动
$ docker-compose up -d 
```
