# 16. 网络

对于网络，大概没有什么是 Linux 不能做的。Linux 被用来建构所有种类的网络系统和应用程序，包括防火墙、路由器、域名服务、网络附加存储（NAS network-attached storage），等等等等。

网络的课题是如此宽泛，需要用到大量的命令来配置和控制。我们将注意力聚焦在一些最常用到的那些命令。选择检查的命令包括那些用来监测网络和传输文件的命令。另外，我们打算探查一个用来远程登录的 `ssh` 程序。本章将学习下列命令：

- `ping` 发送一个 ICMP ECHO_REQUEST 到网络主机
- `traceroute` 打印追踪到网络主机的路由包
- `ip` 显示 / 操纵路由、设备、策略路由和通道
- `netstat` 打印网络连接、路由表、接口统计信息、伪装链接、组播成员
- `ftp` 互联网文件传输程序
- `wget` 非交互式网络下载器
- `ssh` OpenSSH  客户端（远程登录程序）

我们假设有一定的网络背景知识。现在，互联网时代，每个使用计算机的人都需要懂一点基本的网络概念。要充分理解本章，我们需要熟悉下列术语：

- IP 地址（互联网协议地址）
- 主机和域名
- 统一资源标识符（URI）

请查看「扩展阅读」章节，获取关于这些术语的有用的文章。

> **注意：**将要学习的一些命令可能（基于你所用的发行版）需要从你的发行版仓库安装额外的包，有些命令可能需要超级管理员权限来执行。

## 检查和监视网络

即使你不是系统管理员，检查网络的性能和操作，也会很有助益。

### ping

最常用的基本网络命令是 `ping`。`ping` 命令发送一个名为 ICMP ECHO_REQUEST 的专用网络包到一台指定主机。大多数网络设备在收到这个包之后会对其回复，允许验证网络连接。

> **注意：**也可以配置大多数的网络设备（包括 Linux 主机）以忽略这些包。这样部分地遮掩主机，多数是出于安全因素，以防潜在的攻击者。配置防火墙为阻止 ICMP 流量，也很常见。

例如，要看我们是否能到达 `linuxcommand.org`（一个我们最常去的站点），我们可以这样用 `ping`：

```bash
[me@linuxbox ~]$ ping linuxcommand.org
```

一旦启动，`ping` 会持续按指定的间隔（默认是一秒）发送数据包，直到它被中断为止。

```bash
[me@linuxbox ~]$ ping linuxcommand.org
PING linuxcommand.org (66.35.250.210) 56(84) bytes of data.
64 bytes from vhost.sourceforge.net (66.35.250.210): icmp_seq=1 ttl=43 time=107 ms
64 bytes from vhost.sourceforge.net (66.35.250.210): icmp_seq=2 ttl=43 time=108 ms
64 bytes from vhost.sourceforge.net (66.35.250.210): icmp_seq=3 ttl=43 time=106 ms
64 bytes from vhost.sourceforge.net (66.35.250.210): icmp_seq=4 ttl=43 time=106 ms
64 bytes from vhost.sourceforge.net (66.35.250.210): icmp_seq=5 ttl=43 time=105 ms
64 bytes from vhost.sourceforge.net (66.35.250.210): icmp_seq=6 ttl=43 time=107 ms

--- linuxcommand.org ping statistics ---
6 packets transmitted, 6 received, 0% packet loss, time 6010ms
rtt min/avg/max/mdev = 105.647/107.052/108.118/0.824 ms
```

在按 `Ctrl-c` 中断之后（本例中是在第六个包之后），`ping` 打印出性能统计。一个表现良好的网络会展示 0% 的丢包。一个成功的 `ping` 将表明，网络（它的接口卡、电缆、路由和网关）的各个元素正处在总体良好的工作秩序中。

### traceroute

`traceroute` 程序（有些系统中使用类似的 `tracepath` 程序替代）列出从本地系统到指定主机的网络流量的所有的「跳」。如，要看到达 `slashdot.org` 的路由，可以这样做：

```bash
[me@linuxbox ~]$ traceroute slashdot.org
```

输出是这样的：

```bash
traceroute to slashdot.org (216.34.181.45), 30 hops max, 40 byte packets
 1 ipcop.localdomain (192.168.1.1) 1.066 ms 1.366 ms 1.720 ms
 2 * * *
 3 ge-4-13-ur01.rockville.md.bad.comcast.net (68.87.130.9) 14.622 ms 14.885 ms 15.169 ms
 4 po-30-ur02.rockville.md.bad.comcast.net (68.87.129.154) 17.634 ms 17.626 ms 17.899 ms
 5 po-60-ur03.rockville.md.bad.comcast.net (68.87.129.158) 15.992 ms 15.983 ms 16.256 ms
 6 po-30-ar01.howardcounty.md.bad.comcast.net (68.87.136.5) 22.835 ms 14.233 ms 14.405 ms
 7 po-10-ar02.whitemarsh.md.bad.comcast.net (68.87.129.34) 16.154 ms 13.600 ms 18.867 ms
 8 te-0-3-0-1-cr01.philadelphia.pa.ibone.comcast.net (68.86.90.77) 21.951 ms 21.073 ms 21.557 ms
 9 pos-0-8-0-0-cr01.newyork.ny.ibone.comcast.net (68.86.85.10) 22.917 ms 21.884 ms 22.126 ms
10 204.70.144.1 (204.70.144.1) 43.110 ms 21.248 ms 21.264 ms 11 cr1-pos-0-7-3-1.newyork.savvis.net (204.70.195.93) 21.857 ms
cr2-pos-0-0-3-1.newyork.savvis.net (204.70.204.238) 19.556 ms cr1-pos-0-7-3-1.newyork.savvis.net (204.70.195.93) 19.634 ms
12 cr2-pos-0-7-3-0.chicago.savvis.net (204.70.192.109) 41.586 ms 42.843 ms cr2-tengig-0-0-2-0.chicago.savvis.net (204.70.196.242) 43.115 ms
13 hr2-tengigabitethernet-12-1.elkgrovech3.savvis.net (204.70.195.122) 44.215 ms 41.833 ms 45.658 ms
14 csr1-ve241.elkgrovech3.savvis.net (216.64.194.42) 46.840 ms 43.372 ms 47.041 ms
15 64.27.160.194 (64.27.160.194) 56.137 ms 55.887 ms 52.810 ms
16 slashdot.org (216.34.181.45) 42.727 ms 42.016 ms 41.437 ms
```

在输出的信息中，我们可以看到从我们的测试系统到 `slashdot.org` 需要穿越 16 个路由。路由器提供身份信息，我们可以看到主机名，IP 地址和性能数据

### ip



### netstat



## 通过网络传输文件



### ftp



### lftp - 一个更好的 ftp



### wget



## 与远程主机安全通信



### ssh



### scp 和 sftp



## 总结



## 扩展阅读

