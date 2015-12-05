---
layout: post
title: "Gre 隧道与 Keepalived"
date: 2015-12-05 10:29
comments: true
categories: System
description: keepalived
---

这一篇文章是做了不少功课的。

* [什么是 Gre 隧道](#第一节)
* [什么是 Vrrp ](#第二节)
* [KeepAlived 是什么](#第三节)
* [用Keepalived 怎么玩](#第四节)
* [附录](#第五节)


<h3 id="第一节">什么是 Gre 隧道 </h3>

GRE 隧道是一种 IP-2-IP 的隧道，是通用路由封装协议，可以对某些网路层协议的数据报进行封装，使这些被封装的数据报能够在 IPv4/IPv6 网络中传输。Tunnel 是一个虚拟的点对点的连接，提供了一条通路使封装的数据报文能够在这个通路上传输，并且在一个Tunnel 的两端分别对数据报进行封装及解封装。Linux 上创建 GRE 隧道，需要 ip_gre 内核模块，它是Pv4 隧道的驱动程序。

假设我有2台服务器，想做这两台之间创建 GRE 隧道

* $server_A_ip 表示服务器A的IP
* $server_B_ip 服务器B 的内网IP

```
# 在 A 机器上执行
# 创建 GRE 协议的 虚拟网络设备, 指定本地和对端 IP
sudo ip link add gretap1 type gretap local $server_a_ip remote $server_b_ip 
sudo ip link set dev gretap1 up  # 启动该设备
sudo ip addr add dev gretap1 10.1.1.2/24 # 给该设备一个虚拟网络地址

# 在 B 机器上执行
# 创建 GRE 协议的 虚拟网络设备, 指定本地和对端 IP
sudo ip link add gretap1 type gretap local $server_b_ip remote $server_a_ip 
sudo ip link set dev gretap1 up  # 启动该设备
sudo ip addr add dev gretap1 10.1.1.3/24 # 给该设备一个虚拟网络地址
```

如果想停止或者删除上述网卡

```
ip link set gretap1 down
ip tunnel del gretap1
```

至此点到点得隧道建立。


<h3 id="第二节">什么是 vrrp 协议 </h3>

VRRP(Virtual Router Redundancy Protocol), 即虚拟路由冗余协议。是实现路由器高可用的容错协议。

即将N台提供相同功能的路由器组成一个路由器组，这个组里面有一个 master 和多个 backup， 但在外界看来就像一台一样，构成虚拟路由器，拥有一个虚拟IP（vip），占有这个IP 的 master 实际负责 ARP 相应和转发 IP 数据包， 组中的其它路由器作为备份的角色处于待命状态。 master 会发组播消息，当 backup 在超时时间内收不到 vrrp 包时就认为 master 宕掉了，这时就需要根据VRRP的优先级来选举一个backup当master，保证路由器的高可用。


<h3 id="第三节"> Keepalived 是什么 </h3>

Keepalived 是一个基于 VRRP 协议来实现的服务高可用方案，可以利用其来避免IP单点故障。

* 一个经典的案例

一个WEB服务至少会有2台服务器运行 Keepalived，一台为主服务器，一台为备份服务器, 但是对外表现为一个虚拟IP，主服务器会发送特定的消息给备份服务器，当备份服务器收不到这个消息的时候，即主服务器宕机的时候，备份服务器就会接管虚拟IP，继续提供服务，从而保证了高可用性。


<h3 id="第四节">怎么玩 Keepalived</h3>

* 安装

```
sudo apt-get install keepalived
```

之前提到的，我们在 A, B 两台服务器建立起了 GRE 隧道了。 现在我们有一个虚拟的内网IP， 姑且叫做 $virtual_third_ip
接着在 A 服务器上

* 配置

编辑服务器 A, B 的 `/etc/keepalived/keepalived.conf`

```

global_defs {
    router_id LVS_DEVEL
}

vrrp_instance VI_1 {
    state MASTER
    interface gretap1 # 绑在建立起来的隧道上
    virtual_router_id 51
    # 优先级别,越高的设备会被弄成主设备, A,B 服务器要有所差别，其他都一样
    priority 100          advert_int 1      # 广播时间间隔
    authentication {  #  验证相关
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        $virtual_third_ip dev eth0
    }
}
```

我们将服务器  A 作为 Master, 服务器 B 当做 Backup, 当服务器 A 需要停机的时候，$virtual_third_ip 就会漂移到服务器 B 上面。 我们两台机器都有相同配置的 Nginx 服务，就可以保障机器出现故障的时候，Nginx 服务丝毫不受影响。


<h3 id="第五节"> 附录 </h3>

* [鸟哥的网络知识](http://linux.vbird.org/linux_server/0140networkcommand.php)
* [GRE tuneling](http://www.tldp.org/HOWTO/Adv-Routing-HOWTO/lartc.tunnel.gre.html)
* [VRRP](http://baike.baidu.com/link?url=N1-VGuzQC0PJ2bCnOzYn-XRTlN8eFGCvIJQlTI6KDL5Fx3EQxoRGTrxazb11jfZQqlfeA6q2Ka0VKRVEc0Kdu3GEyhqe1W_Ae2h0Tqu5NacIjOSaSnUVeOe-9QV5dB8q0Wv_uq8-vqdnQICt39UZFK)
