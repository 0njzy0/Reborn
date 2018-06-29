# 配置详解

## 简单说明一下

配置具体可以分为4块内容：

* GENERAL
* PROXY
* ROUTER
* DNS

每块区域使用 yaml 格式，注意key值后的`:`后面必须有一个空格。
当前解析代码比较暴力，会把密码中的`#`视作注释，后续会修复。

关于系统的网络配置问题。rebron 启动的时候会创建一个独立网络服务，之后的DNS、系统代理会配置在该服务上，不会影响到其他的服务。

### GENERAL

后续会开放配置，现在不可以用，不要填这块区域

### PROXY

代理服务器配置

注：socks5 协议还不支持用户名密码

### ROUTER

路由规则

### DNS

DNS 设置


## 具体模版

```
[GENERAL]
# IP 归属地识别
#   type 为 ip 数据库类型, 可以设置成以下值 geolite, ip2location, ipip
#   path 为 ip 数据库路径, 且有如下规则
#     1. 当 path 为文件路径时即为数据库文件路径
#     2. 当 path 为文件目录时, 按照一下文件名匹配数据库文件
#          ipip         默认文件名为 17monipdb.dat (https://www.ipip.net/)
#          geolite      默认文件名为 GeoLite2-Country.mmdb (https://www.maxmind.com)
#          ip2location  默认文名件为 
#     3. 当 path 为空时从下列目录中查找数据库文件, 文件名见规则2
#          "~/Library/Application Support/Reborn/GEOIP/"
#          "应用程序路径/Contents/Resources/"
geoip:
  type: geolite
  path: ~

# 日志级别, 包含以下值: error, warning, info, debug, verbose
loglevel: info

# 系统代理设置
#   http   设置系统 http 和 https 代理, 可设值 enable, disable
#   socks5 设置系统 socks5 代理, 可设值 enable, disable
#   whitelist  白名单, 匹配到白名单中的域名不走系统代理
#   PAC 提供 PAC 支持
system-proxy:
  http: enable
  socks5: disable
  whitelist: localhost
  # PAC: 

[PROXY]
# 代理服务器设置
#  name      代理服务器别名
#  server    服务器地址
#  port      端口
#  timeout   连接超时时间, 单位毫秒
#  type      协议类型, 当前支持 shadowsocks, socks5, 后续将支持
#            https, http2, v2ray
#            http, http-tls(HTTP Over TLS)
#            socks5, socks5-tls(SOCKS5 Over TLS)
#  
#  协议特有字段
#    socks5 包含两个可选字段, 必须同时出现或不出现
#      username  用户名
#      password  密码
#
#    shadowsocks 必须包含以下两个字段
#      method 加密方式支持以下值
#        aes-192-ctr
#        aes-128-ctr
#        aes-256-ctr
#        aes-128-cfb
#        aes-192-cfb
#        aes-256-cfb
#        camellia-128-cfb
#        camellia-192-cfb
#        camellia-256-cfb
#        bf-cfb
#        rc4-md5
#        salsa20
#        chacha20
#        chacha20-ietf
#        aes-192-gcm
#        aes-128-gcm
#        aes-256-gcm
#        chacha20-ietf-poly1305
#      password 密码
#      udp_relay 是否开启 udp 转发，默认关闭，使用 enable 或者 disable

- name: ss
  server: yourserver.com
  port: 8388
  timeout: 600
  type: shadowsocks
  method: chacha20-ietf
  password: password
  udp_relay: enable

[ROUTER]
# 路由规则
# 规则组包含以下几部分
#   reject(阻止连接)
#   direct(直接连接)
#   所有代理服务器的别名
# 规则组中内容为目标 IP 地址或域名, 书写格式如下
#   IP 匹配
#     IPv4 地址:    192.168.1.1
#     IPv4 地址段:  192.168.1.1/16
#     IPv6 地址:    [::192.168.1.1], [2001:0002::1]
#     IPv6 地址段:  [::192.168.1.1/96], [2001:0002::/48]
#   域名匹配:
#     qq.com 会匹配改域名下所有子域名如 www.qq.com、vip.qq.com 等

# 直连规则组
direct:
  - jd.com
  - 114.114.114.114, 119.29.29.29
# 阻断规则组
reject:
  - ads.mopub.com, cpro.baidu.com
  - ad.api.3g.youku.com
# 自定义代理规则组
ss:
  - facebook.com, twitter.com
# IP 归属地对应的规则组
# 具体 code 见：https://en.wikipedia.org/wiki/ISO_3166-1#Current_codes
geoip:
  cn: direct
  hk: direct
  jp: reject
# 进程对应的规则组, 进程名称作为 key, 对应的规则作为 value. 进程名大小写不敏感
process:
  ss-local: direct
  thunder: direct
# 当所有规则都无法匹配时将使用 default 中的规则组进行路由
default: ss


# DNS 配置
[DNS]
# server  默认 DNS 服务器, 格式如下
#   system 系统 DNS 服务器
#   114.114.114.114  服务器地址, 端口使用默认值 53
#   114.114.114.114:5353 服务器地址:端口地址
# 可以设置多组如: 
# server: system, 114.114.114.114:53, 119.29.29.29:53
server: system

# 本地 hosts 映射
hosts:
  test: 127.0.0.1

# DNS 防污染服务器设置
#   enable    是否启用
#   server    DNS 服务器地址:端口号
#   protocol  协议类型, 当前仅支持 tcp, 后续将支持 udp, opendns, dnssec, dnscrypt, dns-https(DNS over HTTPS)
#   proxy     使用代代理服务器别
#  
#   当查询的域名或查询结果 ip 匹配到如下规则时，跳过 smartdns 查询:
#     skip_geoip 
#     skip_ip
#     skip_domain
#   当查询的域名或查询结果 ip 匹配到如下规则时，强制进行 smartdns 查询:
#     force_ip
#     force_domain
smartdns:
  enable: true
  server: 8.8.8.8:53
  # 改字段当前仅支持 tcp 被忽略
  # protocol: tcp
  skip_geoip: cn, hk
  skip_ip: 10.0.0.0/8
  skip_domain: baidu.com, qq.com
  # 
  force_ip:
    # DNS cache pollution
    # https://zh.wikipedia.org/wiki/%E5%9F%9F%E5%90%8D%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%93%E5%AD%98%E6%B1%A1%E6%9F%93
    - 4.36.66.178, 8.7.198.45, 23.89.5.60, 31.13.74.40, 37.61.54.158, 42.123.125.237, 46.82.174.68, 49.2.123.56, 54.76.135.1, 59.24.3.173, 60.19.29.22, 61.131.208.210, 61.131.208.211, 64.33.88.161, 64.33.99.47, 64.66.163.251, 65.104.202.252, 65.160.219.113, 66.45.252.237, 72.14.205.99, 72.14.205.104, 74.125.127.113, 77.4.7.92, 78.16.49.15, 92.242.144.2, 93.46.8.89, 108.160.166.92, 110.249.209.42, 118.5.49.6, 120.192.83.163, 123.129.254.12, 123.129.254.13, 123.129.254.14, 123.129.254.15, 125.211.213.132, 128.121.126.139, 159.106.121.75, 169.132.13.103, 183.221.250.11, 185.85.13.155, 188.5.4.96, 189.163.17.5, 192.67.198.6, 197.4.4.12, 202.98.24.122, 202.98.24.124, 202.98.24.125, 202.106.1.2, 202.181.7.85, 203.98.7.65, 203.161.230.171, 207.12.88.98, 208.56.31.43, 209.36.73.33, 209.145.54.50, 209.220.30.174, 211.94.66.147, 211.98.70.195, 211.98.70.226, 211.98.70.227, 211.98.71.195, 211.138.34.204, 211.138.74.132, 213.169.251.35, 216.221.188.182, 216.234.179.13, 218.93.250.18, 220.165.8.172, 220.165.8.174, 220.250.64.20, 221.179.46.190, 243.185.187.39, 249.129.46.48, 253.157.14.165
  force_domain: facebook.com
  proxy: ss
```
