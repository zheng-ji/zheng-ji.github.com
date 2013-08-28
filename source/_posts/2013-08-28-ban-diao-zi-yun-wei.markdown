---
layout: post
title: "半吊子运维"
date: 2013-01-30 13:52
comments: true
categories: Server
---
今日一台服务器挂了。有幸来到了中华广场的中捷机房。体验了网宿的生活，机房里机柜林立，管理森严（无数保安与监控）。关键是又冷又吵.此行目的给它重装了系统，写代码的人也能有机会做做半吊子运维了.携带了U盘是Ubuntu LTS 12.04的镜像

####400 G的硬盘做了分区：
在删除了原来的分区之后，建立了新的两个Primary分区，分别是
+ 挂载点/ 给了76G大小，文件系统的类型是ext4
+ /home 给了320G大小，文件系统的类型是ext4
+ swap 给了是4G（跟内存一致）的空间

#### 配置源的修改并更新
```
sudo vi /etc/apt/source.list
sudo apt-get dist-upgrade
```
ssh 登录的端口号以及权限设置,安全很重要

```
sudo vi /etc/ssh/sshd_config
#更改端口号
Port xxxx
#修改
PermitRoorLogin No
```

激活另一块网卡（主机有2个网卡）

```
sudo vi /etc/network/interface
iface eth1 inet static
address 192.168.1.120
netmask 255.255.255.0
```
执行命令 

```
ifup eth1
```

Linux系统性能调优,sysctl.conf的编辑

```
net.ipv4.ip_forward = 0
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.default.accept_source_route = 0
kernel.sysrq = 0
kernel.core_uses_pid = 1
kernel.shmmni = 4096
kernel.sem = 250 32000 100 128
fs.file-max = 6553600
fs.nr_open = 5000000
net.ipv4.tcp_syncookies = 1
kernel.msgmnb = 65536
kernel.msgmax = 65536
kernel.shmmax = 68719476736
kernel.shmall = 4294967296
net.ipv4.tcp_max_tw_buckets = 200000
net.ipv4.tcp_sack = 1
net.ipv4.tcp_window_scaling = 1
net.ipv4.tcp_rmem = 4096 87380 4194304
net.ipv4.tcp_wmem = 4096 16384 4194304
net.core.wmem_default = 8388608
net.core.rmem_default = 8388608
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.core.netdev_max_backlog = 300000
net.core.somaxconn = 300000
net.ipv4.tcp_max_orphans = 3276800
net.ipv4.tcp_max_syn_backlog = 300000
net.ipv4.tcp_timestamps = 1
net.ipv4.tcp_synack_retries = 1
net.ipv4.tcp_syn_retries = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_mem = 94500000 915000000 927000000
net.ipv4.tcp_fin_timeout = 1
net.ipv4.tcp_keepalive_time = 30
net.ipv4.ip_local_port_range = 1024 65000
```

使sysctl生效

```
/sbin/sysctl -p
/sbin/sysctl -w net.ipv4.route.flush=1
```

修改NTP网络时间

```
sudo apt-get install ntp
```









