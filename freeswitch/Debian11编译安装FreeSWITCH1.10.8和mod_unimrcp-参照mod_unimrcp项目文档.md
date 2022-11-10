# Debian11编译安装FreeSWITCH1.10.8和mod_unimrcp-参照mod_unimrcp项目文档

## 安装Debian11系统

略

> 至少要安装vim git wget

## 下载/克隆各种资源

```bash
cd /usr/src
git clone https://github.com/signalwire/freeswitch
git clone https://github.com/signalwire/libks
git clone https://github.com/freeswitch/sofia-sip
git clone https://github.com/freeswitch/spandsp
git clone https://github.com/signalwire/signalwire-c
git clone https://github.com/unispeech/unimrcp.git
git clone https://github.com/freeswitch/mod_unimrcp.git
wget https://www.unimrcp.org/project/component-view/unimrcp-deps-1-6-0-tar-gz/download -O unimrcp-deps-1.6.0.tar.gz
# 国内连接github很累，另外不保证各资源仓库以后更新后对应的操作方法是否会变，建议资源下载成功后自己留一份备份
# 后面的操作如果在root账号下就不要再用sudo了
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
    libshout3-dev libmpg123-dev libmp3lame-dev\
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
# 安装FreeSWITCH
cd freeswitch
./bootstrap.sh -j
./configure
make -j`nproc`
sudo make install
cd ..
# 安装unimrcp-deps包
tar zxvf unimrcp-deps-1.6.0.tar.gz
cd unimrcp-deps-1.6.0
cd libs/apr
./configure --prefix=/usr/local/apr
make
sudo make install
cd ..
cd apr-util
./configure --prefix=/usr/local/apr --with-apr=/usr/local/apr
make
sudo make install
cd ../../..
# 安装unimrcp
cd unimrcp
./bootstrap
./configure --with-sofia-sip=/usr
make
sudo make install
cd ..
# 安装mod_unimrcp
cd mod_unimrcp
export PKG_CONFIG_PATH=/usr/local/freeswitch/lib/pkgconfig:/usr/local/unimrcp/lib/pkgconfig
./bootstrap.sh
./configure
make
sudo make install
```
