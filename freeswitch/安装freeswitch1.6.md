# 初步在ubuntu16.04系统上安装freeswitch1.6

------

## 1. 操作系统和软件版本

ubuntu16.04

freeswitch v1.6

python2.7

lua5.3

## 2. 准备工作

### 安装系统

最小安装ubuntu16.04系统，安装ssh server

### 网络等必要配置

根据实际网络环境和个要习惯来配置

- 静态IP：

```bash
sudo vim /etc/network/interfaces
```

示例（根据实际情况设置，照抄大概率无效）

```txt
auto eth0
iface eth0 inet static
address 192.168.0.234
netmask 255.255.255.0
gateway 192.168.0.1
dns-nameserver 219.148.204.66
dns-nameserver 219.149.6.99
dns-nameserver 114.114.114.114
```

- root登录：

```bash
sudo vim /etc/ssh/sshd_config
```

- root密码：

```bash
sudo passwd root
```

- 修改更新源：

```bash
sudo vim /etc/apt/sources.list
```

```txt
deb http://mirrors.aliyun.com/ubuntu/ xenial main
deb-src http://mirrors.aliyun.com/ubuntu/ xenial main

deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-updates main

deb http://mirrors.aliyun.com/ubuntu/ xenial universe
deb-src http://mirrors.aliyun.com/ubuntu/ xenial universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates universe
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-updates universe

deb http://mirrors.aliyun.com/ubuntu/ xenial-security main
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-security main
deb http://mirrors.aliyun.com/ubuntu/ xenial-security universe
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-security universe
```

> 在mirrors.aliyun.com里可以查看具体配置方法

- 更新系统

```bash
apt update && apt upgrade -y
```

### 安装依赖

- apt部分

```bash
sudo apt install python-dev swig ffmpeg yasm unixodbc-dev libshout3-dev libmpg123-dev libmp3lame-dev libsndfile-dev autoconf automake devscripts libopus-dev libvorbis0a libogg0 libogg-dev libvorbis-dev gawk g++ git-core libjpeg-dev libncurses5-dev libtool-bin pkg-config libtiff5-dev libperl-dev libgdbm-dev libdb-dev gettext libssl-dev libcurl4-openssl-dev libpcre3-dev libspeex-dev libspeexdsp-dev libsqlite3-dev libedit-dev libldns-dev libpq-dev
```

如果需要开启mod_lua模块，还需要安装lua

```bash
sudo apt install lua5.3 liblua5.3-dev
```

- pip安装

```bash
wget https://bootstrap.pypa.io/pip/2.7/get-pip.py
python get-pip.py
pip install pydub
pip install python-ESL
pip install pika
pip install dbutils
```

### 下载源码

```bash
git clone https://github.com/signalwire/freeswitch.git
cd freeswitch
git checkout v1.6
git remote rm origin
```

> 最后一句是断开本地目录和远程代码库的关联

## 3. 安装freeswitch

- 配置lua（如果不需要mod_lua模块，可跳过）

```bash
cp /usr/include/lua5.3/*.h src/mod/languages/mod_lua/
```

```bash
sudo ln -s /usr/lib/x86_64-linux-gnu/liblua5.3.so /usr/lib/x86_64-linux-gnu/liblua.so
```

- 预处理

```bash
sudo ./bootstrap.sh -j
```

- 预配置模块

```bash
vim modules.conf
```

打开注释

```conf
formats/mod_shout
languages/mod_python
event_handlers/mod_cdr_pg_csv
asr_tts/mod_unimrcp
```

如果不需要使用lua语言模块，则将下面内容加注释

```conf
#languages/mod_lua
```

- 预编译

```bash
./configure --with-python=/usr/bin/python2.7 --with-lua=/usr/bin/lua5.3 --enable-core-pgsql-support
```

- 编译安装

```bash
sudo make
sudo make mod_cdr_pg_csv-install
sudo make mod_unimrcp-install
sudo make install
```

