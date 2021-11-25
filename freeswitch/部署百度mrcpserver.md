# 部署百度智能呼叫中心的MRCPServer

---

## 官方链接

<https://cloud.baidu.com/doc/SPEECH/s/8kay0g6pq>

## 系统要求

centos7+

## 操作系统 & 必要配置

- 最小安装centos7+系统（比如7.9）
- 网卡名称配置为eth0
- 配置服务器IP
- 安装必要软件

```bash
yum install -y net-tools telnet git vim wget
```

## 部署MRCPServer

### 下载sdk

```bash
curl https://ai.baidu.com/download?sdkId=111 -L -o mrcpserver.tar.gz
```

### 解压

```bash
tar zxvf mrcpserver.tar.gz
```

### 部署配置

- 预处理

```bash
cd MRCPServer
./bootstrap.sh
```

- 配置IP和端口

```bash
cd mrcp-server
vim conf/unimrcpserver.xml
```

定位到`unimrcpserver->properties->ip`节点对绑定IP进行配置，可以有多种配置方式

```xml
<!-- 可以直接设置IP地址(客户端无论在远程还是本地都只能通过这个ip来调用) -->
<ip>183.211.245.48</ip>
<!-- 可以设置成type="auto"，由程序自动获取机器IP（默认只支持本机127.0.0.1） -->
<ip type="auto">
<!-- 可以填写网卡名称，自动获取网口的IP（少用，客户端无论在远程还是本地都只能通过这个网口的ip来调用） -->
<ip type="iface">eth0</ip>
<!-- 同时支持127.0.0.1调用和公网IP调用 -->
<ip>0.0.0.0</ip>
```

定位到`unimrcpserver->components->sip-uas->sip-port`节点，可以配置SIP端口

```xml
<!-- 软件默认SIP端口是5060，可以自定义如8060，不要与其他服务冲突（udp和tcp） -->
<sip-port>5060</sip-port>
```

定位到`unimrcpserver->components->rtsp-uas->rtsp-port`节点，可以配置RTSP端口

```xml
<!-- 软件默认RTSP端口是1554，可以自定义如8554，不要与其他服务冲突（TCP） -->
<rtsp-port>1554</rtsp-port>
```

定位到`unimrcpserver->components->mrcpv2-uas->mrcp-port`节点，可以配置mrcp端口

```xml
<!-- 软件默认MRCP端口是1544，可以自定义如8544，不要与其他服务冲突（TCP） -->
<mrcp-port>1544</mrcp-port>
```

定位到`unimrcpserver->components->rtp-factory`节点，可以配置rtp端口范围

```xml
<!-- 软件默认是5000～6000，可以自定义，注意不要和其他服务冲突（TCP） -->
<rtp-port-min>5000</rtp-port-min>
<rtp-port-max>6000</rtp-port-max>
```

- 配置asr

```bash
vim conf/mrcp-asr.conf
```

定位到`AUTH_APPID`和`AUTH_APPKEY`，这两个值分别对应百度控制台语音技术应用的AppID和API Key，网址<https://console.bce.baidu.com/ai/#/ai/speech/app/list>

```conf
# AppID（照抄无效）
AUTH_APPID : 2400000
# API Key（照抄无效）
AUTH_APPKEY : "FfMfDOdAdjBqCaLKAmNfqquW"
```

- 配置tts

```bash
vim conf/mrcp-proxy.conf
```

定位到`AUTH_APPID`和`AUTH_APPKEY`，这两个值分别对应百度控制台语音技术应用的AppID和API Key，网址<https://console.bce.baidu.com/ai/#/ai/speech/app/list>

```conf
# AppID（照抄无效）
AUTH_APPID : 2400000
# API Key（照抄无效）
AUTH_APPKEY : "FfMfDOdAdjBqCaLKAmNfqquW"
```

- 配置启动控制文件

```bash
vim conf/unimrcpserver_control.conf
```

定位到`_check_cmd_pro="./bin/check 127.0.0.1 1544"`，把127.0.0.1替换成unimrcpserver.xml配置的ip（如果auto这里就不用换了），把1544替换成在unimrcpserver.xml配置的mrcp-port

## 启动服务

### 以调试模式启动系统

做好配置后，首次启动服务建议以调试模式启动，以便可以在console里看到启动过程。

```bash
# 根据sdk保存位置不同，你的具体路径不一定跟这里一样
cd /root/MRCPServer/mrcp-server
./bin/unimrcpserver -r . &
```

正常情况下，应该会看到类似下面的输出

