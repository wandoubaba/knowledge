# SSH免密码连接服务器

> 前提：服务器已经部署了你的私钥对应的公钥文件
---

## 使用默认密钥文件连接服务器

```bash
ssh username@server
```

例：

```bash
ssh root@192.168.0.8
```

> 使用`~/.ssh/id_rsa`文件以root身份登录192.168.0.8主机

## 使用指定密钥文件连接服务

```bash
ssh -i keyfile username@server
```

例：

```bash
ssh -i ~/.ssh/bob root@192.168.0.8
```

> 使用`~/.ssh/bob`私钥文件以root身份登录192.168.0.8主机

## 指定密钥文件并且服务器ssh端口不是默认的22

```bash
ssh -i ~/.ssh/bob root@192.168.0.8 -p 22222
```

> 服务器把ssh端口设置成22222，需要凭`~/.ssh/bob`文件登录
