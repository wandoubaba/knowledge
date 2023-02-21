## FreeSWITCH对接MRCP服务（以阿里云SDM为例）

> wandoubaba / 2023-02-21

在阿里云的的智能语音服务中，除了提供常规的ASR、TTS等能力外，也有完整的MRCP服务（阿里名称：SDM），不但可以对接私有化的智能语音能力，同时还可以对接公有云的ASR和TTS。

### 网络规划

- MRCP服务器

|配置项|配置值|
|---|---|
|IP| 192.168.0.10|
|SIP端口|7010（TCP&UDP）|
|MRCP端口|1544、1554（TCP）|
|RTP端口|10000-20000（UDP）|

- FreeSWITCH服务器

|配置项|配置值|
|---|---|
|IP|192.168.0.60|
|SIP端口|5060（TCP&UDP）|
|RTP端口（与MRCP通信）|40000-50000（UDP）|

### 部署SDM

目前阿里云并没有开放SDM服务，通过阿里云智能语音服务的客户经理可以得以《SDM(MRCP-SERVER)公共镜像使用》文档，按照文档部署，比较容易。

摘抄阿里云《SDM(MRCP-SERVER)公共镜像使用》文档中的一段话：

> SDM(MRCP-SERVER)目前在公共云上有对应的镜像仓库，用户可以直接拉取公共云镜像到本地，然后部署使用，一定程度上简化了部署和接入的成本。

### 安装mod_unimrcp模块

详见《为FreeSWITCH安装mod_unimrcp模块》。

### 配置mod_unimrcp

在FreeSWITCH的安装目录修改`conf/autoload_configs/modules.conf.xml`文件：

```xml
<!-- 添加一行 -->
<load module="mod_unimrcp"/>
```

在FreeSWITCH的安装目录下创建`conf/mrcp_profiles/aliyun-mrcpserver.xml`文件：

```xml
<include>
  <profile name="aliyun-mrcpserver" version="2">
    <param name="client-ip" value="$${local_ip_v4}"/>
    <param name="client-port" value="5060"/>
    <param name="server-ip" value="192.168.0.10"/>
    <param name="server-port" value="7010"/>
    <param name="resource-location" value=""/>
    <param name="sip-transport" value="tcp"/>
    <param name="sdp-origin" value="Freeswitch"/>
    <param name="rtp-ip" value="$${local_ip_v4}"/>
    <param name="rtp-port-min" value="40000"/>
    <param name="rtp-port-max" value="50000"/>
    <param name="speechsynth" value="speechsynthesizer"/>
    <param name="speechrecog" value="speechrecognizer"/>
    <param name="codecs" value="PCMU PCMA L16/96/8000"/>
  </profile>
</include>
```

创建或修改`conf/autoload_configs/unimrcp.conf.xml`文件：

```xml
<configuration name="unimrcp.conf" description="UniMRCP Client">
  <settings>
    <!-- UniMRCP profile to use for TTS -->
    <!-- value对应aliyun-mrcpserver.xml中profile的name -->
    <param name="default-tts-profile" value="aliyun-mrcpserver"/>
    <!-- UniMRCP profile to use for ASR -->
    <!-- value对应aliyun-mrcpserver.xml中profile的name -->
    <param name="default-asr-profile" value="aliyun-mrcpserver"/>
    <!-- UniMRCP logging level to appear in freeswitch.log.  Options are:
         EMERGENCY|ALERT|CRITICAL|ERROR|WARNING|NOTICE|INFO|DEBUG -->
    <param name="log-level" value="DEBUG"/>
    <!-- Enable events for profile creation, open, and close -->
    <param name="enable-profile-events" value="false"/>

    <param name="max-connection-count" value="100"/>
    <param name="offer-new-connection" value="1"/>
    <param name="request-timeout" value="3000"/>
  </settings>

  <profiles>
    <X-PRE-PROCESS cmd="include" data="../mrcp_profiles/*.xml"/>
  </profiles>
</configuration>
```

在`grammar`目录下创建`alimrcp.gram`文件：

```js
#JSGF V1.0;
/** JSGF Grammar for example */
grammar example;
public <results> = [];
```

修改unimrcp配置后需要重启FreeSWITCH服务。

### 通过dialplan测试

在FreeSWITCH安装目录里创建`conf/dialplan/default/mrcp.xml`文件：

```xml
<include>
  <!-- 一个简单实现echo的脚本，用于测试lua模块是否好用 -->
  <extension name="to_asr">
    <condition field="destination_number" expression="^(001)$">
      <action application="detect_speech" data="unimrcp:aliyun-mrcpserver alimrcp default"/>
      <action application="echo"/>
    </condition>
  </extension>
</include>
```

在fs_cli中执行`reloadxml`，然后盯着控制台，用分机呼叫001，接通后说一句话，应该会在控制台看到识别结果，类似：

```xml
<?xml version="1.0" encoding="utf-8"?>
<result>
  <interpretation grammar="session:default" confidence="1">
    <instance>
      <result>中午吃什么？</result>
      <beginTime>730</beginTime>
      <endTime>2220</endTime>
      <taskId>511ba061b8a54d7b93e0f5675534febd</taskId>
      <waveformUri>5d27104393eb4dbe-1.wav</waveformUri>
    </instance>
    <input mode="speech">中午吃什么？</input>
  </interpretation>
</result>
```

恭喜，到这里，你的FreeSWITCH已经可以“听”和“说”了。
