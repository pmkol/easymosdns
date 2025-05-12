# EasyMosdns v3.5

基于Mosdns的精准DNS分流策略，仅需几分钟即可搭建一台支持ECS的无污染DNS服务器。<br />

## About Mosdns

[Mosdns](https://github.com/IrineSistiana/mosdns) 是一个插件化的DNS转发/分流器。用户可以按需拼接插件，搭建适合自己的DNS服务器。<br />

配置灵活的优点可以满足各种场景下的DNS使用，相对也会有较高的使用门槛，无法开箱即用。<br />
<br />

## About EasyMosdns

[EasyMosdns](https://apad.pro/easymosdns) 是基于Mosdns制作的EDNS部署方案，内置中国大陆地区优化规则与分流API，满足DNS日常使用场景。<br />

#### 转发规则：

- 本地上游并发请求 DNSPod 与 AliDNS
- 远程上游优先请求 EasyMosDNS分流API(DoH)
- 远程上游超时请求 GoogleDNS 与 OpenDNS
- 支持使用socks5代理连接远程上游

#### 功能说明：

- 支持EDNS解析，根据域名与中国大陆IP列表智能分流，查询结果无污染
- 污染列表与自定义列表中的域名，请求上游DNS时自动替换附带的用户IP子网信息，保护隐私
- 强化Hosts功能，域名支持多个IP，支持IPv6
- 支持指定ECS，强制域名附带指定的ECS解析
- DNS缓存时间优化，自动更新缓存，支持Redis持久化存储，根据场景自动切换缓存规则
- 轻度过滤恶意广告，可通过白名单自定义过滤规则
- 屏蔽TYPE65与非中国大陆地区的IPv6请求，自动保留纯IPv6域名的请求，以获取更好的网络体验
- 支持规则自动更新，提供直连/CDN两种下载更新规则的方式
- 支持上游节点故障时自动转移，优化DNS服务的稳定性

```
注意：
内置的分流API(DoH)存在请求限制，请勿修改分流API规则或在本项目之外使用
如需在更多客户端体验分流效果，请通过打赏项目获取无请求限制的分流API
```

#### 打赏项目：

打赏项目的用户可参与测试支持DoT与DoH(http2/3)且更精准的分流API
[前往发电](https://afdian.com/a/maplecool) <br />

#### 更多信息：

查看搭建教程/测试DNS效果 请前往
https://apad.pro/easymosdns/ <br />
<br />

## Installation

#### 请先确认：

- Mosdns已安装且可以正常运行
- Mosdns版本 4.5.3
- Mosdns工作目录是否为 /etc/mosdns
- config配置文件已备份
- config配置文件中'protocol: udp'与'protocol: tcp'字段下方的`addr:`使用的端口号

#### 部署示例：
>- 案例操作系统为Almalinux，仅供参考，不同Linux发行版本命令可能略有差异 <br />
>- 脚本工作目录为 /etc/mosdns 不建议更改，如需更改请自行修改源码 <br />
>- 如 mosdns 二进制文件未在 /usr/local/bin 或 /usr/bin 目录下，应在二进制文件所在目录下执行命令 <br />
>- 脚本默认启动 UDP/TCP 53端口运行mosdns，如需更改端口，请修改config配置文件结尾处的`protocol:` <br />
---
- 卸载Mosdns服务
```bash
mosdns service stop
mosdns service uninstall
```
> 如果Mosdns的工作目录为 /etc/mosdns 无需卸载服务 <br />
> 安装完成后使用 mosdns service restart 命令重启mosdns服务即可

- 下载源码解压缩
```bash
wget https://mirror.apad.pro/dns/easymosdns.tar.gz
tar xzf easymosdns.tar.gz
mv /etc/mosdns /etc/mosdns.old
mv easymosdns /etc/mosdns
```

- 重新安装并启动Mosdns服务
```bash
mosdns service install -d /etc/mosdns -c config.yaml
mosdns service start
```
> 看到 service is running 证明部署成功 <br />
>
> 启动参数示例：
> ```bash
> /usr/local/bin/mosdns start --as-service -d /etc/mosdns -c config.yaml
> ```

- 开启防火墙端口
> 部署完成后，请开启防火墙的端口 <br />
> 默认情况下，应开启 UDP53 TCP53 TCP80 TCP443 TCP853 端口
<br />

#### 配置自定义规则：

- 自定义hosts

请添加规则至 /etc/mosdns/hosts.txt
> 格式与常规hosts规则不同，域名在前、IP在后，支持一行多个IP，支持IPv6 <br />
> hosts规则示例：
> ```bash
> dns.google 8.8.8.8 8.8.4.4 2001:4860:4860::8888
> ```

- 强制域名附带本地ECS解析

请添加域名至 /etc/mosdns/ecs_cn_domain.txt
> 域名规则有多个匹配方式:  <br />
> 以 domain: 开头，域匹配。e.g: domain:google.com 会匹配自身 `google.com`，以及其子域名 `www.google.com`, `maps.l.google.com` 等 <br />
> 以 full: 开头，完整匹配。e.g: full:google.com 只会匹配自身 <br />
> 以 keyword: 开头，关键字匹配。e.g: keyword:google.com 会匹配包含这个字段的域名，如 `google.com.hk`, `www.google.com.hk` <br />
> 以 regexp: 开头，正则匹配(Golang 标准)。e.g: regexp:.+\.google\.com$

- 强制域名附带指定ECS解析

请添加域名至 /etc/mosdns/ecs_noncn_domain.txt
> 域名规则匹配方式同上 <br />
> 部分在中国大陆没有CDN节点的域名，会向中国大陆的访问者分配距离较远的CDN节点 <br />
> 将域名添加至该名单后，当域名有距离中国较近的CDN节点时会自动分配，优化访问速度，例如 github.com
<br />

## Using EasyMosdns

- 如果只想快速搭建一台无污染的DNS服务器，仅需阅读后续文档中的自动更新规则部分
- 内置默认规则，根据中国大陆地区网络环境优化，基本满足DNS日常使用场景
- 该项目已将常用功能封装成插件形式，只需修改 /etc/mosdns/config.yaml 配置文件中的相应插件参数即可

#### 手动/自动更新规则

- 直连下载更新规则
```bash
/etc/mosdns/rules/update
```

- 通过CDN下载更新规则
```bash
/etc/mosdns/rules/update-cdn
```

- 自动更新规则

使用 crontab -e 命令在最后一行添加对应规则即可
> 每日5点整通过CDN下载更新规则的示例
> ```bash
> 0 5 * * * /etc/mosdns/rules/update-cdn
> ```

#### 切换本地上游DNS

- 转发至本地服务器的插件
```bash
  - tag: forward_local
    type: fast_forward
    args:
      upstream:
        - addr: "223.5.5.5"
        - addr: "119.29.29.29"
```
> 修改 `addr:` 后的DNS地址即可 <br />
> 默认使用AliDNS+DNSPod同时查询的方式解析 <br />
> 运营商DNS不支持ECS，仅建议网络环境为内网时使用

#### 切换远程上游DNS

- 转发至远程服务器的插件
```bash
  - tag: forward_remote
    type: fast_forward
    args:
      upstream:
        - addr: "tcp://208.67.220.220:5353"
          enable_pipeline: true
          #socks5: "127.0.0.1:1080"
        - addr: "udpme://8.8.8.8"
```
> 修改 `addr:` 后的DNS地址即可 <br />
> 如果不确定上游DNS是否支持pipeline，请将 `enable_pipeline:` 设置为false <br />
> 默认使用GoogleDNS+OpenDNS同时查询的方式解析，如遇到连接问题建议配置socks5代理 <br />
> OpenDNS不支持指定ECS，如需配置指定ECS，应开启OpenDNS的socks5代理

#### 切换分流API/DNS

- 转发至分流服务器的插件
```bash
  - tag: forward_easymosdns
    type: fast_forward
    args:
      upstream:
        - addr: "https://mosdns.apad.pro/api-query"
          bootstrap: "223.6.6.6"
          #dial_addr: "ip:port"
```
> 修改 `addr:` 后的DNS地址即可 <br />
> 可以使用 `dial_addr:` 指定分流服务器的IP与端口，优化访问速度 <br />
> 默认使用EasyMosdns项目官方的分流API，如需切换应确定所选的API或DNS服务支持分流功能

#### 开启Redis持久化缓存

- 缓存的插件
```bash
  - tag: cache_lan
    type: cache
    args:
      size: 8192
      #redis: "redis://127.0.0.1:6379/0"
      lazy_cache_ttl: 86400
      cache_everything: true
      lazy_cache_reply_ttl: 1
  - tag: cache_wan
    type: cache
    args:
      size: 131072
      compress_resp: true
      #redis: "redis://127.0.0.1:6379/0"
      lazy_cache_ttl: 86400
      cache_everything: true
      lazy_cache_reply_ttl: 5
```
> 将两处 `#redis: "redis://127.0.0.1:6379/0"` 更改为 `redis: "redis://127.0.0.1:6379/0"` <br />
> 需部署Redis服务，默认连接本机6379端口

#### 配置指定ECS

- 指定ECS的插件
```bash
  - tag: ecs_global
    type: ecs
    args:
      auto: false
      ipv4: "168.95.1.0"
      ipv6: "2001:b000:168::"
      force_overwrite: true
```
> 修改 `ipv4:` 与 `ipv6:` 后的IP段即可 <br />
> 默认ECS参数已针对中国大陆地区优化，根据网络环境自动判断开关 <br />
> 如果将ECS设置为本地公网IP段，将关闭指定ECS功能 <br />
> 当代理客户端使用本地DNS解析时，应将ECS指定为距离代理IP出口最近的IP段，以获取最佳的访问速度

#### 关闭广告过滤功能

- 屏蔽广告域名
```bash
        - if: query_is_ad_domain
          exec:
            - black_hole
            - ttl_1h
            - _return
```
> 将 `black_hole` 更改为 `forward_local`

#### 关闭非本地IPv6屏蔽功能

- 优先返回ipv4结果
```bash
        - _prefer_ipv4
```
> 将三处 `- _prefer_ipv4` 更改为 `#- _prefer_ipv4`

#### 开启DoH监听

修改 /etc/mosdns/config.yaml 配置文件 <br />
去除以下注释
```bash
      - protocol: http
        addr: "127.0.0.1:9053"
        url_path: "/dns-query"
        get_user_ip_from_header: "X-Forwarded-For"
```
建议使用http模式，前端使用nginx反代 <br />
> nginx配置示例：
> ```bash
>   server {
>     ...
>     location /dns-query {
>	  set_real_ip_from 0.0.0.0/0;
>     real_ip_header X-Forwarded-For;
>     proxy_pass http://127.0.0.1:9053/dns-query;
>     }
>   }
> ```

#### 开启DoT监听

修改 /etc/mosdns/config.yaml 配置文件 <br />
去除以下注释，指定SSL证书位置
```bash
      - protocol: tls             
        addr: "0.0.0.0:853"
        cert: "/etc/mosdns/yourdomain.cert"
        key: "/etc/mosdns/yourdomain.key"
```

#### 修改UDP/TCP监听端口

示例：
```bash
      - protocol: udp
        addr: "0.0.0.0:8053"
      - protocol: tcp
        addr: "0.0.0.0:8053"
```

#### 更多帮助和支持

- 项目完整开源，可用于部署DNS服务器、开发软路由插件等多种场景
- 使用源码仅需标明使用 [EasyMosdns](https://apad.pro/easymosdns) 的源码即可
- 配置文件相关问题，请查阅 [Mosdns-Wiki](https://irine-sistiana.gitbook.io/mosdns-wiki/mosdns-v4)
<br />

## Contact Us

For feedback, questions, and to follow the progress of the project: <br />
[Telegram Group](https://t.me/+VeV5wt1E6FA5Ue-x)
