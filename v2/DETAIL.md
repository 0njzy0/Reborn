# 配置详解

## 简单说明一下

配置具体可以一下几块：

* 常规配置: GENERAL
* 代理服务器: PROXY
* 命名代理服务器: PROXY.proxyname
* 路由规则: RULES
* DNS 配置: DNS
* DNS 查询规则: DNS-RULES
* 规则组: GROUP.groupname
* 规则文件: FILE.filename


关于系统的网络配置问题。reborn 启动的时候会创建一个独立网络服务，之后的DNS、系统代理会配置在该服务上，不会影响到其他的服务。

### GENERAL

后续会开放配置，现在不需要填这块区域

### 代理服务器配置

头部定义

* 默认代理  

      [PROXY]

* 命名代理  
  `DIRECT`、`REJECT`、`PROXY` 为保留关键字，不允许作为名称。  
  比如定义一个名为 `us` 的代理服务器，可以按照如下格式：

      [PROXY.us]

必填项

* 类型

      type = shadowsocks
      type = socks5
* 代理服务器

      server = yourserver.com
      server = 133.29.14.53
* 端口号
      
      port = 8388

---

#### shadowsocks

* 加密算法

      method = aes-128-ctr
      method = aes-192-ctr
      method = aes-256-ctr
      method = aes-128-cfb
      method = aes-192-cfb
      method = aes-256-cfb
      method = camellia-128-cfb
      method = camellia-192-cfb
      method = camellia-256-cfb
      method = bf-cfb
      method = rc4-md5
      method = salsa20
      method = chacha20
      method = chacha20-ietf
      method = aes-192-gcm
      method = aes-128-gcm
      method = aes-256-gcm
      method = chacha20-ietf-poly1305

* 密码

      password = 12345678

* 是否开启 UDP 转发

      # 开启转发
      udp_relay = enable
      # 禁用转发
      udp_relay = disable

#### socks5

注：socks5 协议还不支持用户名密码



### 路由规则

适用于 `[RULES]` 和 `[DNS-RULES]`，一行表示一条规则，格式如下  

* [组策略], 规则类型, 规则内容, 路由策略
* FINAL, 路由策略. 当所有规则都不匹配时，由这条规则觉得使用哪种策略建立连接

#### 组策略 - 可选项

* GROUP
* FILE  暂不支持

#### 规则类型

* DOMAIN: 域名
* DOMAIN-SUFFIX: 域名后缀
* PROCESS: 进程名
* IP-CIDR: IP 地址/IP 地址段
* GEOIP: 区域名

#### 规则内容

不使用组策略时，该值为当前规则类型值

当使用组策略时，该值为组名, 即 `[GROUP.dns-pollution]` 中的 `dns-pollution`，组策略中的每一行都需要符合当前规则类型

#### 路由策略

* DIRECT
* REJECT
* PROXY
* 服务器名，具体就是指 `[PROXY.us]` 中的 `us`  

示例:

    DOMAIN, qq.com, DIRECT
    DOMAIN-SUFFIX, ad.google.com, REJECT
    IP-CIDR, 114.114.114.114, DIRECT
    IP-CIDR, 119.29.29.0/24, DIRECT

    GROUP, DOMAIN, china500, DIRECT
    GROUP, DOMAIN-SUFFIX, google-list, PROXY
    GROUP, IP-CIDR, chinaip, DIRECT

    GEOIP, CN, DIRECT
    FINAL, PROXY

### DNS 配置

* 本地 dns 服务器  
  `system` 表示使用系统默认服务器, 自定义服务器采用 `ip:port` 格式, 如果不写端口号则默认使用 53 端口。如果有多个 dns 服务器，使用 `,` 分割，具体示例如下：

      server = system
      server = 114.114.114.114
      server = 114.114.114.114:53
      server = system, 114.114.114.114
      server = system, 114.114.114.114:53
      server = 114.114.114.114:53, 119.29.29.29:53


* 远程 dns 服务器  
  使用 `ip:port` 格式，不支持多个服务器，不能缺省端口号

      remote = 8.8.8.8:53



## 示例模版

```
# 定义默认代理服务器
[PROXY]
type = shadowsocks
server = yourserver.com
port = 8388
method = rc4-md5
password = 12345678

# 定义一个局域网中的 socks5 代理，名为 local
[PROXY.local]
type = socks5
server = 192.168.1.100
port = 1080

# 定义一个名为 us 的 shadowsocks 代理
[PROXY.us]
type = shadowsocks
server = us.yourserver.com
port = 8388
method = rc4-md5
password = 12345678

[DNS]
# 使用系统默认的 dns 服务器
server = system
# 使用 google 的公共 dns 服务器避免污染
remote = 8.8.8.8:53

[RULES]
# 位于局域网段 192.168.1.1/24 的连接走代理服务器 local
IP-CIDR, 192.168.1.1/24, local
# 域名后缀匹配 tplink.com 时走代理服务器 local
DOMAIN-SUFFIX, tplink.com, local
# 域名后缀匹配 google.com 时走代理服务器 PROXY
DOMAIN-SUFFIX, google.com, PROXY

# 使用规则组 chinaip，该组内的 ip 走直连规则
GROUP, IP-CIDR, chinaip, DIRECT
# 使用规则组 china500，该组内的域名(域名全匹配)走直连规则
GROUP, DOMAIN, china500, DIRECT
# 使用规则组 ad, 该组内的域名(域名后缀匹配)拒绝连接
GROUP, DOMAIN-SUFFIX, ad, REJECT

# 进程名为 ss-local 的进程使用直接规则
PROCESS, ss-local, DIRECT

# ip 识别结果为中国的使用直连规则
GEOIP, CN, DIRECT
# ip 识别结果为美国的使用 us 代理服务器连接
GEOIP, US, us

# 所有规则都不匹配时，使用 PROXY 代理服务器连接
FINAL, PROXY

[DNS-RULES]
# 当本地 dns 服务器解析出来的 ip 在 dns-pollution 组内时
# 使用代理服务器重新解析域名
GROUP, IP-CIDR, dns-pollution, PROXY

# 当本地 dns 服务器解析出的 ip 为中国 ip 时，直接使用该结果，不在用远程 dns 服务器重新解析
GEOIP, CN, DIRECT
GEOIP, US, us
FINAL, PROXY

[GROUP.chinaip]
1.0.1.0/24
1.0.2.0/23
1.0.8.0/21
1.0.32.0/19

[GROUP.china500]
qq.com
baidu.com
360.cn

[GROUP.ad]
ad.google.com
ad.baidu.com

[GROUP.dns-pollution]
4.36.66.178
8.7.198.45
23.89.5.60
```