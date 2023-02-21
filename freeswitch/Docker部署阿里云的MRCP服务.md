## Docker部署阿里云的MRCP服务（版权）

> wandoubaba / 2023-02-21

在阿里云的的智能语音服务中，除了提供常规的ASR、TTS等能力外，也有完整的MRCP服务（阿里名称：SDM），不但可以对接私有化的智能语音能力，同时还可以对接公有云的ASR和TTS。

引用阿里云《SDM(MRCP-SERVER)公共镜像使用》文档中的一段话：

> SDM(MRCP-SERVER)目前在公共云上有对应的镜像仓库，用户可以直接拉取公共云镜像到本地，然后部署使用，一定程度上简化了部署和接入的成本。

### 环境依赖

机器需求：4C8G

操作系统：CentOS7.2及以上（其他支持Docker环境的较新版本的Linux发行版）

软件环境要求：Docker环境

### 安装过程

安装部署主要包括两个过程：到阿里云开通公共云语音服务、部署SDM镜像。

#### 开通阿里云智能语音服务

> 下面这些操作都是在阿里云的控制台里完成的。

- 注册阿里云账号，这里省略具体步骤。

- 登录阿里云控制台，到“智能语音交互服务”页面开通ASR、TTS相关服务

- 阿里云的语音服务都支持试用版免费使用，只不过并发路数有限，开发测试可以用。

- 到阿里云的“Access Key管理页面”创建并获取一组供SDM使用的具有调用智能语音服务能力的“Access Key ID”和“Access Key Secret”（注意妥善保存）。

- 到阿里云“智能语音交互”控制台->“全部项目”，如果没有项目创建一个项目，项目类型选择“语音识别+语音合成+语音分析“，主要是要拿到项目的“AppKey”。

- 注意“AppKey”对应项目的“项目功能配置”中，要选择8K模型。

经过以上操作，在云端的基础能力就已经配置好了，一共得到了3个变量：“Access Key ID”、“Access Key Secret”、“AppKey”，后面部署SDM时会用到这几个变量。

#### 部署SDM镜像

> 下面的操作是在要部署SDM的服务器上进行的，如果是以root登录的话，就自行忽略`sudo`，假设Linux系统上已经安装好了docker服务。

开始操作之前，最好事先在主机上为SDM规划一个目录，用于保存配置文件和日志等运行时文件，本文假设这个目录是`/data/sdm`，下面的操作都默认在`/data/sdm`目录下进行。

##### 初始化配置

```sh
# 进入主机准备的sdm文件目录
cd /data/sdm
# 拉取镜像
sudo docker pull registry.cn-shanghai.aliyuncs.com/nls-cloud/sdm:latest
# 确认本地已拉取的镜像
sudo docker images | grep sdm
# 初始化本地环境
sudo docker run -d --privileged --net=host --name nls-cloud-sdm -v `pwd`/logs:/home/admin/logs -v `pwd`/data:/home/admin/disk registry.cn-shanghai.aliyuncs.com/nls-cloud/sdm:latest standalone
```

到这里只是用到初始化环境，将容器中的SDM目录映射到宿主机上，由于相关配置参数还没有修改，所以容器最终不会运行起来（如果真运行起来了，也请用`docker stop`命令将它停了吧），下面需要进行相关配置操作。

##### 配置

首次启动后，容器内的配置文件在宿主机`/data/sdm/data/nls-cloud-sdm/conf`目录下，一般来说只需要修改其中的3个配置文件：nlstoken.json、service-asr.json、service-tts.json。

- nlstoken.json（配置AccessKeyId和AccessKeySecret）：

```json
{
    "AccessKeyId": "前面得到的AccessKeyId",
    "AccessKeySecret": "前面得到的AccessKeySecret",
    "DefaultToken": ""
}
```

说明：对接公有云时，DefaultToken的值要是空白。

- service-asr.json（主要配置url和AppKey）：

```json
{
    "url": "wss://nls-gateway.cn-shanghai.aliyuncs.com/ws/v1",
    "appkey": "前面得到的项目AppKey",
}
```

- service-tts.json（主要配置url和AppKey）：

```json
{
    "url": "wss://nls-gateway.cn-shanghai.aliyuncs.com/ws/v1",
    "appkey": "前面得到的项目AppKey",
}
```

其他asr和tts的配置一般不用修改，如果有特殊需求的话就按需配置好了。

##### 启动

```sh
sudo docker start nls-cloud-sdm
# 查看是否启动成功
sudo docker ps
# 查看服务进程是否已拉起
ps -ef | grep alimrcp-server
# 查看端口是否监听
sudo lsof -i:7010
# 查看日志中有没有ERROR信息
cat logs/nls-cloud-sdm/alimrcp-server.log | grep ERROR
# 设置容器自启动
sudo docker update --restart=always nls-cloud-sdm
```

##### 端口

默认情况下，SDM使用以下端口与客户端（如FreeSWITCH）通信，与服务端（阿里云）不需要开放端口，只需要能连接阿里云公有云服务即可。

- 7010（TCP&UDP）：SIP端口。
- 1544、1554（TCP）：MRCP协议端口。
- 10000-20000（UDP）：RTP协议端口，用来传输语音流。

##### 并发

SDM单机资源默认支持100路ASR+100路TTS，这个并发指的是SDM与客户端（如FreeSWITCH）的资源处理能力，与公有云的ASR、TTS并发能力无关。
