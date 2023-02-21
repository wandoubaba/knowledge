## Debian11安装lua5.2和luarocks

> wandoubaba / 2023-01-18

严格来说，这个内容并不只针对FreeSWITCH，不过打造一个合适的lua的编写和运行环境也是玩转FreeSWITCH的一个前提，而且笔者是通过FreeSWITCH才认识lua的，所以把搭建lua环境也当作是FreeSWITCH的一个案例看待。

### 安装lua5.2

> 后面的操作过程中如果使用root账号的话就不需要加`sudo`了

```sh
sudo apt-get install -y lua5.2
```

### 安装luarocks

```sh
sudo apt-get install -y unzip
cd /opt
wget https://luarocks.org/releases/luarocks-3.9.2.tar.gz
tar zxpf luarocks-3.9.2.tar.gz
cd luarocks-3.9.2
./configure && make
sudo make install
```

### 验证

```sh
# 查看lua版本号
lua -v
# 查看luarocks版本号
luarocks --version
```
