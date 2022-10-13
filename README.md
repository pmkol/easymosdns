# EasyMosdns

简化Mosdns基本功能使用的辅助脚本，仅需几分钟即可搭建一台支持ECS的无污染DNS服务器。<br />

- 无需重新编译，适配原生的 Mosdns 4.1+
- 内置中国大陆地区的优化规则，满足DNS日常使用场景，开箱即用
- 常用功能通过脚本控制，大幅度降低使用门槛
- 使用shell语言编写，对Linux系统具有较好的兼容性，CentOS/RedHat 7+已通过测试
<br />

## About Mosdns

[Mosdns](https://github.com/IrineSistiana/mosdns) 是一个插件化的DNS转发/分流器。用户可以按需拼接插件，搭建适合自己的DNS服务器。<br />

配置灵活的优点可以满足各种场景下的DNS使用，相对也会有较高的使用门槛，无法开箱即用。<br />
<br />

## About EasyMosdns

[EasyMosdns](https://apad.pro/easymosdns) 是基于Mosdns制作的EDNS方案，通过脚本降低使用门槛，内置中国大陆规则，开箱即用。<br />

#### 转发规则：

- 本地上游使用 DNSPod | AliDNS
- 远程上游使用 GoogleDNS
- 远程上游备用 EasyMosDNS (自建DoH) | TUNA DNS
- 支持使用socks5代理连接 远程上游
- 支持切换本地/远程上游转发的DNS

#### 功能说明：

- 支持EDNS解析，根据域名与中国大陆IP列表智能分流，查询结果无污染
- 污染列表与自定义列表中的域名，请求上游DNS时自动替换附带的用户IP子网信息，保护隐私
- 强化Hosts功能，域名支持多个IP，支持IPv6
- 支持自定义ECS，强制域名附带中国大陆/台湾地区的ECS解析
- DNS缓存时间优化，自动更新缓存，支持Redis持久化存储，可根据场景切换缓存规则
- 轻度过滤恶意广告，可通过白名单自定义过滤规则
- 屏蔽TYPE65与非中国大陆地区的IPv6请求，自动保留纯IPv6域名的请求，以获取更好的网络体验
- 支持规则自动更新，提供直连/CDN/Socks5三种下载更新规则的方式
- 支持上游节点故障时自动转移，优化DNS服务的稳定性

#### 更多信息：

查看搭建教程/测试DNS效果 请前往
https://apad.pro/easymosdns/ <br />
<br />

## Installation

#### 请先确认：

- Mosdns已安装且可以正常运行
- Mosdns版本 4.1+
- Mosdns工作目录是否为/etc/mosdns
- config配置文件已备份
- config配置文件中'protocol: udp'与'protocol: tcp'字段下方的'addr'使用的端口号

#### 部署示例：
>- 案例操作系统为CentOS 7，仅供参考，不同Linux发行版本命令可能略有差异 <br />
>- 脚本工作目录为 /etc/mosdns 不建议更改，如需更改请自行修改源码 <br />
>- 如 mosdns 二进制文件未在 /usr/local/bin 或 /usr/bin 目录下，应在二进制文件所在目录下执行命令 <br />
>- 脚本默认启动 UDP/TCP 53端口运行mosdns，如需更改端口，请修改config配置文件中的'protocol' <br />
>- 如 mosdns 部署在内网，应关闭ECS模式，请参考文档的 [切换ECS模式](https://github.com/pmkol/easymosdns#切换ecs模式) <br />
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

- 强制域名附带中国大陆ECS解析

请添加域名至 /etc/mosdns/ecs_cn_domain.txt
> 仅对来自中国大陆地区的访问IP生效

- 强制域名附带中国台湾ECS解析

请添加域名至 /etc/mosdns/ecs_tw_domain.txt
> 部分在中国大陆没有CDN节点的域名，会向中国大陆的访问者分配距离较远的CDN节点 <br />
> 将域名添加至该名单后，当域名有距离中国较近的CDN节点时会自动分配，优化访问速度，例如 github.com
<br />

## Using EasyMosdns

- 如果只想快速搭建一台无污染的DNS服务器，仅需阅读后续文档中的自动更新规则部分
- 内置默认规则，根据中国大陆地区网络环境优化，基本满足DNS日常使用场景，开箱即用
- 附带管理脚本，帮助用户快速更改mosdns配置，无需学习编写配置文件，大幅度降低使用门槛

#### 脚本初始化
首次使用管理脚本应执行命令
```bash
chmod +x /etc/mosdns/tools/config-reset
/etc/mosdns/tools/config-reset
```
>使用脚本初始化功能，会重置config配置参数，并将其余脚本赋予执行权限<br />
>后续只需执行下列脚本，即可实现相应功能

#### 切换本地上游DNS

- AliDNS
```bash
/etc/mosdns/local/alidns
```

- DNSPod (DoT)
```bash
/etc/mosdns/local/dnspod
```

>修改本地上游DNS，只会影响在本地名单里的域名解析<br />
>当运营商存在DNS劫持时，本地上游DNS应切换为DNSPod (DoT)<br />
>使用IP判断的域名，本地域名自动使用AliDNS+DNSPod同时查询的方式解析

#### 切换远程上游DNS

- GoogleDNS (默认)
```bash
/etc/mosdns/remote/googledns
```

- 通过Socks5代理连接的 GoogleDNS 
```bash
/etc/mosdns/remote/googledns-socks
```

- EasyMosDNS (自建DoH)
```bash
/etc/mosdns/remote/easymosdns
```

- TUNADNS
```bash
/etc/mosdns/remote/tunadns
```

- 通过Socks5代理连接的远程主机DNS <br />
```bash
/etc/mosdns/remote/host-socks
```
需在提供Sokcs5服务的远程主机上部署53端口的DNS <br />
> Socks5地址默认连接本机1080端口 <br />
> 如需更改Socks5地址，请参考文档的 [配置Socks5代理](https://github.com/pmkol/easymosdns#配置socks5代理)

#### 切换缓存策略

- 关闭缓存
```bash
/etc/mosdns/cache/non-cache
```

- 开启内存缓存，关闭自动更新，缓存条数1024
```bash
/etc/mosdns/cache/tiny-cache
```

- 开启内存缓存，开启自动更新，缓存条数10240
```bash
/etc/mosdns/cache/small-cache
```

- 开启Redis持久化缓存，开启自动更新，缓存条数102400
```bash
/etc/mosdns/cache/redis-large-cache
```

- 开启Redis持久化缓存，开启自动更新，缓存条数10240000
```bash
/etc/mosdns/cache/redis-huge-cache
```

>开启Redis需部署Redis服务，默认连接本机6379端口 <br />
>如需修改Redis地址，请修改 /etc/mosdns/include.conf 后，重新切换缓存策略


#### 手动/自动更新规则

- 直连下载更新规则
```bash
/etc/mosdns/rules/update
```

- 通过CDN下载更新规则
```bash
/etc/mosdns/rules/update-cdn
```

- 通过Socks5下载更新规则
```bash
/etc/mosdns/rules/update-socks
```
> Socks5地址默认连接本机1080端口 <br />
> 如需更改Socks5地址，请参考文档的 [配置Socks5代理](https://github.com/pmkol/easymosdns#配置socks5代理)

- 自动更新规则

使用 crontab -e 命令在最后一行添加对应规则即可
> 每日5点整通过CDN下载更新规则的示例
> ```bash
> 0 5 * * * /etc/mosdns/rules/update-cdn
> ```

#### 配置Socks5代理

- 更改Socks5代理地址

请修改 /etc/mosdns/include.conf 后，运行 socks5-reload 脚本
```bash
/etc/mosdns/tools/socks5-reload
```
> Socks5默认地址 127.0.0.1:1080

#### 切换ECS模式

- 自动ECS

自动将用户请求的来源地址作为ECS附加到请求
```bash
/etc/mosdns/tools/ecs-on
```

- 关闭ECS

部署在内网的DNS服务器应关闭ECS
```bash
/etc/mosdns/tools/ecs-off
```

#### 开启/关闭 恶意广告过滤功能

- 开启
```bash
/etc/mosdns/tools/adblock-on
```

- 关闭
```bash
/etc/mosdns/tools/adblock-off
```

- 白名单

请添加域名至 /etc/mosdns/ecs_tw_domain.txt

#### 开启/关闭 IPv6屏蔽功能

- 开启
```bash
/etc/mosdns/tools/ipv4-global-on
```
> 仅对非中国大陆地区拥有IPv4的域名生效

- 关闭
```bash
/etc/mosdns/tools/ipv4-global-off
```

#### 如何开启DoH

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

#### 如何开启DoT

修改 /etc/mosdns/config.yaml 配置文件 <br />
去除以下注释，指定SSL证书位置
```bash
      - protocol: tls             
        addr: "0.0.0.0:853"
        cert: "/etc/mosdns/yourdomain.cert"
        key: "/etc/mosdns/yourdomain.key"
        idle_timeout: 10  
```

#### 如何修改UDP/TCP端口号

修改 /etc/mosdns/config.yaml 配置文件 <br />
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
- 配置文件相关问题，请查阅 [Mosdns-Wiki](https://irine-sistiana.gitbook.io/mosdns-wiki/)

#### 如何支持作者
[前往打赏](https://afdian.net/a/maplecool)
<br />

## Contact Us

For feedback, questions, and to follow the progress of the project: <br />
[Telegram Group](https://t.me/+VeV5wt1E6FA5Ue-x)<br />
