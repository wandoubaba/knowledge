# docker部署minio服务

## 单节点-单存储

> 基于linux系统

### 拉取image

```shell
# 推荐quay.io
docker pull quay.io/minio/minio
# 备用dockerhub
docker pull bitnami/minio
```

### 创建环境变量文件

```shell
touch /etc/default/minio
vim /etc/default/minio
```

在环境变量文件中编辑如下内容

```conf
# MINIO_ROOT_USER and MINIO_ROOT_PASSWORD sets the root account for the MinIO server.
# This user has unrestricted permissions to perform S3 and administrative API operations on any resource in the deployment.
# Omit to use the default values 'minioadmin:minioadmin'.
# MinIO recommends setting non-default values as a best practice, regardless of environment

MINIO_ROOT_USER=myminioadmin
MINIO_ROOT_PASSWORD=minio-secret-key-change-me

# MINIO_VOLUMES sets the storage volume or path to use for the MinIO server.

MINIO_VOLUMES="/mnt/data"

# MINIO_SERVER_URL sets the hostname of the local machine for use with the MinIO Server
# MinIO assumes your network control plane can correctly resolve this hostname to the local machine

# Uncomment the following line and replace the value with the correct hostname for the local machine.

#MINIO_SERVER_URL="http://minio.example.net"
```

### 启动container

> 下面的命令不能直接复制

```shell
# 下面命令中的PATH要换成本地路径，如/data/minio
docker run -dt                                  \
  -p 9000:9000 -p 9090:9090                     \
  -v PATH:/mnt/data                             \
  -v /etc/default/minio:/etc/config.env         \
  -e "MINIO_CONFIG_ENV_FILE=/etc/config.env"    \
  --name "minio_local"                          \
  quay.io/minio/minio server                    \
  --console-address ":9090"
```

### 查看容器状态

```shell
docker logs minio
```

正常的话应该能看到类似如下信息

```console
Status:         1 Online, 0 Offline.
API: http://10.0.2.100:9000  http://127.0.0.1:9000
RootUser: myminioadmin
RootPass: minio-secret-key-change-me
Console: http://10.0.2.100:9090 http://127.0.0.1:9090
RootUser: myminioadmin
RootPass: minio-secret-key-change-me

Command-line: https://min.io/docs/minio/linux/reference/minio-mc.html
   $ mc alias set myminio http://10.0.2.100:9000 myminioadmin minio-secret-key-change-me

Documentation: https://min.io/docs/minio/container/index.html
```

### 通过web访问minio面板

容器启动成功后，minio服务的web面板在本机可以通过`http://localhost:9090`访问，如果端口开放正确，通过`http://IP:9090`可以打开minio的web面板，如果有前置nginx的话，只要做指向http://IP:9090的反向代理就可以了。

web面板的账号密码就是在`/etc/default/minio`这个环境变量文件中配置的`MINIO_ROOT_USER`和`MINIO_ROOT_PASSWORD`。
