# centos7防火墙操作

---

centos7默认使用firewall作为防火墙（iptables仍然有效）。

## 查看防火墙服务状态

```bash
firewall-cmd --state
```

## 启动、关闭、状态

```bash
# 查看状态
systemctl status firewalld
# 启动
systemctl start firewalld
# 停止
systemctl stop firewalld
# 重启
systemctl restart firewalld
# 禁卡开机启动
systemctl disable firewalld
# 开机启动
systemctl enable firewalld
```

## 端口操作

```bash
# 查看已经打开的端口
firewall-cmd --list-ports
# 查看当前所有配置（包括端口、服务等所有）
firewall-cmd --list-all
# 查看firewall所有可识别的服务
firewall-cmd --get-service
# 查看端口是否开通（如果开的是服务，直接查端口有可能会是no）
firewall-cmd --query-port=22/tcp
# 查看服务是否开通
firewall-cmd --query-service=ssh
# 开放某个端口(permanent是永久生效)
firewall-cmd --permanent --add-port=80/tcp
# 开放某段端口范围
firewall-cmd --permanent --add-port=5000-6000/tcp
# 关闭某个端口
firewall-cmd --permanent --remove-port=80/tcp
# 开放某个服务(的默认端口)
firewall-cmd --permanent --add-service=http
# 关闭某个服务(的默认端口)
firewall-cmd --permanent --remove-service=http
# 对指定IP开放（相当于白名单）
firewall-cmd --permanent --add-source=192.168.1.100
# 对指定IP段开放（相当于白名单）
firewall-cmd --permanent --add-source=192.168.1.0/24
# 移除某IP
firewall-cmd --permanent --remove-source=192.168.1.100
```

## 复杂操作

```bash
# 允许指定IP访问本机指定端口
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.100" port protocol="tcp" port="80" accept'
# 允许指定IP段访问本机端口范围
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" port protocol="tcp" port="8080-8090" accept'
# 禁止指定IP访问本机指定端口
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.100" port protocol="tcp" port="80" reject'
# 屏蔽某个IP的所有请求
firewall-cmd --permanent --add-rich-rule='rule family=ipv4 source address=103.145.13.92 reject'
```

## 配置生效

任何操作执行完成后，都需要重新装载或重启服务以使其生效：

```bash
firewall-cmd --reload
# 或
service firewalld restart
# 或
systemctl restart firewalld
```
