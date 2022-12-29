# ubuntu16.04安装FreeSWITCH1.6

## 基本安装

### （可省）更换阿里源

```shell
vim /etc/apt/source.list
```

```plaint
deb https://mirrors.aliyun.com/ubuntu/ xenial main
deb-src https://mirrors.aliyun.com/ubuntu/ xenial main
deb https://mirrors.aliyun.com/ubuntu/ xenial-updates main
deb-src https://mirrors.aliyun.com/ubuntu/ xenial-updates main
deb https://mirrors.aliyun.com/ubuntu/ xenial universe
deb-src https://mirrors.aliyun.com/ubuntu/ xenial universe
deb https://mirrors.aliyun.com/ubuntu/ xenial-updates universe
deb-src https://mirrors.aliyun.com/ubuntu/ xenial-updates universe
deb https://mirrors.aliyun.com/ubuntu/ xenial-security main
deb-src https://mirrors.aliyun.com/ubuntu/ xenial-security main
deb https://mirrors.aliyun.com/ubuntu/ xenial-security universe
deb-src https://mirrors.aliyun.com/ubuntu/ xenial-security universe
```

### 系统更新

```shell
apt update && apt upgrade -y
```

### 安装依赖

```shell
apt-get install -y git build-essential automake autoconf libtool g++ zlib1g-dev libjpeg-dev libncurses5-dev libsqlite3-dev libcurl4-openssl-dev libpcre3-dev libspeex-dev libspeexdsp-dev libspeex-dev libldns-dev libedit-dev libssl-dev pkg-config yasm lua5.2 liblua5.2-dev liblua5.2 libopus-dev libsndfile-dev libtool libpq-dev pkg-config libtiff5-dev libtiff5 libvpx-dev libvpx3 libvpx3 libopus-dev uuid-dev libsndfile-dev ffmpeg python python-dev libmpg123-dev libshout3-dev libmp3lame-dev
```

### 安装cmake

> 这个过程比较消耗时间

```shell
cd /opt
wget https://github.com/Kitware/CMake/releases/download/v3.25.1/cmake-3.25.1.tar.gz
tar zxvf cmake-3.25.1.tar.gz
cd cmake-3.25.1
./bootstrap
make
make install
```

### 安装FreeSWITCH

> github可能经常连不上，所以git clone这一步可以在安装依赖或者安装cmake时开新终端同步来搞，多试几次

```shell
cd /opt
git clone -b v1.6 https://github.com/signalwire/freeswitch.git
git clone https://github.com/signalwire/libks.git
git clone https://github.com/signalwire/signalwire-c.git
# 编译freeswitch
cd freeswitch
./bootstrap.sh -j
./configure
cd ..
# 编译安装libks
cd libks/
cmake .
make
make install
cd ..
# 编译安装signalwire-c
cd signalwire-c
cmake .
make
make install
cp /usr/local/lib/pkgconfig/.pc /usr/lib/pkgconfig/
cp -f /usr/local/lib/ /lib64/
cd ..
# 安装freeswitch
cd freeswitch
make
make install
ln -s /usr/local/freeswitch/bin/* /usr/bin/
```

### 安装声音

```bash
make sounds-install && \
make moh-install && \
make cd-sounds-install && \
make cd-moh-install && \
make uhd-sounds-install && \
make uhd-moh-install
```

### 安装模块

```bash
cd /opt/freeswitch
vim modules.conf
# 为下面这几行解除注释
# mod_unimrcp
# mod_shout
# mod_python
# mod_xml_curl
```
