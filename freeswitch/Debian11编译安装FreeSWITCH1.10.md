# Debian11编译安装FreeSWITCH1.10

---

[toc]

---

## 安装操作系统

### 配IP时可能需要用到DNS信息

| 地区运营商 | DNS |
|---|---|
|江苏移动|112.4.0.55, 221.131.143.69|

### 给root配一个复杂点的密码，同时再配置一个非root的管理员账号

|账号|密码|
|---|---|
|root|BU1#0sgqX$|
|wkzz|WKzz051223|

> 江苏移动用mirrors.163.com源速度比较快

### 系统安装完成后可以进行系统升级并安装一些基础软件

```shell
apt update && apt upgrade -y
apt install -y wget git vim lrzsz net-tools
```

### 在Debian系统里想要SSH登录root账号

```shell
vim /etc/ssh/sshd_conf
```

修改内容如下：

```ini
#PermitRootLogin prohibit-password
PermitRootLogin yes

#PasswordAuthentication yes
PasswordAuthentication yes
```

修改后重启一下ssh服务

```shell
/etc/init.d/ssh restart
```

### 添加静态路由解决个别IP无法被本机PING通的问题

```shell
vim /etc/network/interfaces
```

在`iface ens192 inet static`配置段的后面添加两行

```ini
    up ip route add 183.211.245.35/32 via 183.211.245.1
    down ip route del 183.211.245.35/32 via 183.211.245.1
```

修改后的完整配置应该类似下面：

```ini
iface ens192 inet static
    address 183.211.245.51/25
    gateway 183.211.245.1
    up ip route add 183.211.245.35/32 via 183.211.245.1
    down ip route del 183.211.245.35/32 via 183.211.245.1
```

### 配置DNS

```shell
vim /etc/resolv.conf
```

文件内容如下：

```ini
nameserver 112.4.0.55
nameserver 221.131.143.69
```

修改配置后重启网卡（具体设备名称因机而异）：

```shell
ifdown ens192 && ifup ens192
```

### 在Debian系统里启用ll命令

```shell
vim ~/.bashrc
```

解除alias ll行的注释或者添加一行：

```ini
alias ll='ls $LS_OPTIONS -l --color=auto'
```

退出终端重新进入或者使用下面命令生效：

```shell
source ~/.bashrc
```

## 安装lua和luarocks环境

### 下载并安装lua5.4.4

> lua安装包的下载地址：<http://www.lua.org/ftp/>

```shell
apt install -y build-essential libreadline-dev unzip
```

```shell
cd /usr/src
wget http://www.lua.org/ftp/lua-5.4.4.tar.gz
tar zxvf lua-5.4.4.tar.gz
cd lua-5.4.4
make linux test
make install
```

> 合并命令：`cd /usr/src && wget http://www.lua.org/ftp/lua-5.4.4.tar.gz`，下载成功后再执行`tar zxvf lua-5.4.4.tar.gz && cd lua-5.4.4.tar.gz && make linux test && make install && cd ..`

### 下载并安装luarocks3.9.0

> luarocks安装包的下载地址<https://luarocks.github.io/luarocks/releases/>

```shell
cd /usr/src
wget https://luarocks.github.io/luarocks/releases/luarocks-3.9.0.tar.gz
tar zxfv luarocks-3.9.0.tar.gz
cd luarocks-3.9.0
./configure
make
make install
```

> 合并命令：`cd /usr/src && wget https://luarocks.github.io/luarocks/releases/luarocks-3.9.0.tar.gz`，下载成功后再执行 `tar zxfv luarocks-3.9.0.tar.gz && cd luarocks-3.9.0 && ./configure && make && make install && cd ..`

## 编译安装FreeSWITCH

### 安装FreeSWITCH

> 不知道后来从什么时间点开始，通过signalwire的源去安装FreeSWITCH所需依赖包时，需要使用向Signalwire平台申请的Access Token，临时提供一个`pat_jMxihv2uTh3ivpPdSqUMffB3`

```shell
# 这个token不要照抄，要替换成自己的
TOKEN=pat_jMxihv2uTh3ivpPdSqUMffB3

apt update && apt install -yq gnupg2 wget lsb-release
wget --http-user=signalwire --http-password=$TOKEN -O /usr/share/keyrings/signalwire-freeswitch-repo.gpg https://freeswitch.signalwire.com/repo/deb/debian-release/signalwire-freeswitch-repo.gpg

echo "machine freeswitch.signalwire.com login signalwire password $TOKEN" > /etc/apt/auth.conf
echo "deb [signed-by=/usr/share/keyrings/signalwire-freeswitch-repo.gpg] https://freeswitch.signalwire.com/repo/deb/debian-release/ `lsb_release -sc` main" > /etc/apt/sources.list.d/freeswitch.list
echo "deb-src [signed-by=/usr/share/keyrings/signalwire-freeswitch-repo.gpg] https://freeswitch.signalwire.com/repo/deb/debian-release/ `lsb_release -sc` main" >> /etc/apt/sources.list.d/freeswitch.list

apt update

# Install dependencies required for the build
apt build-dep freeswitch -y

# then let's get the source. Use the -b flag to get a specific branch
cd /usr/src/
git clone https://github.com/signalwire/freeswitch.git -b v1.10 freeswitch
cd freeswitch

# Because we're in a branch that will go through many rebases, it's
# better to set this one, or you'll get CONFLICTS when pulling (update).
git config pull.rebase true

# ... and do the build
./bootstrap.sh -j
./configure
make
make install
```

