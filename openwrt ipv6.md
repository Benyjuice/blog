#OpenWrt Barrier Breaker 14.07 IPv6 Configuretion in Compus Network

## 1.链接IPv6
OpenWrt Barrier Breaker 14.07内建IPv6支持,我的mw150r v5.0/v6.3刷入 TL-WR741N/ND v4的固件之后，PPPoE拨号直接获取IPv6地址
不用复杂的设置。记得在高级选项启用IPv6协商(Enable IPv6 negotiation on the PPP link)。

## 2.使用*radvd*转播IPv6地址及转发规则设置
### 2.1 安装必要package
`opkg install radvd kmod-ip6tables kmod-ipt-nat6 ip6tables ip`
###2.2 配置/etc/config/radvd.conf
>interface br-lan {  
>  AdvSendAdvert on;  
>   MinRtrAdvInterval 5;  
>   MaxRtrAdvInterval 10;  
>   AdvManagedFlag off;  
>    AdvOtherConfigFlag off;  
>    AdvDefaultPreference high;  
>    prefix fc00:0101:0101::/64 {  
>        AdvOnLink on;  
>        AdvAutonomous on;  
>        AdvRouterAddr on;  
>    };  
>};  

*注意:此处使用了fc00:0101:0101::/64作为前缀(prefix)  

### 2.3 配置/etc/config/network  
删掉以下内容  
config globals 'globals'                                             #删掉  
    option ula_prefix 'xxxxxxxxxxxx'                          #删掉  
  
config interface ‘lan’  
    option ip6assign ‘60’                                            #删掉  
在 config  interface 'lan'下添加  
option ip6addr 'fc00:0101:0101::1/64'
*注意:此处必须与/etc/config/radvd.conf的prefix相符  
在末尾添加:  
config route6  
        option interface 'wan6'  
        option target '::/1'  
        option metric '256'  

config route6  
        option interface 'lan'  
        option target 'fc00:1:2:3::/64'  
        option metric '256'  
        
保存退出  
### 2.3 配置/etc/cnfig/dhcp
删除以下内容:  
config dhcp 'lan'  
     option dhcpv6 'server'                                          #删掉  
     option ra 'server'                                                  #删掉  
### 2.4 使radvd开机启动
正常使用`/etc/init.d/radvd enable`即可,如果这样启动失败，请自行解决或者在/etc/rc.local文exit 0前加入:  
`radvd -C /etc/config/radvd.conf`

### 2.5 配置转发规则/etc/firware.user
加入ip6tables -t nat -I POSTROUTING -s fc00:101:101::/64 -j MASQUERADE
*注意:此处fc00:101:101::/64必须与/etc/config/radvd.conf的prefix一致


## 参考链接
https://blog.blahgeek.com/2014/02/22/openwrt-ipv6-nat/ [在OpenWRT上安装使用IPv6 NAT]部分  
http://bbs.rs.xidian.me/forum.php?mod=viewthread&tid=741739&mobile=2 [5、以防万一，建议大家删掉以下语句]部分和[5.17添加内容]关于路由表说明部分。由于不是西电的学生博主无法查看其附件内容。
