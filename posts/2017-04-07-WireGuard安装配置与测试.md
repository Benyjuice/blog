---
title: WireGuard安装配置与测试
date: 2017-04-07 17:34:04
---
[WireGuard](https://www.wireguard.io/)是由[ZX2C4](https://www.zx2c4.com/)开发，目前还处于开发阶段，可能会不稳定或者其他安全问题。

# 1.安装WireGuard
WireGuard提供了Ubuntu、Debian、Fedora/CentOS、Arch、OpenSUSE、Gentoo、Exherbo、
NixOS、OpenWRT、Void、Mac OS X（只有管理工具，没有内核模块）等发行版/操作系统的包。

详细情况或者获取最新的支持更新请访问https://www.wireguard.io/install/#option-a-distribution-packages

下面介绍如何从源码安装WireGuard

## 1.1 安转必要的工具
Ubuntu或Debian（其他发行版请参考https://www.wireguard.io/install/#option-b-compiling-from-source）
```bash
sudo apt-get install libmnl-dev linux-headers-$(uname -r) build-essential pkg-config
```
## 1.2 获取源码
官方推荐下载最新的的源码压缩包（最新的源码包请参考https://www.wireguard.io/install/#option-b-compiling-from-source）:
```bash
wget https://git.zx2c4.com/WireGuard/snapshot/WireGuard-0.0.20170324.tar.xz
```
或者用```git```获取源码:
```bash
git clone https://git.zx2c4.com/WireGuard
```

## 1.3 编译内核模块和工具
```bash
cd WireGuard/src
make
```

## 1.4 安装（需要root权限）
```bash
make install
```
这将同时安装内核模块`wireguard.ko`和配置工具`wg`、`wg-quick`

# 2.配置
## 2.1 基于命令行的配置
`wg`是WireGuard提供的命令行配置工具，但仍然要借助linux的其他工具才能完成配置，如`ip、 resolvconf`等

### 2.1.1 服务端配置
服务端需要有公网IP，其配置过程如下（某些命令需要root权限）:
```bash
wg genkey > privateA #生成服务端的私钥
wg pubkey < privateA > publicA #由私钥生成公钥
ip link add dev wg0 type wireguard #增加一个新的网卡wg0
ip address add dev wg0 192.168.2.1/24 #给wg0设置IP地址
wg set wg0 listen-port 51820 private-key /path/to/privateA peer <客户端的公钥> allowed-ips 192.168.2.0/24 #启动WireGuard, /path/to/privateA为私钥文件路径， 客户端的公钥为下一步为客户端生成的公钥
```

### 2.1.2 客户端配置
客户端与服务端配置基本类似，只有开启WIreGuard的擦书不一样
```bash
wg genkey > privateB #生成服务端的私钥
wg pubkey < privateB > publicB #由私钥生成公钥
ip link add dev wg0 type wireguard #增加一个新的网卡wg0
ip address add dev wg0 192.168.2.2/24 #给wg0设置IP地址,与服务端在同一网段
wg set wg0 listen-port 51820 private-key /path/to/privateB peer <服务端的公钥> allowed-ips 0.0.0.0/0 end-point <服务端IP>:51820 #启动WireGuard, 服务端的公钥为上一步为客户端生成的公钥.allowed-ips必须为0.0.0.0/0
```


此时，在客户端执行`ping -I wg0 192.168.2.1`应该可以看到响应了。

### 2.1.3 路由和防火墙配置
客户端需要配置路由，下面简单粗暴地将所有流量都经`wg0`转发:
```bash
ip route add default dev wg0 via 192.168.2.1
```
客户端还可能需要设置DNS，直接编辑`/etc/resolv.conf`即可.

服务端需要配置NAT:
```bash
iptables -t nat -I POSTROUTING -s 192.168.2.0/24 -j MASQUERADE
```

## 2.2 基于配置文件的配置
`WireGuard`提供了更加灵活方便的配置工具`wg-quick`,其基于配置文件能够快速的设置服务端和客户端，并且能够方便的在服务端实现多用户。

下面是一个多用户的服务端配置文件例子:
`file wg0.conf`
```
[Interface]
Address = 10.192.122.1/24
Address = 10.10.0.1/16
SaveConfig = true
PrivateKey = yAnz5TF+lXXJte14tji3zlMNq+hd2rYUIgJBgB3fBmk=
ListenPort = 51820

[Peer]
PublicKey = xTIBA5rboUvnH4htodjb6e697QjLERt1NAB4mZqp8Dg=
AllowedIPs = 10.192.122.3/32, 10.192.124.1/24

[Peer]
PublicKey = TrMvSoP4jYQlY6RIzBgbssQqY3vxI2Pi+y71lOWWXX0=
AllowedIPs = 10.192.122.4/32, 192.168.0.0/16

[Peer]
PublicKey = gN65BkIKy1eCE9pP1wdc8ROUtkHLF2PfAqYdyYBz6EA=
AllowedIPs = 10.10.10.230/32
```
该配置文件允许3个客户端链接,使用wg-quick开启服务:
```bash
wg-quick up /path/to/wg0.conf
```
如果配置文件位于`/etc/wireguard/`目录下，可以直接使用`wg-quick up wg0`开启.


客户端的配置与服务端类似:
```
[Interface]
Address = 10.200.100.8/24
PostUp = echo nameserver 10.200.100.1 | resolvconf -a %i -m 0 -x
PostDown = resolvconf -d %i
PrivateKey = oK56DE9Ue9zK76rAc8pBl6opph+1v36lm7cXXsQKrQM=
PresharedKey = /UwcSPg38hW/D9Y3tcS1FOV0K1wuURMbS0sesJEP5ak=

[Peer]
PublicKey = GtL7fZc/bLnqZldpVofMCD6hDjrK28SsdLxevJ+qtKU=
AllowedIPs = 0.0.0.0/0
Endpoint = demo.wireguard.io:51820
```
其中PostUp字段为开启时执行的命令PostDown是关闭时执行的命令，此处用来设置和清除DNS.

## 2.3 systemd服务
略

# 3.与ss的对比测评
先说我的网络环境，家用宽带电信50M光纤，上传40M，爬墙服务器是gcloud的试用机，asia-east1-c节点。
## 3.1 YouTube 4k测试
* ss结果
![ss](/picture/youtube_4k_ss.png)

* WireGuard节点
![WireGuard](/picture/youtube_4k_wg.png)

## 3.2 speedtest测试
* ss结果
![ss](http://www.speedtest.net/result/6197914145.png)

* WireGuard结果
![wireguard](http://www.speedtest.net/result/6197919202.png)

## 3.3 结论
可以看出，在我的网络环境下，WireGuard的实测网速还没有ss来的快，个人认为是由于WireGuard底层由UDP封装的缘故。

但不可否认，WireGuard的底层在Linux内核中实现，其效率应该不会太差，根据官方的文档WireGuard比OpeVPN和IPSec的效率更好，参见https://www.wireguard.io/performance/#benchmarking

# 4 Connect Me
Connect Me via [GitHub](https://github.com/benyjuice/blog/) or [E-mail](mailto:fuxixi1991@gmail.com)