```console
Version: mrcp-asr | v2.0.0 | 20200609-175030 | 0e285e16 | mrcp-asr-ctrip-1.5
Version: mrcp-tts | v2.0.0 | 20200601-212708 | 24a1c9b1 | dev-liantong.mrcp1.5.0
```

> 调试模式启动后，如果想要关闭服务，需要用`ps -aux | grep mrcp`查看相关进程的PID，然后用`kill -9 PID`杀掉进程

### 以启护进程方式启动服务

在生产环境时，建使用启动脚本，以守护进程方式启动服务。

执行`${SERVER_ROOT}/mrcp-server/bin/`下的`unimrcpserver_control`并带上控制参数：

```bash
cd /root/MRCPServer/mrcp-server/bin
#启动
./unimrcpserver_control start
#停止
./unimrcpserver_control stop
#重启
./unimrcpserver_control restart
```

> restart指令必须在start状态下才可以使用，修改配置文件后需要restart才能生效，但是如果修改的是ip或者端口，restart会出错，这时就只能`ps -aux | grep mrcp`然后再`kill -9 PID`杀掉相关进程后，再发重新start

### 安全设置

MRCPServer对于客户端没有安全校验机制，如果不启用防火墙或者中是单纯的开放端口，结果可能会被任意freeswitch接入，为了避免这种情况发生，应该对MRCPServer开启防火墙并采用指定IP机制，具体操作如下：

```txt
MRCPServer服务器
IP：183.211.245.48
SIP-PORT：8060/udp, 8060/tcp
RTSP-PORT：1554/tcp
MRCP-PORT：1544/tcp
RTP：5000-6000/udp

freeswitch服务器
IP：112.4.97.6
```

在MRCP服务端配置防火墙规则如下：

```bash
# 向112.4.97.6开放sip端口8060/tcp
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="112.4.97.6" port protocol="tcp" port="8060" accept'
# 向112.4.97.6开放sip端口8060/udp
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="112.4.97.6" port protocol="udp" port="8060" accept'
# 向112.4.97.6开放rtsp端口1554/tcp
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="112.4.97.6" port protocol="tcp" port="1554" accept'
# 向112.4.97.6开放mrcp端口1544/tcp
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="112.4.97.6" port protocol="tcp" port="1544" accept'
# 向112.4.97.6开放rtp范围5000-6000/udp
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="112.4.97.6" port protocol="udp" port="5000-6000" accept'
```

### 用sdk中的客户端进行asr测试

#### 在本机测试

- 保证${SERVER_ROOT}/mrcp-server/conf/mrcp-asr.conf文件中的相关配置填写正确

```text
* AUDIO_CONTROLLER_ADDR，百度上游服务地址(默认值当前有效)
* AUTH_APPID和AUTH_APPKEY，从百度官网中获取的APPID和API Key的值。
* NEED_SAVE_AUDIO，是否保存语音识别时用户语音，默认1为保存
* AUDIO_SPLIT_TIME，是VAD判断的间隔时间（单位毫秒）
```

- 配置LD_LIBRARY_PATH变量

```bash
# 注意把${SERVER_ROOT}替换成真实路径
export LD_LIBRARY_PATH=${SERVER_ROOT}/mrcp-server/lib:$LD_LIBRARY_PATH
```

- 修改测试程序配置文件

conf/client-profiles/unimrcp.xml 是测试工具的配置文件，需要将其中的unimrcpclient->settings->sip-settings->server-ip的值修改为本机IP，端口设置为主程序端口，如5060。

- 执行测试程序

  - 切换到 ${SERVER_ROOT}/mrcp-server/bin 目录下
  - 执行 ./asrclient
  - 待测试程序启动后，在控制台输入 run grammar.xml xeq.pcm
  - 在控制台会显示识别结果，在log目录下的mrcp_debug.log中也可以看到整个识别过程

> 更多信息可参考<https://ai.baidu.com/ai-doc/SPEECH/Ekaxz1mkz>

### 用sdk中的客户端进行tts测试

待完善

### 服务开机自启动

待完善

### 配置文件说明

#### vad配置文件conf/vad.conf

```conf
sp_threshold : 0.2          # 有效声音的音量阈值（多少以上有效）
sil_threshold : 0.1         # 被认为无效声音的音量阈值（多少以下无效）
max_wait_duration : 150     # 最大等待时长
max_sp_duration : 24000     # 最长说话时间
max_sp_pause : 150          # 最大说话暂停时长
head_sil_duration : 18000   # 开头允许的最大静音时长
start_back_frame : 40
end_back_frame : 40
debug_log : 0
sample_rate : 8000
cmvnfile : global.cmvn
dnnfile : vad.dnn
package_len : 1600

vad_num : 10
```
