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

## 部署过程（推荐Docker）

> 确保已有docke环境

```bash
# 下载安装包
wget http://www.stunprotocol.org/stunserver-1.2.16.tgz
# 解压
tar xvf stunserver-1.2.16.tgz
# 转移目录
mv stunserver /usr/local/
cd /usr/local/stunserver
docker image build -t=stun-server-image .
docker run -d -p 3478:3478/tcp -p 3478:3478/udp --name=stun-server stun-server-image
```

## 部署过程（编译安装）

* 如果是centos系统：

```bash
# 安装依赖
yum install -y gcc gcc-c++ make boost-devel openssl-devel
cd /usr/local/src
```

* 如果是debian系统：

```bash
apt install -y libboost1.74-all-dev
cd /usr/src
```

```bash
# 下载安装包
wget http://www.stunprotocol.org/stunserver-1.2.16.tgz
# 解压
tar xvf stunserver-1.2.16.tgz
# 转移目录
mv stunserver /usr/local/
cd /usr/local/stunserver
# 安装
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
