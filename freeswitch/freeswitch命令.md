# freeswitch有用的命令

---

## 启动freeswitch

```bash
freeswitch -nc  # 后台启动
freeswitch -nc -nonat   # 后台启动且关闭自动nat（有独立公网IP时不需要开启nat功能，能明显提高启动速度）
```

## fs_cli中使用的命令

- **shutdown**

> 关闭freeswitch服务

- **reloadxml**

> 重载配置（如拨号计划）

- **reloadacl**

> 重载acl

- **sofia profile internal restart**

> 重启internal服务

- **sofia profile external restart**

> 重启external服务

- **sofia status profile internal**

> 查看服务端口等信息

结果示例：

```text
=================================================================================================
Name             internal
Domain Name      N/A
Auto-NAT         false
DBName           sofia_reg_internal
Pres Hosts       183.211.245.46,183.211.245.46
Dialplan         XML
Context          public
Challenge Realm  auto_from
RTP-IP           183.211.245.46
SIP-IP           183.211.245.46
URL              sip:mod_sofia@183.211.245.46:5060
BIND-URL         sip:mod_sofia@183.211.245.46:5060;transport=udp,tcp
WS-BIND-URL     sip:mod_sofia@183.211.245.46:5066;transport=ws
WSS-BIND-URL     sips:mod_sofia@183.211.245.46:7443;transport=wss
HOLD-MUSIC       local_stream://moh
OUTBOUND-PROXY   N/A
CODECS IN        OPUS,G722,PCMU,PCMA,VP8
CODECS OUT       OPUS,G722,PCMU,PCMA,VP8
TEL-EVENT        101
DTMF-MODE        rfc2833
CNG              13
SESSION-TO       0
MAX-DIALOG       0
NOMEDIA          false
LATE-NEG         true
PROXY-MEDIA      false
ZRTP-PASSTHRU    true
AGGRESSIVENAT    false
CALLS-IN         8
FAILED-CALLS-IN  0
CALLS-OUT        2
FAILED-CALLS-OUT 0
REGISTRATIONS    2
```

- **sofia status profile internal reg**

> 查看注册用户

结果示例：

```text
Registrations:
=================================================================================================
Call-ID:     G7ZfP264pLJawfNLKuFg1A..
User:        1002@183.211.245.46
Contact:     "" <sip:1002@39.152.207.190:43502;transport=TCP;rinstance=c8aef170677f30f6>
Agent:       Zoiper rv2.10.6.2
Status:      Registered(TCP)(unknown) EXP(2021-09-14 13:42:51) EXPSECS(359)
Ping-Status: Reachable
Ping-Time: 0.00
Host:        fstesting
IP:          39.152.207.190
Port:        43502
Auth-User:   1002
Auth-Realm:  183.211.245.46
MWI-Account: 1002@183.211.245.46

Call-ID:     JRlQZx3SoV0ElHunKRrQNg..
User:        1001@183.211.245.46
Contact:     "" <sip:1001@39.152.207.190:41043;transport=UDP;rinstance=eaf774985a9b3b03>
Agent:       Zoiper rv2.10.6.2
Status:      Registered(UDP)(unknown) EXP(2021-09-14 13:38:15) EXPSECS(83)
Ping-Status: Reachable
Ping-Time: 0.00
Host:        fstesting
IP:          39.152.207.190
Port:        41043
Auth-User:   1001
Auth-Realm:  183.211.245.46
MWI-Account: 1001@183.211.245.46

Total items returned: 2
=================================================================================================

```
