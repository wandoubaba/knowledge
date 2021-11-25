# ubuntu系统配置NFS服务端及客户端

## 配置nfs服务端与客户端

> nfs的作用是实现各主机间的共享存储空间，`切记不要在公网环境中使用nfs！！！`

### 1. NFS服务端安装与配置

- 安装nfs服务，需要先安装：

```shell
    sudo apt install nfs-kernel-server nfs-common
```

- 配置nfs共享目录：

```bash
    vim /etc/exports
```

- 添加如下两行：

```ini
/storage        192.168.1.0/24(rw,sync,no_subtree_check,no_root_squash,fsid=0)
/storage        127.0.0.1/24(rw,sync,no_subtree_check,no_root_squash,fsid=0)
```

> 其中`/storage`为需要共享的路径，`Tab`键后的ip地址是允许访问的网段（根据实际情况修改），括号内则是一些权限（不建议改动）。

- 重启nfs服务并刷新共享配置：

```bash
    /etc/init.d/nfs-kernel-server restart
    exportfs -rv
```

### 2. NFS服务端的端口处理

> NFS服务是基于RPC协议的服务，通过`rpcinfo -p`命令会发现除了`111`和`2049`端口外，其`mountd`和`nlockmgr`以及`status`对应的端口是随机分配的，这样几乎无法搞防火墙和端口映射了，为了不让服务器裸奔，需要对nfs-kernel-server主机做一些配置以固定其端口。

- 修改`/etc/default/nfs-common`文件，配置`status`服务端口为`40001`：

```ini
    STATDOPTS="--port 40001"
```

- 修改`/etc/default/nfs-kernel-server`文件，配置`mountd`服务端口为`40002`：

```ini
    RPCMOUNTDOPTS="--manage-gids -p 40002"
```

- 创建`/etc/modprobe.d/options.conf`文件，添加如下内容，配置`nlockmgr`端口为`40003`：

```ini
    options lockd nlm_udpport=40003 nlm_tcpport=40003
```

- 在`/etc/modules`文件中添加`lockd`：

```ini
# /etc/modules: kernel modules to load at boot time.
#
# This file contains the names of kernel modules that should be loaded
# at boot time, one per line. Lines beginning with "#" are ignored.
lockd
```

- 重启服务器（`reboot`），然后再运行`rpcinfo -p localhost`，就会看到，各种端口都已经被固定了。

### 3. NFS客户端安装与配置

- 安装nfs客户端（以ubuntu为例）：

```shell
    sudo apt install nfs-common -y
```

- 确保客户端与服务端的网络连接正常，在客户端主机上执行以下操作：

```shell
    showmount -e 192.168.1.68
```

> 正常情况下会命令执行后会返回服务器开放出来的共享目录路径，包含类似`/storage 192.168.1.0/24`的信息。

- 打开`/etc/fstab`文件：

```shell
    vim /etc/fstab
```

- 在文件末尾所加行（`tab`键间隔）后保存并退出编辑器：

```ini
192.168.1.68:/storage   /storage        nfs     defaults        0       0
```

> 其中第1段IP为本机IP地址，第2段为要挂载到`/storage`目录，后面几段原样照抄。
> 如果在nfs服务端本机内配置，可将ip地址改为127.0.0.1。

- 创建挂载点目录：

```shell
    mkdir /storage
```

> 完成以上操作后，就会在系统启动时自动将服务器的/storage目录挂载到本机的/storage目录。

- 以下命令可实现一次性的手动挂载（如果未在fstab中配置的话，重启服务器将失效）：

```shell
    sudo mount -t nfs -o nolock 192.168.1.68:/storage /storage
```
