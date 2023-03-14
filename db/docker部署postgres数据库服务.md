# docker部署postgres数据库服务

基于docker和docker-compose，操作前要先安装这两个服务和工具。

## 创建存储目录

比如创建`/data/postgres/`目录。

```sh
mkdir /data/postgres -p
```

## 创建编排文件

创建文件`/data/pgspostgresql/docker-compose.yml`，内容如下（注意自己改管理员账号密码）：

```yml
version: "3.1"
services:
  db:
    image: postgres
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
