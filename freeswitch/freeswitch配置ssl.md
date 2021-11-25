# Freeswitch配置SSL证书

---

## 申请证书

向域名提供商申请SSL证书，然后下载证书文件，应该会得到2个文件，分别是`xxx.key`和`xxx.pem`。

## 上传证书（至freeswitch目录）

把`xxx.key`文件和`xxx.pem`文件上传到freeswitch安装目录的`certs`目录下，如`/usr/local/freeswitch/certs`。

## 合成wss.pem文件

> 先备份原有的wss.pem文件

```bash
cat xxx.pem xxx.key > wss.pem
```

## 配置wss端口

```bash
vim /usr/local/freeswitch/conf/sip_profiles/internal.xml
```

```xml
<param name="wss-binding" value=":7443"/>
```

## 重启freeswitch服务

```bash
freeswitch -stop
freeswitch -nc -nonat
```

## 客户端配置

在sip.js或jssip或其他webrtc客户端配置服务器访问地址为`wss://域名:7443`
