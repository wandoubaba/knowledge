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
yum install gcc g++ make
yum install boost-devel
yum install openssl-devel
# 下载安装包
cd /usr/local/src
curl -OL http://www.stunprotocol.org/stunserver-1.2.16.tgz
# 解压
tar xvf stunserver-1.2.16.tgz
# 安装
cd stunserver
make
# 校验
./stuntestcode
# 运行
./stunserver &
```

## 防火墙

```bash
firewall-cmd --permanent --add-port="3478/udp"
```

## 配置freeswitch

```bash
cd /usr/local/freeswitch/conf
vim vars.xml
```

把stun-set中的服务地址换成刚配置好的stun服务器地址
