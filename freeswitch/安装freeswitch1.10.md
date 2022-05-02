# 基于centos7.9(2009)安装freeswitch1.10

---

## 安装centos7.9(2009)操作系统

略

## 系统更新和安装依赖

```bash
yum update -y
# 安装fs依赖
yum install -y http://files.freeswitch.org/freeswitch-release-1-6.noarch.rpm epel-release
# 安装ffmpeg需要
rpm --import http://li.nux.ro/download/nux/RPM-GPG-KEY-nux.ro
rpm -Uvh http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-1.el7.nux.noarch.rpm

yum install -y yum-utils git gcc gcc-c++ automake autoconf libtool libtiff-devel libjpeg-devel openssl-devel vim

yum install -y alsa-lib-devel bison broadvoice-devel bzip2 curl-devel libdb4-devel e2fsprogs-devel erlang flite-devel g722_1-devel gdbm-devel gnutls-devel ilbc2-devel ldns-devel libcodec2-devel libcurl-devel libedit-devel libidn-devel libmemcached-devel libogg-devel libsilk-devel libsndfile-devel libtheora-devel libuuid-devel libvorbis-devel libxml2-devel lua-devel lzo-devel ncurses-devel net-snmp-devel opus-devel pcre-devel perl perl-ExtUtils-Embed pkgconfig portaudio-devel postgresql-devel python-devel python-devel soundtouch-devel speex-devel sqlite-devel unbound-devel unixODBC-devel which yasm zlib-devel libshout-devel libmpg123-devel lame-devel rpm-build libX11-devel libyuv-devel swig wget ffmpeg ffmpeg-devel

# 单独下载spandsp源码
cd /usr/local/src
git clone https://github.com/freeswitch/spandsp.git
cd spandsp
./bootstrap.sh
./configure
make
make install
ldconfig

# 单独下载sofia-sip（SIP协议栈）源码
cd /usr/local/src
git clone https://github.com/freeswitch/sofia-sip.git
cd sofia-sip
./bootstrap.sh
./configure
make
make install
ldconfig
cd ..

# 编译安装cmake 3.7.2
cd /usr/local/src
wget https://cmake.org/files/v3.7/cmake-3.7.2.tar.gz
tar zxvf cmake-3.7.2.tar.gz
cd cmake-3.7.2
./bootstrap
gmake
gmake install

# 安装libatomic
yum install -y libatomic uuid-devel libuuid-devel

# 单独下载libks源码（需要cmake 3.7.2以上版本）
cd /usr/local/src
git clone https://github.com/signalwire/libks.git
cd libks
cmake .
make
make install

# 单独安装opus-dev（否则在freeswitch里make时可能会报You must install libopus-dev to build mod_opus）
cd /usr/local/src
wget https://archive.mozilla.org/pub/opus/opus-1.3.1.tar.gz
tar zxvf opus-1.3.1.tar.gz
cd opus-1.3.1
./configure
make
make install

export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig

# 安装python组件
curl https://bootstrap.pypa.io/pip/2.7/get-pip.py --output get-pip-2.7.py
python get-pip-2.7.py -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com
#这两个命令用完之后使用pip install 后面要加上
-i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com
# 验证pip是否安装成功可以用 `pip --version`
# pip安装python组件
pip install pydub -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com
pip install python-ESL -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com
pip install pika -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com
pip install DBUtils==2.0.3  -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com
# pip install dbutils （python 2.7 is not support dbutils炸裂）
```

## 开始安装

```bash
cd /usr/local/src
git clone -b v1.10 https://github.com/signalwire/freeswitch.git freeswitch
# 如果github连接不顺畅的话，可以试试码云镜像仓库（更新慢1天）
# git clone -b v1.10 https://gitee.com/mirrors/FreeSWITCH.git freeswitch
cd freeswitch
./bootstrap.sh -j
```

- 编辑modules.conf文件

```bash
vim modules
```

根据需要打开或关闭注释

```conf
formats/mod_shout
languages/mod_python
#event_handlers/mod_cdr_pg_csv
asr_tts/mod_unimrcp
#开启freeswitch自带的http状态界面
xml_int/mod_xml_rpc
```

如果需要使用mod_xml_curl的话

```conf
xml_int/mod_xml_curl
```

给不需要的模块加上注释

```conf
#applications/mod_av
#applications/mod_signalwire
```

然后保存退出

- 开始编译安装

```bash
export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig
./configure --with-python=/usr/bin/python2.7 --with-lua=/usr/bin/lua --enable-core-pgsql-support
# 如果在spandsp位置报错，可能是忘export那一句
make
#make mod_cdr_pg_csv-install
make mod_unimrcp-install
# 如果需要xml_curl模块的话
make mod_xml_curl-install
make mod_xml_rpc-install
make install
```

- 安装音频文件（英文）

```bash
make cd-sounds-install && \
make cd-moh-install && \
make uhd-sounds-install && \
make uhd-moh-install && \
make hd-sounds-install && \
make hd-moh-install && \
make sounds-install && \
make moh-install

# make moh-install && make sounds-install && make hd-moh-install && make hd-sounds-install && make uhd-moh-install && make uhd-sounds-install && make cd-moh-install && make cd-sounds-install
```

- 建立软连接

```bash
sudo ln -sf /usr/local/freeswitch/bin/freeswitch /usr/local/bin/
sudo ln -sf /usr/local/freeswitch/bin/fs_cli /usr/local/bin/
```

