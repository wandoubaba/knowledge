# 搭建STUN服务

---

> 以centos7为例

* github地址：

```url
https://github.com/jselbie/stunserver
```

* STUNServer官网：

```url
http://www.stunprotocol.org/
```

## 部署过程

```bash
# 安装依赖
yum install -y gcc gcc-c++ make boost-devel openssl-devel
# 下载安装包
cd /usr/local/src
curl -OL http://www.stunprotocol.org/stunserver-1.2.16.tgz
# 解压
tar xvf stunserver-1.2.16.tgz
# 转移目录
mv stunserver /usr/local/
# 安装
cd /usr/local/stunserver
make
# 校验
./stuntestcode
# 启动服务
./stunserver &
```

## 防火墙

```bash
firewall-cmd --permanent --add-port=3478/udp
firewall-cmd --reload
```

## 开机启动stun服务

```bash
vim /etc/rc.local
```

在最后添加一行

```bash
/usr/local/stunserver/stunserver &
```

对rc.local添加执行权限

```bash
chmod +x /etc/rc.d/rc.local
```

## 配置freeswitch

```bash
cd /usr/local/freeswitch/conf
vim vars.xml
```

把stun-set中的服务地址换成刚配置好的stun服务器地址
