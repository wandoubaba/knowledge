# docker操作

---

## docker容器设置开机自启动

> 前提是docker服务先被设置成开机自启

- 新建容器时配置自启参数

```bash
docker run --restart=always <容器id> 或 <容器名称>
```

- 已经存在的容器配置自启

```bash
docker update --restart=always <容器id> 或 <容器名称>
```

- 取消容器自启

```bash
docker update --restart=no <容器id> 或 <容器名称>
```

- 批量设置所有容器自启

```bash
docker update --restart=always $(docker ps -aq)
```
