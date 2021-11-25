# SSH客户端保持连接的方法

---

在客户端电脑的`~/.ssh/config`文件开头做如下配置：

```conf
Host *
    ServerAliveInterval 30
```

> 表示每隔30秒自动向服务端发送一个no-op包
