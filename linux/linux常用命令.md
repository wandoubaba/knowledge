# linux常用命令和操作技巧

---

## 查看linux发行版本

- centos

```bash
rpm -q centos-release
```

结果大概如下：

centos-release-7-9.2009.1.el7.centos.x86_64

- ubuntu

```bash
lsb_release -a
```

## 查看本机的公网IP

```bash
curl cip.cc
```

## 用curl下载文件（不需要单独安装wget）

```bash
curl http://domain/path -L -O
```

将会在path的最后一段作为文件名来保存文件

```bash
curl http://domain/path -L -o filename
```

将会以filename为文件名保存文件

## centos系统临时切换成英文/中文

```bash
export LANG=en_US.UTF-8
```

```bash
export LANG=zh_CN.UTF-8
```

## ll按时间或大小排序

```bash
# 按时间降序（最新的在最后）
ll -rt
# 按大小降序（最小的在最后）
ll -Sh
```
