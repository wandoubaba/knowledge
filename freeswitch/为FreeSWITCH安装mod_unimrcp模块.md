## 为FreeSWITCH安装mod_unimrcp模块

> wandoubaba / 2023-01-18

本文操作过程基于Debian11操作系统。自FreeSWITCH1.10.8开始，mod_unimrcp已经从FreeSWITCH主库分离，成为独立项目，因此不能再用`make mod_unimrcp-install`命令安装模块了。

### 确保FreeSWITCH已经安装成功

安装过程请参见[Debian编译安装FreeSWITCH](debian.html)。

### 安装unimrcp和依赖

下面的操作如果在root账号下，请省略`sudo`前缀。

```sh
sudo apt-get install wget tar
wget https://www.unimrcp.org/project/component-view/unimrcp-deps-1-6-0-tar-gz/download -O unimrcp-deps-1.6.0.tar.gz
tar xvzf unimrcp-deps-1.6.0.tar.gz
cd unimrcp-deps-1.6.0
# 安装apr
cd libs/apr
./configure --prefix=/usr/local/apr
make
sudo make install
cd ..
# 安装apr-util
cd apr-util
./configure --prefix=/usr/local/apr --with-apr=/usr/local/apr
make
sudo make install
cd ../../..
# 安装unimrcp
git clone https://github.com/unispeech/unimrcp.git
cd unimrcp
./bootstrap
./configure --with-sofia-sip=/usr
make
sudo make install
cd ..
```

### 安装mod_unimrcp

按照下面的程序清单执行完毕后，在FreeSWITCH的安装目录下的mod目录中会出现`mod_unimrcp.so`文件，如`/usr/local/freeswitch/mod/mod_unimrcp.so`。

```sh
cd /opt
git clone https://github.com/freeswitch/mod_unimrcp.git
cd mod_unimrcp
export PKG_CONFIG_PATH=/usr/local/freeswitch/lib/pkgconfig:/usr/local/unimrcp/lib/pkgconfig
./bootstrap.sh
./configure
make
sudo make install
```

### 在FreeSWITCH配置中启用mod_unimrcp

编辑配置文件`/usr/local/freeswitch/conf/autoload_configs/modules.conf.xml`，在`configuration->modules`节点下，追加下面一行配置：

```xml
<load module="mod_unimrcp"/>
```

如果FreeSWITCH是已启动状态，在fs控制台执行load mod_unimrcp即可加载模块。

此时执行`load mod_unimrcp`后，有可能会看到`mod_unimrcp.c:3893 Could not open unimrcp.conf`之类的报错，那是因为我们还没有对unimrcp模块做具体配置，后面会有专门的文档介绍unimrcp模块的具体使用。
