# 为ubuntu系统配置静态ip地址

---

> 如果使用非root用户，需要在命令前加sudo

## ubuntu 16.04

```bash
vim /etc/network/interfaces
```

只修改eth0部分（或者是别的网卡名）

```conf
auto eth0
iface eth0 inet static
    address 183.211.245.49
    netmask 255.255.255.128
    broadcast 183.211.245.127
    gateway 183.211.245.1
    dns-nameservers 221.131.143.69
```

如果想用dhcp自动获取ip的话，照下面改

```conf
auto eth0
iface eth0 inet dhcp
```