### 安装音频文件（英文）

```shell
cd /usr/src/freeswitch
make cd-sounds-install
make cd-moh-install
make uhd-sounds-install
make uhd-moh-install
make hd-sounds-install
make hd-moh-install
make sounds-install
make moh-install

# cd /usr/src/freeswitch && make moh-install && make sounds-install && make hd-moh-install && make hd-sounds-install && make uhd-moh-install && make uhd-sounds-install && make cd-moh-install && make cd-sounds-install
```

## 配置FreeSWITCH

### conf/vars.xml中的配置项

> 主要是sip用户默认密码、stun地址、sip端口号等配置

```shell
vim /usr/local/freeswitch/conf/vars.xml
```

```xml
<!-- 改成其他密码 -->
<X-PRE-PROCESS cmd="set" data="default_password=654654321"/>
<!-- <X-PRE-PROCESS cmd="stun-set" data="external_rtp_ip=stun:stun.freeswitch.org"/> -->
<X-PRE-PROCESS cmd="stun-set" data="external_rtp_ip=$${local_ip_v4}"/>
<!-- <X-PRE-PROCESS cmd="stun-set" data="external_sip_ip=stun:stun.freeswitch.org"/> -->
<X-PRE-PROCESS cmd="stun-set" data="external_sip_ip=$${local_ip_v4}"/>
<!--根据实际需要改成需要的端口号 -->
<X-PRE-PROCESS cmd="set" data="internal_sip_port=5060"/>
<X-PRE-PROCESS cmd="set" data="internal_tls_port=5061"/>
<X-PRE-PROCESS cmd="set" data="external_sip_port=5080"/>
<X-PRE-PROCESS cmd="set" data="external_tls_port=5081"/>
```

### conf/sip_profiles/internal.xml中的配置项

> 主要是与内部sip账号相关的配置

```shell
vim /usr/local/freeswitch/conf/sip_profiles/internal.xml
```

```xml
<!-- 默认是public，要改成default -->
<param name="context" value="default"/>
<!-- 如果要在WebRTC场景中听到早期媒体（回铃音），就要解除这一行的注释 -->
<param name="enable-100rel" value="true"/>
<!-- 如果需要WebRTC场景中使用，需要在末尾加上后面这几个配置项 -->
<param name="apply-candidate-acl" value="rfc1918.auto"/>
<param name="apply-candidate-acl" value="localnet.auto"/>
<param name="apply-candidate-acl" value="candidate"/>
```

### conf/autoload_configs/acl.conf.xml中的配置项

> 主要用来制定可（通过ESL)连接FreeSWITCH的IP规则

```shell
vim /usr/local/freeswitch/conf/autoload_configs/acl.conf.xml
```

下面的文件内容要根据实际情况配置，不能照抄。

```xml
<list name="lan" default="allow">
    <!-- 拒绝来自内网的特定网断连接 -->
    <node type="deny" cidr="192.168.42.0/24"/>
    <!-- 允许来自内网的特定IP连接 -->
    <node type="allow" cidr="192.168.42.42/32"/>
</list>
<list name="domains" default="deny">
    <node type="allow" domain="$${domain}"/>
    <!-- 允许本机连接 -->
    <node type="allow" cidr="127.0.0.1/32"/>
    <!-- 允许特定IP地址连接 -->
    <node type="allow" cidr="183.211.245.0/25"/>
    <node type="allow" cidr="112.4.97.0/25"/>
</list>
```

### conf/autoload_configs/event_socket.conf.xml中的配置项

> 用于配置ESL相关参数

```shell
vim /usr/local/freeswitch/conf/autoload_configs/event_socket.conf.xml
```

```xml
<configuration name="event_socket.conf" description="Socket Client">
  <settings>
    <param name="nat-map" value="false"/>
    <!-- listen-ip改成0.0.0.0 -->
    <param name="listen-ip" value="0.0.0.0"/>
    <!-- 端口号根据实际需要修改，默认8021 -->
    <param name="listen-port" value="8021"/>
    <!-- 密码一定要改，默认是ClueCon -->
    <param name="password" value="ClueConAAAA"/>
    <!-- 允许acl白名单内的IP 访问 -->
    <param name="apply-inbound-acl" value="domains"/>
    <!--<param name="apply-inbound-acl" value="loopback.auto"/>-->
    <!--<param name="stop-on-bind-error" value="true"/>-->
  </settings>
</configuration>
```

### conf/autoload_configs/modules.conf.xml中的配置

> 负责开启或关闭一些模块，开启模块时需要确定模块已经被正确安装

```shell
vim /usr/local/freeswitch/conf/autoload_configs/modules.conf.xml
```

