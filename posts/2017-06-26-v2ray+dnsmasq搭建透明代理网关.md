---
title: v2ray+dnsmasq搭建透明代理网关
date: 2017-06-26 17:18:32
---
# 1.网络环境介绍
`
Internet <--> 光猫 <--> Netgare WNDR4300 <-->Ubuntu Gateway
`
其他设备通过WNDR4300上网，有特殊需求的(代理翻墙)设置网关为Ubuntu Gateway的地址(192.168.1.254)

# 2.v2ray和dnsmasq的安装
## 2.1 安装v2ray的客户端[\[v2ray官网\]](https://www.v2ray.com/chapter_01/install.html#linux-安装脚本)
`bash <(curl -L -s https://install.direct/go.sh)`
## 2.2 配置客户端
客户端配置出一个socks5代理端口(以备不时之需)、一个dns端口和一个透明代理端口。虽然透明代理端口也能够处理udp的透明代理，但是我没有成功过;而且单独处理DNS方便分流。

#### 部分配置
```json
  "inbound": {
    "protocol": "socks",
    "port": 8080,
    "settings": {
      "auth": "noauth",
      "udp": false,
      "ip": "127.0.0.1"
    }
  },
  "inboundDetour": [
    {
      "protocol": "dokodemo-door",
      "port":1080,
      "settings":{
        "network": "tcp,udp",
        "timeout": 30,
        "followRedirect": true
      }
    }, 
    {
      "protocol": "dokodemo-door",
      "port":5353,
      "settings":{
        "address":"8.8.8.8",
        "port":53,
        "network": "udp",
        "timeout": 0,
        "followRedirect": false
      }
    }
  ]
```
注意，要取消v2ray自带的路由功能配置，在处理国内ip时与iptables分流会造成环路。具体原因，看官可自行分析，我在v2ray的issue中也有解释。

## 2.2 安装dnsmasq
`sudo apt install dnsmasq`
配置dnsmasq,实现按gfwlist规则的DNS解析分流。在`/etc/dnsmasq.conf`的末尾添加
```
conf-dir=/etc/dnsmasq.d
```
然后创建该目录
`mkdir -p /etc/dnsmasq.d`

*gfwlist2dnsmasq*是一个方便将gfwlist转换成dnsmasq配置文件的工具，安装并生成配置文件：
```bash
git clone https://github.com/cokebar/gfwlist2dnsmasq.git
cd gfwlist2dnsmasq
chmod +x gfwlist2dnsmasq.sh
./gfwlist2dnsmasq -o gfwlist.conf -s gfwlist
cp gfwlist.conf /etc/dnsmasq.d/
```
该配置将gfwlist中的域名通过**127.0.0.1:5353**解析，并将解析得到的ip加入到名为**gfwlist**的ipset中。

## 2.3 iptables和ipset设置
```bash
ipset -N gfwlist iphash
iptables -t nat -A PREROUTING -p tcp -m set --match-set gfwlist dst -j REDIRECT --to-port 1080
iptables -t nat -A OUTPUT -p tcp -m set --match-set gfwlist dst -j REDIRECT --to-port 1080
```
## 
至此，你拥有了一个无污染的dns服务器和一个透明代理服务器。使用时，需将其他网络设备的网关和dns都设置成该网关服务器的地址。

# 3 Connect Me
Connect Me via [GitHub](https://github.com/benyjuice/blog/) or [E-mail](mailto:fuxixi1991@gmail.com)


