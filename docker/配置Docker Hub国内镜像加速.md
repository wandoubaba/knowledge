# 配置Docker Hub国内镜像加速

## Ubuntu 16.04+, Debian 8+, CentOS 7

创建或者修改`/etc/docker/daemon.json`文件：

```json
{
    "registry-mirrors": [
        "https://registry.docker-cn.com"
    ]
}
```

> 可以云阿里云镜像加速服务中获取账号专用的阿里云加速链接，添加到"registry-mirrors"的第一行。

重启服务：

```sh
sudo systemctl daemon-reload
sudo systemctl restart docker
```

用命令`docker info`可以查看配置是否生效。
