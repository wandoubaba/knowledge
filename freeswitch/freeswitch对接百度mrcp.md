# Freeswitch通过mod_unimrcp与百度mrcp-server对接（lua版）

---

## 参考链接

[https://freeswitch.org/confluence/display/FREESWITCH/mod_unimrcp]
[https://ptorch.com/news/206.html]
[https://ptorch.com/news/207.html]

## 安装并加载mod_unimrcp模块

```bash
# 在freeswitch源码目录（不是安装目录）
make mod_unimrcp-install
# 在freeswitch安装目录中编译modules.conf.xml文件
cd /usr/local/freeswitch
vim conf/autoload_configs/modules.conf.xml
```

```xml
<!-- 添加如下配置 -->
<load module="mod_unimrcp"/>
```

## 设置profile文件和conf文件

```bash
vim /usr/local/freeswitch/conf/mrcp_profiles/unimrcpserver-mrcp-v2.xml
```

输入以下内容：

```xml
<include>
    <!-- UniMRCP Server MRCPv2 -->
    <!-- 后面我们使用该配置文件，均使用 name 作为唯一标识，而不是文件名 -->
    <profile name="unimrcpserver-mrcp2" version="2">
    <!-- MRCP 服务器地址和SIP端口号 -->
    <param name="server-ip" value="192.168.16.4"/>
    <!-- mrcp服务器的sip-port -->
    <param name="server-port" value="15060"/>
    <param name="resource-location" value=""/>

    <!-- FreeSWITCH IP、端口以及 SIP 传输方式 -->
    <param name="client-ip" value="192.168.16.4" />
    <!-- freeswitch的sip-port -->
    <param name="client-port" value="5069"/>
    <param name="sip-transport" value="udp"/>

    <param name="speechsynth" value="speechsynthesizer"/>
    <param name="speechrecog" value="speechrecognizer"/>
    <!--param name="rtp-ext-ip" value="auto"/-->
    <!-- 也是freeswitch的ip和rtp端口范围（不是mrcp里配置的rtp范围） -->
    <param name="rtp-ip" value="192.168.16.4"/>
    <param name="rtp-port-min" value="4000"/>
    <param name="rtp-port-max" value="5000"/>
    <param name="codecs" value="PCMU PCMA L16/96/8000"/>

    <!-- Add any default MRCP params for SPEAK requests here -->
    <synthparams>
    </synthparams>

    <!-- Add any default MRCP params for RECOGNIZE requests here -->
    <recogparams>
      <!--param name="start-input-timers" value="false"/-->
    </recogparams>
    </profile>
</include>
```

编辑`unimrcp.conf.xml`文件改`default-tts-profile`和`default-asr-profile`

```bash
vim /usr/local/freeswitch/conf/autoload_configs/unimrcp.conf.xml
```

```xml
<!-- UniMRCP profile to use for TTS -->
<param name="default-tts-profile" value="unimrcpserver-mrcp2"/>
<!-- UniMRCP profile to use for ASR -->
<param name="default-asr-profile" value="unimrcpserver-mrcp2"/>
```

## 设置dialplan

```bash
vim /usr/local/freeswitch/conf/dialplan/default.xml
```

添加一个extension：

```xml
<extension name="unimrcp">
    <condition field="destination_number" expression="^5001$">
        <action application="answer"/>
        <!-- 对应scripts/baidu.lua文件 -->
        <action application="lua" data="baidu.lua"/>
    </condition>
</extension>
```

在`/usr/local/freeswitch/scripts`目录下创建`baidu.lua`文件：

```bash
touch /usr/local/freeswitch/scripts/baidu.lua
vim /usr/local/freeswitch/scripts/baidu.lua
```

文件内容如下：

```lua
session:answer()

--freeswitch.consoleLog("INFO", "Called extension is '".. argv[1]"'\n")
welcome = "/usr/local/freeswitch/sounds/en/us/callie/ivr/8000/ivr-welcome_to_freeswitch.wav"
--
grammar = "baidu"
no_input_timeout = 80000
recognition_timeout = 80000
--

tryagain = 1
while (tryagain == 1) do
--
    session:execute("play_and_detect_speech", welcome .. " detect:unimrcp {start-input-timers=false,no-input-timeout=" .. no_input_timeout .. ",recognition-timeout=" .. recognition_timeout .. "} " .. grammar)
    xml = session:getVariable('detect_speech_result')
 --
    if (xml == nil) then
        freeswitch.consoleLog("CRIT","Result is 'nil'\n")
        tryagain = 0
    else
        freeswitch.consoleLog("CRIT","Result is '" .. xml .. "'\n")
        tryagain = 0
    end
end
--
-- put logic to forward call here
--
session:sleep(250)
session:hangup()
```

> 以上脚本实现当分机用户拨打5001时，freeswitch会自动播放一段录音，并接收用户发出的声音，同时把声音传给mrcp服务器并接收返回结果

在`/usr/local/freeswitch/grammar`目录新增`hello.gram`语法文件，内容为百度mrcp程序句中的语法文件内容：

```xml
<?xml version="1.0"?>
<grammar xmlns="http://www.w3.org/2001/06/grammar" xml:lang="en-US" version="1.0" mode="voice" root="digit">
  <rule id="digit">
    <one-of>
      <item>one</item>
      <item>two</item>
      <item>three</item>
    </one-of>
  </rule>
</grammar>
```

## 让配置生效并测试

```bash
fs_cli
reloadxml
```

## 防火墙

在freeswitch服务器和mrcp服务器都不需要额外开放端口。
