# 在ubuntu系统里把网卡名称改为eth0

---

## 在ubuntu16.04系统

> 非root用户需要在命令前加sudo

### 1. 修改grub文件

```bash
vim /etc/default/grub
```

原文

```conf
GRUB_CMDLINE_LINUX=""
```

改为

```conf
GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0"
```

保存退出

### 2. grub生效

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

### 3. 改网络配置

> 适用于ubuntu16.04，其他版本不一定有效

```bash
vim /etc/network/interfaces
```

原文（类似）

```txt
auto ens160

iface ens160 inet static
```

改为

```txt
auto eth0

iface eth0 inet static
```

### 4. 网卡开机自启（大部分情况可省略）

```bash
systemctl enable networking.service
```

### 5. 重启

```bash
reboot
```

或

```bash
init 6
```

### 6. 查看网络信息

```bash
ip addr
```

---

## 在centos7系统

### 1. 先确定当前网卡名

```bash
ip addr
```

### 2. 改网卡配置文件

> 原网卡名称有可能不一样

```bash
vim /etc/sysconfig/network-scripts/ifcfg-ens192
```

把`DEVICE=ens192`改为`DEVICE=eth0`

把`NAME=ens192`改为`NAME=eth0`

保存退出

再把网卡配置文件的文件名改了

```bash
cd /etc/sysconfig/network-scripts
mv ifcfg-ens192 ifcfg-eth0
```

### 3. 修改grub

```bash
vim /etc/default/grub
```

往`GRUB_COMLINE_LINUX`里面加上`net.ifnames=0 biosdevname=0`

保存退出

执行命令

```bash
grub2-mkconfig -o /boot/grub2/grub.cfg
```

### 4. 重启系统并校验效果

```bash
init 6
```

```bash
ip a
```
