---
title: "斐讯 K2 刷 LEDE 并配置 MentoHUST & IPv6"
date: 2018-07-06 00:49:38
draft: false
toc: true
images:
tags: 
  - untagged
markup: pandoc
---

前几天正值考试之际，我他喵的居然刷起了路由器，真的是罪过罪过。
<!-- more -->

------

# 前置条件
由于斐讯的原厂系统基本不能满足各种折腾的需要，因此刷系统是必须的。一般的思路都是刷完 Breed 之后刷 Pandavan，但是 Pandavan 比较难弄的的是子网中 IPv6 的配置。也就是为了这个 IPv6 我从已经用了快两年的 Pandavan 换到了 LEDE。  
自然，这里说得前置条件就是路由器已经刷好了 Breed。由于相关教程网上很多，所以在此不做叙述。

# 安装 LEDE 与配置 MentoHUST
从 OpenWrt 的官网我们可以找到适用于 K2 使用的刷机文件与各种软件的二进制安装包。之所以要找到软件的安装包是因为刷完 LEDE 之后你可能会玄学地发现有些联网必用的软件都没有，因此需要自己手动安装，然而此时路由器又无法联网，所以你懂的。  
安装好 LEDE 之后，利用 scp 工具将 OpenWrt 可用的 MentoHUST 可执行文件（这个 GitHub 仓库里有一个已经编译好了的）传至路由器上，必要的话 `chmod` 增加可执行权限，并移动至 `/bin` 路径下（反正我移了）。  
紧接着配置 MentoHUST 注意对于华科而言『用户名』是学号一定要注意首字母要大写大写大写（不要问我是怎么知道的），接着『组播地址』选 `1` 即 `锐捷`，『DHCP 方式』选 `1` 即 `二次认证`。其他学校的我不清楚，所以自己百度吧。如果始终认证不成功可能需要抓包，具体操作自行百度。  
至此，路由器应该是可以成功认证并且联网了，不需要 IPv6 的童鞋就可以出门右转利用 opkg 或者自己编译进行更加深入的折腾了。

# 配置 IPv6
1. 利用 opkg 安装 `ip6tables` 与 `kmod-ipt-nat6` 两个包,其中二者分别提供 IPv6 的防火墙与 NAT 支持。
2. 修改文件 `/etc/config/network` 以添加一下字段，若已存在则可不修改或将前缀修改为自己喜欢的。
    ```bash
    config globals 'globals'
           option ula_prefix 'eeee:eeee:eeee::/48'
    ```
    这里实际上设置的是路由器子网中 IPv6 的 IP 池。
3. 为了让 OpenWRT 后面的设备始终能够获得 IPv6 网关，需要在 `/etc/config/dhcp` 文件的 `config dhcp 'lan'` 将 `option ra_default` 设置为 `1`。否则，由于默认分配的是私网地址，OpenWRT 不会向下游设备公布 IPv6 默认路由 (即网关)，可能导致路由器上 IPv6 连通但下游设备不通的情况。
4. 由于 OpenWrt 默认不会配置 IPv6的 NAT 表，因此需要在 `/etc/firewall.user` 文件中加入
    ```bash
    WAN6=eth0
    LAN=br-lan
    ip6tables -t nat -A POSTROUTING -o $WAN6 -j MASQUERADE
    ip6tables -A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
    ip6tables -A FORWARD -i $LAN -j ACCEPT
    ```
    其中 `WAN6` 与 `LAN` 都应改为 `ifconfig` 命令中所匹配的网卡名。
5. 在路由器上利用 `ip -6 route` 将得到如下结果：
    ```bash
    default from 2402:f000:x:xxxx::/64 via fe80::xxxx:xxxx:xxxx:xxxx dev eth0  proto static  metric 512
    ```
    可见默认网关为 `fe80::xxxx:xxxx:xxxx:xxxx`，我们需要将其加入路由表中，因此执行：
    ```bash
    route -A inet6 add default gw fe80::xxxx:xxxx:xxxx:xxxx dev eth0.2
    ```
    需要注意的是，这个命令在每次重启之后都需要被执行。
6. 如果 IPv6 访问不正常可能是 DNS 污染造成的，因此需要手动修改 WAN6 的 DNS。方法是在 `Network->Interfaces->WAN6->Edit->Common Configuration->Advanced Settings` 中取消勾选 `Use DNS servers advertised by peer` 并在 `Use custom DNS servers` 中设置可用的 IPv6 DNS 服务器（谷歌的是 `2001:4860:4860::8888` 与 `2001:4860:4860::8844`）。

# 开机自动运行
MentoHUST 与 添加默认网关的命令都是需要开机自动执行的，当然 Linux 下开机自动执行的实现方式有很多种，这里给出在我的路由器上可用的一种方式。  
在 `/etc/init.d/` 新建文件 `addipv6route`，内容为
```bash
#!/bin/sh /etc/rc.common
# /etc/init.d/addipv6route
START=99
STOP=10
start()
{
route -A inet6 add default gw fe80::1614:4bff:fe7d:4cbd dev eth0.2
logger -t ipv6 "***ipv6***"
}
stop()
{
logger -t ipv6 "***ipv6***"
}
restart()
{
stop
start
}
```
保存并修改权限为 `755`，建立软链接至 `/etc/ec.d/S99addipv6route`。
在 `/etc/init.d/` 新建文件 `mentohust`，内容为
```bash
#!/bin/sh /etc/rc.common
# /etc/init.d/mentohust
START=99
STOP=10
start()
{
mentohust &
logger -t mentohust "***mentohust***"
}
stop()
{
mentohust -k &
}
restart()
{
stop
start
}
```
保存并修改权限为 `755`，建立软链接至 `/etc/ec.d/S99mentohust`。

# WiFi 信号优化
这一步比较玄学，具体的原因我也不是很懂，所以我就说一下我是怎么调的，反正这样调完之后信号真的很不错。

1. 将路由器平放与桌面上并将天线调整为比较正常的竖直向上的朝向（之前我就是因为把路由器竖着放而且改了两根天线的朝向所以不论怎么调信号都不好），当然一旦调好之后路由器怎么放似乎都影响不是特别大了。
2. 对于 5G，似乎信号确实与 LEDE 中设置的发射功率成正相关，故将 『Transmit Power』 的值设置为最大，即 `23dBm` 或 `20dBm`；对于 2.4G，一个有趣的现象是若将发射功率设置为最大则可能出现可以连上路由器但是无法上网的情况，所以我就将 『transmit Power』 设置为 `auto`。『Country Code』 将影响 `Transmit Power` 的最大值，因此也可以测试一下，但是我的设置为 `CN - China` 就已经可以满足我的需求了，所以就没继续测。

最后的结果大概就是离路由器比较近的话 5G 的话跑到 10MB/s 完全没有问题，而 2.4G 的速度在 1~2MB/s。

# 参考资料
-   [Lede、PandoraBox IPv6 NAT66 的实战操作【亲测】](!http://www.right.com.cn/forum/thread-253712-1-1.html)
-   [OpenWRT 路由器作为 IPv6 网关的配置](!https://github.com/tuna/ipv6.tsinghua.edu.cn/blob/master/openwrt.md)
-   [用路由器连上 IPv6 网络](!https://medium.com/@invisprints/%E7%94%A8%E8%B7%AF%E7%94%B1%E5%99%A8%E8%BF%9E%E4%B8%8A-ipv6-%E7%BD%91%E7%BB%9C-f5193fd02712)