- 配置mod

```bash
sudo vim /usr/local/freeswitch/conf/autoload_configs/modules.conf.xml
```

在前3行开启

```xml
    <load module="mod_console"/>
    <load module="mod_logfile"/>
    <load module="mod_xml_curl"/>
```

打开注释

```xml
    <load module="mod_python"/>
    <load module="mod_shout"/>
    <load module="mod_xml_rpc"/>
```

添加配置

```xml
    <!-- <load module="mod_cdr_pg_csv"/> -->
    <load module="mod_unimrcp"/>
    <!--<load module="mod_vad"/>-->
```

注释掉其他不需要的模块

```xml
    <!-- <load module="mod_av"/> -->
    <!-- <load module="mod_signalwire"/> -->
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

- 适配WebRTC（JSSIP/SIPJS）

在`/usr/local/freeswitch/conf/sip_profiles/internal.xml`中添加或修改下面这些配置

```xml
    <param name="apply-candidate-acl" value="rfc1918.auto"/>
    <param name="apply-candidate-acl" value="localnet.auto"/>
    <param name="apply-candidate-acl" value="candidate"/>
```

```xml
    <!-- 取消注释这一行（让前端可以得到早期媒体） -->
    <param name="enable-100rel" value="true"/>
```

- 适配特定终端（以云翌通安卓SDK为例）

在`/usr/local/freeswitch/conf/sip_profiles/internal.xml`中添加下面这些配置

```xml
    <param name="user-agent-string" value="YunEasy"/>
```

- 拨号计划规则

在`/usr/local/freeswitch/conf/sip_profiles/internal.xml`中修改下面这些配置

```xml
    <!-- 默认是public -->
    <param name="context" value="default"/>
```

## 安全配置

- 配置端口

在`/usr/local/freeswitch/conf/vars.xml`文件中：

```xml
<!-- sip端口：终端通过tcp协议连接到这个端口，默认5060 -->
<X-PRE-PROCESS cmd="set" data="internal_sip_port=5060"/>
<!-- tls端口：终端通过tls协议连接到这个端口，默认5061 -->
<X-PRE-PROCESS cmd="set" data="internal_tls_port=5061"/>
```

```xml
<!-- 把原来的stun协议的地址改成下面的内容 -->
<X-PRE-PROCESS cmd="stun-set" data="external_rtp_ip=xxx.xxx.xxx.xxx"/>
<X-PRE-PROCESS cmd="stun-set" data="external_sip_ip=*.*.*.*"/>
```

在`/usr/local/freeswitch/conf/sip_profiles/internal.xml`文件中：

```xml
<!-- ws端口：通过ws协议使用webrtc时需要连接到这个端口，默认5066 -->
<param name="ws-binding"  value=":5066"/>
<!-- wss端口：通过wss协议使用webrtc时需要连接到这个端口，默认7443 -->
<param name="wss-binding" value=":7443"/>
```

- 默认密码

在`/usr/local/freeswitch/conf/vars.xml`文件中：

```xml
<!-- 初始的1000~1019分机使用的默认密码，建议修改 -->
<X-PRE-PROCESS cmd="set" data="default_password=1234"/>
```

- 配置防火墙

```bash
# 开放sip端口tcp协议
sudo firewall-cmd --permanent --add-port=5060/tcp
# 开放sip端口udp协议
sudo firewall-cmd --permanent --add-port=5060/udp
# 开放ws端口
sudo firewall-cmd --permanent --add-port=5066/tcp
# 开放wss端口
sudo firewall-cmd --permanent --add-port=7443/tcp
# 开放rtp端口（范围）
sudo firewall-cmd --permanent --add-port=16384-32768/udp
# 让防火墙配置生效
sudo firewall-cmd --reload
```

- 参考资料

|FireWall Ports|Network Protocol|Application Protocol|Description|
|---|---|---|---|
|1719|UDP|H.323|Gatekeeper RAS port|
|1720|TCP|H.323|Call Signaling|
|3478|UDP|STUN service|Used for NAT traversal|
|3479|UDP|STUN service|Used for NAT traversal|
|5002|TCP|MLP|protocol server|
|5003|UDP| |Neighborhood service|
|5060|UDP & TCP|SIP UAS|Used for SIP signaling (Standard SIP Port, for default Internal Profile)|
|5070|UDP & TCP|SIP UAS|Used for SIP signaling (For default "NAT" Profile)|
|5080|UDP & TCP|SIP UAS|Used for SIP signaling (For default "External" Profile)|
|8021|TCP|ESL|Used for mod_event_socket *|
|16384-32768|UDP|RTP/ RTCP multimedia streaming|Used for audio/video data in SIP and other protocols|
|5066|TCP|Websocket|Used for WebRTC|
|7443|TCP|Websocket|Used for WebRTC|

## 效率

- 关闭ipv6

在`/usr/local/freeswitch/conf/sip_profiles/`目录下

```bash
cd /usr/local/freeswitch/conf/sip_profiles
mv internal-ipv6.xml internal-ipv6.xml.removed
mv external-ipv6.xml external-ipv6.xml.removed
```

## 启动

- 后台快速启动

```bash
freeswitch -nc -nonat
```

- 控制台启动（退出即关闭服务）

```bash
freeswitch
```
