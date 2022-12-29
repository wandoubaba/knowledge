# docker部署rabbitmq，启用stomp，并安装rabbitmq_delayed_message_exchange插件

## 操作

### 下载延时队列插件

针对rabbitmq3.11.x

```bash
wget https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/releases/download/3.11.1/rabbitmq_delayed_message_exchange-3.11.1.ez
```

针对rabbitmq3.10.x

```bash
wget https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/releases/download/3.10.2/rabbitmq_delayed_message_exchange-3.10.2.ez
```

针对rabbitmq3.9.x

```bash
wget https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/releases/download/3.9.0/rabbitmq_delayed_message_exchange-3.9.0.ez
```

### 安装rabbitmq

> 以3.11为例

```bash
# 拉取镜像并启动容器
docker run -d -v /data/rabbitmq/mnesia:/var/lib/rabbitmq/mnesia --hostname rabbitmq --name rabbitmq rabbitmq:3.11-management
```

### 安装并启用插件

```bash
# 将延时队列插件拷贝至容器（以3.11为例）
docker cp rabbitmq_delayed_message_exchange-3.11.1.ez rabbitmq:/plugins
# 进入容器
docker exec -it rabbitmq /bin/bash
# 查看插件
rabbitmq-plugins list
# 列表里会出现rabbitmq_delayed_message_exchange和rabbitmq_web_stomp和rabbitmq_web_stomp_examples
# 启用延时队列插件
rabbitmq-plugins enable rabbitmq_delayed_message_exchange
# 开启stomp插件
rabbitmq-plugins enable rabbitmq_stomp rabbitmq_web_stomp rabbitmq_web_stomp_examples
# 要以再查看插件，确认启用的插件都生效了
rabbitmq-plugins list
# 退出容器
exit
# 将启用了插件的容器提交为新镜像
docker commit rabbitmq rabbitmq:stomp-delay
# 停止并删除原容器
docker rm -f rabbitmq
```

### 用新的命令重新启用新的容器

```bash
docker run -d -p 5672:5672 -p 15672:15672 -p 15674:15674 -p 61613:61613 -v /data/rabbitmq/mnesia:/var/lib/rabbitmq/mnesia --hostname rabbitmq --name rabbitmq rabbitmq:stomp-delay
```

### 检验

用浏览器打开`http://host:15672`：

- 能打开管理界面，说明management插件已经生效。
- 默认账号密码都是guest。
- 在Overview界面的Ports and contexts列表中可以看到stomp端口和web-stomp端口，说明stomp插件和web-stomp插件已经生效。
- 在Exchanges界面的Add a new exchange列表中，看到Type中出现x-delayed-message，说明delayed_message_exchange插件已经生效。

### 容器开机自启

```bash
docker update --restart=always rabbitmq
```
