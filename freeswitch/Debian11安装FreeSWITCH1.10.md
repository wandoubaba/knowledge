## Debian编译安装FreeSWITCH

> wandoubaba / 2023-01-15

本文以Debian11和FreeSWITCH1.10为例，介绍一步一步编译安装FreeSWITCH的方法。

### 先下载/克隆各种资源

```shell
# 假设所有资源都下载到/opt/目录下
cd /opt
# 下载FreeSWITCH源码
git clone -b v1.10 https://github.com/signalwire/freeswitch freeswitch
# 下载libks源码
git clone https://github.com/signalwire/libks
# 下载sofia-sip源码
git clone https://github.com/freeswitch/sofia-sip
# 下载spandsp源码
git clone https://github.com/freeswitch/spandsp
# 下载signalwire-c源码
git clone https://github.com/signalwire/signalwire-c
```

> 国内连接github很累，另外不保证各资源仓库以后更新后对应的操作方法是否会变，建议资源下载成功后自己留一份备份

### 一步一步安装

> 如果后面的操作是在root账号下，就不需要再用sudo了

先安装必要的依赖程序。

```shell
# 安装FreeSWITCH需要的依赖
sudo apt-get install -y \
    build-essential cmake automake autoconf libtool libtool-bin pkg-config \
    libssl-dev zlib1g-dev libdb-dev unixodbc-dev libncurses5-dev libexpat1-dev libgdbm-dev bison erlang-dev libtpl-dev libtiff5-dev uuid-dev \
    libpcre3-dev libedit-dev libsqlite3-dev libcurl4-openssl-dev nasm \
    libogg-dev libspeex-dev libspeexdsp-dev \
    libldns-dev \
    python3-dev \
    libavformat-dev libswscale-dev libavresample-dev \
    liblua5.2-dev \
    libopus-dev \
    libpq-dev \
    libshout3-dev libmpg123-dev libmp3lame-dev \
    libsndfile1-dev libflac-dev libogg-dev libvorbis-dev
# 安装libks
cd libks
cmake . -DCMAKE_INSTALL_PREFIX=/usr -DWITH_LIBBACKTRACE=1
sudo make install
cd ..
# 安装sofia-sip
cd sofia-sip
./bootstrap.sh
./configure CFLAGS="-g -ggdb" --with-pic --with-glib=no --without-doxygen --disable-stun --prefix=/usr
make -j`nproc --all`
sudo make install
cd ..
# 安装spandsp
cd spandsp
./bootstrap.sh
./configure CFLAGS="-g -ggdb" --with-pic --prefix=/usr
make -j`nproc --all`
sudo make install
cd ..
# 安装signalwire-c
cd signalwire-c
PKG_CONFIG_PATH=/usr/lib/pkgconfig cmake . -DCMAKE_INSTALL_PREFIX=/usr
sudo make install
cd ..
```

可以开始安装FreeSWITCH了

```shell
# 编译安装FreeSWITCH
cd freeswitch
./bootstrap.sh -j
./configure
make -j`nproc`
sudo make install
# 安装英文声音资源（可选）
make cd-sounds-install
make cd-moh-install
make uhd-sounds-install
make uhd-moh-install
make hd-sounds-install
make hd-moh-install
make sounds-install
make moh-install
cd ..
```

### 启动FreeSWITCH服务

执行以上步骤后，FreeSWITCH已经被安装到/usr/local/freeswitch目录下了。

```shell
cd /usr/local/freeswitch
# 前台启动服务
bin/freeswitch
```

等待一段时间后，FreeSWITCH服务就已经成功启动了，在当前控制台输入命令`sofia status`可以看到一点配置信息。前台启动方式非常简单，但是一旦执行`...`命令退出控制台后，对应的FreeSWITCH服务也就退出了。如果想在后台启动服务，在执行`bin/freeswitch`时后面加上`-nc`命令参数就可以了。
