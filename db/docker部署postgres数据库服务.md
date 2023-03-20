# docker部署postgres数据库服务

> 本文同时实现按中文拼音排序（将默认语言环境设置为zh_CN.utf8）

基于docker和docker-compose，操作前要先安装这两个服务和工具。

## 创建存储目录

比如创建`/data/postgres/`目录。

```sh
mkdir /data/postgres -p
```

## 创建Dockerfile

> postgres官方的镜像中不包含zh_CN.utf8环境，因此不支持数据按中文拼音排序，所以需要通过Dockerfile生成自定义镜像。**支持中文排序这一特性会损失pg查询性能**。

在`/data/postgres`目录中创建`Dockerfile`文件：

```sh
touch Dockerfile
vim Dockerfile
```

文件内容如下（注意版本要“与时俱进”和“按需选择”）：

```Dockerfile
FROM postgres:14
RUN localdef -i zh_CN -c -f UTF-8 -A /usr/share/locale/locale.alias zh_CN.UTF-8
ENV LANG zh_CN.utf8
```

使用`docker build`创建自定义镜像（修改自己的前缀）：

```sh
docker build -t wkzz/postgres .
```

接下来使用`docker images`命令查看本地镜像：

```sh
docker images
```

应该可以看到`postgres`和`wkzz/postgres`镜像了。

## 创建编排文件

创建文件`/data/pgspostgresql/docker-compose.yml`，内容如下（注意自己改管理员账号密码）：

> `image`一行要用自己编译的镜像名。

```yml
version: "3.1"
services:
  db:
    image: wkzz/postgres
    restart: always
    environment:
      POSTGRES_PASSWORD: wkzz051223
      PGDATA: /var/lib/postgresql/data/pgdata
    ports:
      - 5432:5432
    volumes:
      - ./data:/var/lib/postgresql/data
```

## 启动容器

在`/data/pgsql`目录下执行下面的命令：

```sh
docker-compose up -d
```

## 查看当前postgres版本

先用`docker ps`命令找到postgres容器实例，假设实例id是`52a63c60bb59`：

```sh
docker exec -it 52a63c60bb59 /bin/bash
# 进入容器后执行下面命令
psql --version
# psql (PostgreSQL) 14.1 (Debian 14.1-1.pgdg110+1)
```
