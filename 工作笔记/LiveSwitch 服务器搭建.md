# LiveSwitch 服务器搭建

## 1. 概述

### 1.1. 服务组件图

![LiveSwitch 组件图](https://raw.githubusercontent.com/jugggao/image-hosting/main/notes/%E5%B7%A5%E4%BD%9C%E7%AC%94%E8%AE%B0/liveswitch%20%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%90%AD%E5%BB%BA.md/342813817240143.png)

### 1.2. 服务组件说明

LiveSwitch Server 由三个服务器组件组成：

- **The Gateway**：网关 - 提供通信功能，可以使客户端链接到媒体服务器。
- **The Media Server**：媒体服务器 - 提供 WebRTC 链接，可以使客户端通过 SFU 或 MCU 链接流媒体服务器进行流式传输。
- **The SIP Connector**：SIP 连接器 - 提供与第三方 SIP 服务的互相操作。

| 组件              | 用途                                                                  | 要求                                                                      |
| ----------------- | --------------------------------------------------------------------- | ------------------------------------------------------------------------- |
| Redis             | 存储临时数据                                                          | 3.2.100 版本以上，必须关闭集群模式。目前 LiveSwitch 不支持 Redis 集群模式 |
| PostgreSQL        | 存储配置数据和记录信息                                                | 11 版本以上                                                               |
| RabbitMQ          | 在 Recording Monitor、Recording Mover 和 Recording Muxer 之间传递消息 | 3.8.11 版本以上                                                           |
| Gateway           | 网关服务，连接 LiveSwitch 各个组件                                    | 最新版本                                                                  |
| Media Server      | 媒体服务器，管理 SFU 和 MCU 会话                                      | 最新版本                                                                  |
| SIP Connector     | SIP 连接器，可以使 LiveSwitch 连接到 SIP 端点                         | 可选                                                                      |
| Recording Monitor | 管理记录数据库并运行迁移                                              | 必须开启 Recording Management 功能                                        |
| Recording Mover   | 将录制文件转移至云端                                                  | 可选                                                                      |
| Recording Muxer   | 支持多路录制                                                          | 可选                                                                      |

### 1.3. 服务器环境要求

**开发环境要求**

必须为客户端设置至少一个网关和一个媒体服务器以获取 SFU 或 MCU 连接。

**生产环境要求**

建议：

- 至少两个网关做负载均衡以实现冗余
- 一个媒体服务器集群，根据并发流量峰值来配置集群大小。
- 如果需要进行 SIP 操作和通信则需要 SIP 连接器。

### 1.4. 服务器配置要求

#### 1.4.1. 网关服务器

**防火墙**

- 允许入方向 HTTP 8080 端口（默认端口号）
- 允许入方向 HTTPS 8443 端口（默认端口号）

**硬件**

推荐：

- 4 x CPU
- 4 GB RAM
- 高性能网卡

**高可用性**

- 2+ 网关
- 保证高可用性的负载均衡器

#### 1.4.2. 媒体服务器

**防火墙**

- 对于集群，允许入方向 TCP 8445 端口。如果 8445 端口不可用，媒体服务器会逐步尝试下一个端口，直到有一个端口可用，比如 8446、8447、8448、8449 等。
- 对于客户端链接，入方向 UDP 49152-65535 端口号。

**硬件**

推荐：

- 8 x CPU
- 8 GB RAM
- 高性能网卡

**高可用性**

- 2+ 媒体服务器

#### 1.4.3. SIP 连接器

**防火墙**

对于 SIP 请求，允许入方向 TCP 或 UDP 5061 端口号。

### 1.5. SFU 连接性能估算

对于 720P 的音视频连接，LiveSwitch 每个虚拟 CPU 可以处理约 50 个连接。

因此服务器数量 ~= SFU 连接数 / 服务器 CPU 数 / 50

### 1.6. MCU 连接性能估算

每个 CPU 处理 4 个 MCU 连接。

## 2. 使用 Docker 部署单机版本

### 2.1. 使用 Docker 部署 LiveSwitch Gateway

1. 创建 Docker 网络：

    ```bash
    docker network create liveswitch
    ```

2. 启动 Redis 服务，LiveSwitch 使用 Redis 作为默认存储引擎：

    ```bash
    docker run -d --network liveswitch --publish 6379:6379 --restart always --name redis redis
    ```

3. 启动配置 Postgres 服务：

    ```bash
    sudo docker run -d --env POSTGRES_PASSWORD='Ambow99999999' --network liveswitch --publish 5432:5432 --restart always --name postgres postgres
    ```

4. （可选）如果你计划部署 Recording Management、Recording Muxer 和 Recording Mover 服务，则需要启动 RabbitMQ 并配置 Recording Postgres 数据库：

    ```bash
    docker run -d --network liveswitch --publish 5672:5672 --publish 15672:15672 --restart always --name rabbitmq rabbitmq:3-management

    docker exec -it postgres psql "postgresql://postgres:Ambow99999999@localhost" -c "CREATE DATABASE recording;"

    docker exec -it postgres psql "postgresql://postgres:Ambow99999999@localhost" -c "GRANT ALL PRIVILEGES ON DATABASE recording TO postgres;"
    ```

    本次试验开启所有功能。

5. 启动 LiveSwitch 网关服务：

    - 如果你没有部署 Recording Management，执行以下命令：

        ```bash
        docker run -d \
            --env CONNECTIONSTRINGS:DEFAULT='postgres://postgres:Ambow99999999@postgres:5432/postgres' \
            --env CONNECTIONSTRINGS:CACHE='redis://redis' \
            --network liveswitch --publish 8080:8080 --publish 9090:9090 --publish 8443:8443 --publish 9443:9443 \
            --restart always --name liveswitch-gateway frozenmountain/liveswitch-gateway
        ```

    - 如果你部署了 Recording Management，则执行以下命令：

        ```bash
        docker run -d \
            --env CONNECTIONSTRINGS:DEFAULT='postgres://postgres:Ambow99999999@postgres:5432/postgres' \
            --env CONNECTIONSTRINGS:CACHE='redis://redis' \
            --env Deployments:0:DeploymentId='Default' \
            --env Deployments:0:RecordingManagement:Enabled='true' \
            --env Deployments:0:RecordingManagement:AmqpUri='amqp://guest:guest@rabbitmq:5672' \
            --env Deployments:0:RecordingManagement:PostgresUri='postgres://postgres:Ambow99999999@postgres:5432/recording' \
            --network liveswitch --publish 8080:8080 --publish 9090:9090 --publish 8443:8443 --publish 9443:9443 \
            --restart always \
            --name liveswitch-gateway \
            frozenmountain/liveswitch-gateway
        ```

其他可配置环境变量说明：

| 环境变量                                 | 默认值    | 说明                                                                                                                                |
| ---------------------------------------- | --------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| CONNECTIONSTRINGS:DEFAULT                | 127.0.0.1 | 与 [ServiceStack.Redis](https://github.com/ServiceStack/ServiceStack.Redis#redis-connection-strings) 兼容的连接字符串               |
| TRUSTEDSERVERCERTIFICATESHA1FINGERPRINTS | -         | Redis 数据访问收信任证书 SHA-1 指纹列表（十六进制、无冒号、不区分大小写），当 Redis 流量加密并 Redis 服务器使用自签名证书时需要配置 |
| ADMIN:IPADDRESSES                        | 0.0.0.0   | 配置管理端端口（默认 9090）监听 IP 地址或 CIDR 范围。                                                                               |
| SYNC:IPADDRESSES                         | 0.0.0.0   | 配置通信端口（默认 8080）监听 IP 地址或 CIDR 范围。                                                                                 |

### 2.2. 使用 Docker 部署 LiveSwitch Media Server

```bash
docker run -d \
    --env CONNECTIONSTRINGS:DEFAULT='postgres://postgres:Ambow99999999@127.0.0.1:5432/postgres' \
    --env CONNECTIONSTRINGS:CACHE='redis://127.0.0.1' \
    --env SERVICEBASEURL='http://127.0.0.1:8080' \
    --env EXTERNAL:PUBLICIPADDRESS=1.119.165.244 \
    --env EXTERNAL:PUBLICHOSTNAME=dev-liveswitch.oook.cn \
    --network host \
    --restart always \
    --ulimit nofile=65535:65535 \
    --name liveswitch-media-server \
    frozenmountain/liveswitch-media-server
```

其他可配置环境变量说明：

| 环境变量                  | 默认值    | 说明                                                                                                                                |
| ------------------------- | --------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| CONNECTIONSTRINGS:DEFAULT | 127.0.0.1 | 与 [ServiceStack.Redis](https://github.com/ServiceStack/ServiceStack.Redis#redis-connection-strings) 兼容的连接字符串               |
| REGION                    | -         | LiveSwitch 网关在选择 LiveSwitch 媒体服务器来处理入站客户端流时使用的字符串值。指示区域的客户端将尽可能与具有匹配值的媒体服务器匹配 |
| INTERNAL:IPADDRESSES      | 0.0.0.0   | 内部监听的 IP 地址或 CIDR 范围                                                                                                      |
| EXTERNAL:IPADDRESSES      | 0.0.0.0   | 外部监听的 IP 地址或 CIDR 范围                                                                                                      |
| EXTERNAL:PUBLICIPADDRESS | - | 可从外部访问的当前机器的公共 IP 地址。如果设置，这将覆盖部署配置中外部 STUN 服务器提供的自动检测。 |
| EXTERNAL:PUBLICHOSTNAME   | -         | 可以从外部访问的当前服务器的 DNS 地址，如果开启了 TURNS 此配置项必须配置                                                            |



### 2.3. 使用 Docker 部署 LiveSwitch SIP Connector

```bash
docker run -d \
    --env CONNECTIONSTRINGS:DEFAULT='postgres://postgres:Ambow99999999@postgres:5432/postgres' \
    --env CONNECTIONSTRINGS:CACHE='redis://redis' \
    --env SERVICEBASEURL='http://liveswitch-gateway:8080' \
    --network liveswitch \
    --publish 5060:5060 \
    --restart always \
    --name liveswitch-sip-connector \
    frozenmountain/liveswitch-sip-connector
```

其他可配置环境变量说明：

| 环境变量                  | 默认值    | 说明                                                                                                                                |
| ------------------------- | --------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| CONNECTIONSTRINGS:DEFAULT | 127.0.0.1 | 与 [ServiceStack.Redis](https://github.com/ServiceStack/ServiceStack.Redis#redis-connection-strings) 兼容的连接字符串               |
| REGION                    | -         | LiveSwitch 网关在选择 LiveSwitch 媒体服务器来处理入站客户端流时使用的字符串值。指示区域的客户端将尽可能与具有匹配值的媒体服务器匹配 |

### 2.4. 使用 Docker 部署 LiveSwitch Recording Monitor

```bash
docker run -d \
    --env CONNECTIONSTRINGS:DEFAULT='postgres://postgres:Ambow99999999@postgres:5432/postgres' \
    --env CONNECTIONSTRINGS:CACHE='redis://redis' \
    --env SERVICEBASEURL='http://liveswitch-gateway:8080' \
    --network liveswitch \
    --restart always \
    --name liveswitch-recording-monitor \
    frozenmountain/liveswitch-recording-monitor
```

其他可配置环境变量说明：

| 环境变量                  | 默认值    | 说明                                                                                                                  |
| ------------------------- | --------- | --------------------------------------------------------------------------------------------------------------------- |
| CONNECTIONSTRINGS:DEFAULT | 127.0.0.1 | 与 [ServiceStack.Redis](https://github.com/ServiceStack/ServiceStack.Redis#redis-connection-strings) 兼容的连接字符串 |

### 2.5. 使用 Docker 部署 LiveSwitch Recording Muxer

```bash
docker run -d \
    --env CONNECTIONSTRINGS:DEFAULT='postgres://postgres:Ambow99999999@postgres:5432/postgres' \
    --env CONNECTIONSTRINGS:CACHE='redis://redis' \
    --env SERVICEBASEURL='http://liveswitch-gateway:8080' \
    --network liveswitch \
    --restart always \
    --name liveswitch-recording-muxer \
    frozenmountain/liveswitch-recording-muxer
```

其他可配置环境变量说明：

| 环境变量                  | 默认值    | 说明                                                                                                                  |
| ------------------------- | --------- | --------------------------------------------------------------------------------------------------------------------- |
| CONNECTIONSTRINGS:DEFAULT | 127.0.0.1 | 与 [ServiceStack.Redis](https://github.com/ServiceStack/ServiceStack.Redis#redis-connection-strings) 兼容的连接字符串 |

### 2.6. 使用 Docker 部署 LiveSwitch Recording Mover

```bash
docker run -d \
    --env CONNECTIONSTRINGS:DEFAULT='postgres://postgres:Ambow99999999@postgres:5432/postgres' \
    --env CONNECTIONSTRINGS:CACHE='redis://redis' \
    --env SERVICEBASEURL='http://liveswitch-gateway:8080' \
    --network liveswitch \
    --restart always \
    --name liveswitch-recording-mover \
    frozenmountain/liveswitch-recording-mover
```

其他可配置环境变量说明：

| 环境变量                  | 默认值    | 说明                                                                                                                  |
| ------------------------- | --------- | --------------------------------------------------------------------------------------------------------------------- |
| CONNECTIONSTRINGS:DEFAULT | 127.0.0.1 | 与 [ServiceStack.Redis](https://github.com/ServiceStack/ServiceStack.Redis#redis-connection-strings) 兼容的连接字符串 |


## 3. 问题收集

### 3.1. 无法收到任何应答

**问题详细**

IOS 人员调试时通过 `dev-liveswitch.oook.cn` LiveSwitch 网关无法连接，报错日志如下：

```bash
2022-04-30 11:28:01.073996+0800 SwitchLiveDemo[599:131495] [connection] nw_resolver_start_query_timer_block_invoke [C1] Query fired: did not receive all answers in time for dev-liveswitch.oook.cn:443
2022-04-30 11:28:01.941865+0800 SwitchLiveDemo[599:131532] [connection] nw_resolver_start_query_timer_block_invoke [C2] Query fired: did not receive all answers in time for dev-liveswitch.oook.cn:443
2022-04-30 11:28:01.989164+0800 SwitchLiveDemo[599:131511] [LiveSwitch][599][10796688896] WARN    [FMLiveSwitchIceDatagramSocketManager][-] 2022-04-30T03:28:01.988Z Allocate request from 192.168.1.10:55148 to 1.202.193.88:3478 timed out.
```

**问题排查**

- 首先，从目的套接字 `1.202.193.88:3478` 中的 `3478` 这个端口我并没有开放。而 `3478` 又是 LiveSwitch Media Server 的端口号。因此可以判断，客户端也需要直接连接 LiveSwitch Media Server 的 3348 端口号；
- 其次，`1.202.193.88` 这个公网 IP 地址和我们无关，不知道怎么生成的这个公网 IP 地址。

所以我们要开放这个端口号，并尝试修复这个公网地址。

从管理后台查到这个  IP 地址：

![Media Servers PublicIPAddress 信息](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220430161600.png)

的确是错误的。

**问题解决**

1. 解决公网 IP 地址错误

    部署 LiveSwitch Media Server 时，我没有指定 `EXTERNAL:PUBLICIPADDRESS` 和 `EXTERNAL:PUBLICHOSTNAME`（环境变量含义参考上文）。因此 LiveSwitch Media Server 会采用自动发现来自动配置公网 IP，很明显这个自动发现的是错误的（目前还不知道为什么自动发现错误），因此我们需要手动配置这两个环境变量去指定公网 IP 和公网域名，重新部署 LiveSwitch Media Server：

    ```bash
    docker run -d \
        --env CONNECTIONSTRINGS:DEFAULT='postgres://postgres:Ambow99999999@127.0.0.1:5432/postgres' \
        --env CONNECTIONSTRINGS:CACHE='redis://127.0.0.1' \
        --env SERVICEBASEURL='http://127.0.0.1:8080' \
        --env EXTERNAL:PUBLICIPADDRESS=1.119.165.244 \
        --env EXTERNAL:PUBLICHOSTNAME=dev-liveswitch.oook.cn \
        --network host \
        --restart always \
        --ulimit nofile=65535:65535 \
        --name liveswitch-media-server \
        frozenmountain/liveswitch-media-server
    ```

2. 解决端口号不通的问题

    Media Server 主动监听 UDP/TCP 的 3478 端口号：

    ```
    $ netstat -anputl |grep 3478
    tcp        0      0 172.18.0.1:3478         0.0.0.0:*               LISTEN      520240/dotnet       
    tcp        0      0 10.10.113.23:3478       0.0.0.0:*               LISTEN      520240/dotnet       
    udp        0      0 172.18.0.1:3478         0.0.0.0:*                           520240/dotnet       
    udp        0      0 10.10.113.23:3478       0.0.0.0:*                           520240/dotnet
    ```

    开放此端口号后，问题解决，客户端成功可以建立连接，连接如下图所示。

    ![3478 端口号连接](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/企业微信截图_16513046849517.png)

现在观察后台的信息，发现 IP 地址已经显示成我映射的公网 IP 了。

![LiveSwitch Media Server 正确公网信息](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220430163250.png)

## 4. 参考

- https://developer.liveswitch.io/liveswitch-server/server/server.html
- https://developer.liveswitch.io/liveswitch-server/server/install/install-using-docker.html