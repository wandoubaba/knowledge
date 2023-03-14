# docker部署minio服务

基于docker和docker-compose，操作前要先安装这两个服务和工具。

## 创建存储目录

比如创建`/data/minio/`目录，准备用其中的`/data/minio/data`保存所有存储桶（bucket）和其中的对象，用其中的`/data/minio/config`映射服务配置。

```sh
mkdir /data/minio -p
```

## 创建编排文件

创建文件`/data/minio/docker-compose.yml`，内容如下（注意自己改管理员账号密码）：

```yml
version: "3.1"
services:
  minio:
    image: quay.io/minio/minio:latest
    hostname: "minio"
    restart: always
    environment:
      MINIO_ROOT_USER: wkzz
      MINIO_ROOT_PASSWORD: wkzz051223
      MINIO_VOLUMES: /mnt/data
    ports:
      - 9090:9090
      - 9000:9000
    volumes:
      - ./data:/mnt/data
      - ./config:/root/.minio/
    command: server --console-address ':9090'
```

## 启动容器

在`/data/minio`目录下执行下面的命令：

```sh
docker-compose up -d
```

## 验证安装

服务器防火墙要放行`9090`端口，或者在nginx上做好对应的代理，这里以开端口为例，如果服务器的IP是192.168.0.8，那么在web浏览器上打开`http://192.168.0.8:9090`，应该可以看到登录页，填入在config.env中设置的MINIO_ROOT_USER和MINIO_ROOT_PASSWORD，应该可以登录minio的web控制台。