- 安装声音包

```bash
sudo make sounds-install
sudo make moh-install
sudo make cd-sounds-install
sudo make cd-moh-install
sudo make uhd-sounds-install
sudo make uhd-moh-install
```

- 建立软连接

```bash
sudo ln -sf /usr/local/freeswitch/bin/freeswitch /usr/local/bin/
sudo ln -sf /usr/local/freeswitch/bin/fs_cli /usr/local/bin/
```

- 配置mod

```bash
vim /usr/local/freeswitch/conf/autoload_configs/modules.conf.xml
```

打开注释

```xml
    <load module="mod_python"/>
    <load module="mod_shout"/>
```

添加配置

```xml
    <load module="mod_cdr_pg_csv"/>
    <load module="mod_unimrcp"/>
    <load module="mod_vad"/>
```

如果不需要lua支持，则注释下面内容

```xml
    <!-- <load module="mod_lua"> -->
```

- 配置acl白名单

```bash
vim /usr/local/freeswitch/conf/autoload_configs/acl.conf.xml
```

根据自己网络的实际情况进行配置（照抄大概率无效）

```xml
<list name="domains" default="deny">
<!-- domain= is special it scans the domain from the directory to build t$ -->
<node type="allow" domain="$${domain}"/>
<!-- use cidr= if you wish to allow ip ranges to this domains acl. -->
<!-- <node type="allow" cidr="192.168.0.0/24"/> -->
    <node type="allow" cidr="192.168.0.112/24"/>
    <node type="allow" cidr="192.168.0.50/24"/>
<!-- ==================这里添加 本机ip   127.0.0.1 ======================== -->
<!-- ==================这里添加 本机内网ip   ======================== -->
<!-- ==================这里添加 本机外网ip   ======================== -->
<!-- ==================这里添加 web内网ip   192.168.2.173======================== -->
<!-- ==================这里添加 web外网ip   39.107.68.127======================== -->
<!-- ==================这里添加  runcall  内外网Ip======================== -->
    <node type="allow" cidr="192.168.2.178/24"/>
    <node type="allow" cidr="39.107.70.84/24"/>
</list>
```

- 配置ESL

```bash
vim /usr/local/freeswitch/conf/autoload_configs/event_socket.conf.xml
```

```xml
<configuration name="event_socket.conf" description="Socket Client">
        <settings>
            <param name="nat-map" value="false"/>
            <!--ip 统一为0.0.0.0-->
            <param name="listen-ip" value="0.0.0.0"/>
            <!-- 端口号 默认8021 -->
            <param name="listen-port" value="8021"/>
            <!-- 密码统一Aicyber -->
            <param name="password" value="Aicyber"/>
            <!-- 允许acl白名单内的IP 访问 -->
            <param name="apply-inbound-acl" value="domains"/>
            <!--<param name="apply-inbound-acl" value="loopback.auto"/>-->
            <!--<param name="stop-on-bind-error" value="true"/>-->
        </settings>
</configuration>
```

- 配置VAD（可暂时省略）

把mod_vad.so（到已经配置好的服务器上去拉）和mod_G729.so（已经有了）放到/usr/local/freeswitch/mod目录下，并给可执行权

```bash
chmod +x mod_vad.so
```

- 配置网关（可暂时省略）

在/usr/local/freeswitch/conf/sip_profiles/external 目录下上传网关模板gw1.xml（这个模板在动态生成xml时会用到）

手动添加网关配置（可暂时省略）

```xml
<include>
    <gateway name="gw1"><!--PSTN -->
        <param name="realm" value="192.168.0.153"/>
        <param name="register" value="false"/>
    </gateway>
</include>
```

## 4. 启动freeswitch

```bash
freeswitch -nc
```

通过fs_cli可以进入freeswitch控制台（freeswitch服务启动需要等一段时间，而且可能会很长）

```bash
fs_cli --password=Aicyber
```