```xml
<!-- av模块一般用不上，注释掉 -->
<!-- <load module="mod_av"/> -->
<!-- signalwire模块一般也用不上，注释掉 -->
<!-- <load module="mod_signalwire"/> -->
<!-- 语音信箱基本也没什么用，注释掉 -->
<!-- <load module="mod_voicemail"/> -->
```

### 关闭ipv6配置

> 为了提高一点启动效率而已，如果需要IPV6就跳过这一步

```shell
cd /usr/local/freeswitch/conf/sip_profiles/
mv external-ipv6.xml external-ipv6.xml.removed
mv internal-ipv6.xml internal-ipv6.xml.removed
```

### 做freeswitch和fs_cli的软连接

```shell
ln -sf /usr/local/freeswitch/bin/freeswitch /usr/local/bin/
ln -sf /usr/local/freeswitch/bin/fs_cli /usr/local/bin/
```

## 配置ufw防火墙

```shell
apt install -y ufw
# 基础规则
ufw default allow outgoing && ufw default deny incoming && ufw allow ssh
# FS-SIP
ufw allow 5060
# FS-SIP-EXTERNAL
ufw allow 5080
# FS-WS
ufw allow 5066/tcp
# FS-WSS
ufw allow 7443/tcp
# Event Socket
ufw allow 8021/tcp
# FS-RTP
ufw allow 16384:32768/udp
# FS-MRCP-RTP
ufw allow 4000:5000/udp
# ufw开机自启
systemctl enable ufw
# 启动ufw
ufw enable
# 检查ufw开机项
systemctl list-unit-files | grep ufw
```

## 启动FreeSWITCH

```shell
# 控制台启动，退出即关闭服务
freeswitch -nonat
# 也可以后台启动，启动后需要使用fs_cli查看控制台，需要用shutdown命令关闭服务
freeswitch -nc -nonat
```

## fs_cli连接

```shell
vim ~/.fs_cli_conf
```

```ini
[default]
host => 127.0.0.1
port => 8021
password => ClueConAAAA
debug => 6
```

做了以上配置后，直接`fs_cli`就能查看控制台，`...`退出控制台

## SIP终端注册测试

> 假设这个FreeSWITCH服务器和SIP终端的网络是互通的

**注册信息：**

|项|值|
|---|---|
|服务器IP|FreeSWITCH服务器的IP或域名|
|SIP端口号|5060，或是在vars.xml中配置的其他端口号|
|用户名|1001~1019中的任意一个分机号都行|
|密码|654654321，或是在vars.xml中配置的其他默认密码|
|网络协议|TCP或UDP都可以|

> 如果终端分机注册成功，用分机呼叫`9664`可以听到通话保持音乐，呼叫`9196`可以听到`echo`效果（你说什么他回什么），如果以上结果正常，说明本次FreeSWITCH服务器安装成功，可以开始后面的具体的开发和配置了。

## 向已安装的FreeSWITCH添加模块

### 添加mod_shout模块

> mod_shout模块可以将通话录音保存为mp3文件

先进入FreeSWITCH的源码目录

```shell
cd /usr/src/freeswitch
vim modules.conf
```

修改如下内容

```ini
# 找到mod_shout一行，解除行首的注释
formats/mod_shout
```

保存退出后，再执行下面的命令

```shell
make mod_shout-install
```

待编译安装完成后，在FreeSWITCH的安装目录`/usr/local/freeswitch`中的`mod`目录下就已经存在`mod_shout.so`这个文件了。

接下来要回到FreeSWITCH的安装目录

```shell
cd /usr/local/freeswitch
vim conf/autoload_configs/modules.conf.xml
```

编辑文件内容

```xml
<!-- 找到这一行，解除注释，如果没有就直接在后面添加 -->
<load module="mod_shout"/>
```

最后进入FreeSWITCH控制台`fs_cli`，在控制台中执行命令`load mod_shout`，到此mod_shout模块已经安装完成并在FreeSWITCH服务器生效。

### 添加mod_unimrcp模块

> mod_unimrcp模块可以通过mrcp协议实现实时语音识别和语音合成能力

先进入FreeSWITCH的源码目录

```shell
cd /usr/src/freeswitch
vim modules.conf
```

修改如下内容

```ini
# 找到mod_mod_unimrcp一行，解除行首的注释
asr_tts/mod_unimrcp
```

保存退出后，再执行下面的命令

```shell
make mod_unimrcp-install
```

待编译安装完成后，在FreeSWITCH的安装目录`/usr/local/freeswitch`中的`mod`目录下就已经存在`mod_unimrcp.so`这个文件了。

接下来要回到FreeSWITCH的安装目录

```shell
cd /usr/local/freeswitch
vim conf/autoload_configs/modules.conf.xml
```

编辑文件内容

```xml
<!-- 找到这一行，解除注释，如果没有就直接在后面添加 -->
<load module="mod_unimrcp"/>
```

最后进入FreeSWITCH控制台`fs_cli`，在控制台中执行命令`load mod_unimrcp`，到此mod_unimrcp模块已经安装完成并在FreeSWITCH服务器生效。
