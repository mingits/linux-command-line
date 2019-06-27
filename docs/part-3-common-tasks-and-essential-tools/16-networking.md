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

在输出的信息中，我们可以看到从我们的测试系统到 `slashdot.org` 需要穿越 16 个路由。路由器提供身份信息，我们可以看到主机名，IP 地址和包含在三个从本地系统到路由器的往返时间的性能数据。由于路由器不会提供身份信息（因为路由配置、网络拥堵、防火墙等等原因），我们在第二跳上看到的是星号。为了防止路由信息被阻挡，有时候可以加 `-T` 或 `-I` 选项来克服这一点。

### ip

`ip` 程序是一个多功能的网络配置工具，它利用现代 Linux 内核中可用功能的全系列网络。它取代了早期而现在已经弃用了的 `ifconfig` 程序。用 `ip`，我们可以检查系统的网络接口和路由表。

```bash
[me@linuxbox ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
        valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
        valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether ac:22:0b:52:cf:84 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.14/24 brd 192.168.1.255 scope global eth0
        valid_lft forever preferred_lft forever
    inet6 fe80::ae22:bff:fe52:cf84/64 scope link
        valid_lft forever preferred_lft forever
```

上面的例子中，我们看到测试系统上有两个网络接口。第一个叫 `lo`，是<u>回环接口</u>（*loopback interface*），一个虚拟接口，系统用来「对自己说话」。第二个叫 `eth0`，是以太网接口。

在执行临时网络诊断时，重要的是要在每个接口的第一行中寻找 `UP` 这个单词的存在，它指示了该网络接口是活跃的；还有第三行中 `inet` 域中有效的 IP 地址的存在。由于系统使用<u>动态主机配置协议</u>（*Dynamic Host Configuration Protocol* DHCP）

### netstat

`netstat` 程序用来检查各种网络设置和统计数据。通过使用其众多的选项，我们可以看到网络设置中的大量特性。使用 `-ie` 选项，我们能检查系统中的网络接口。

```bash
[me@linuxbox ~]$ netstat -ie
eth0    Link encap:Ethernet HWaddr 00:1d:09:9b:99:67
        inet addr:192.168.1.2 Bcast:192.168.1.255 Mask:255.255.255.0
        inet6 addr: fe80::21d:9ff:fe9b:9967/64 Scope:Link
        UP BROADCAST RUNNING MULTICAST MTU:1500 Metric:1
        RX packets:238488 errors:0 dropped:0 overruns:0 frame:0
        TX packets:403217 errors:0 dropped:0 overruns:0 carrier:0
        collisions:0 txqueuelen:100
        RX bytes:153098921 (146.0 MB) TX bytes:261035246 (248.9 MB)
        Memory:fdfc0000-fdfe0000
lo      Link encap:Local Loopback
        inet addr:127.0.0.1 Mask:255.0.0.0
        inet6 addr: ::1/128 Scope:Host
        UP LOOPBACK RUNNING MTU:16436 Metric:1
        RX packets:2208 errors:0 dropped:0 overruns:0 frame:0
        TX packets:2208 errors:0 dropped:0 overruns:0 carrier:0
        collisions:0 txqueuelen:0
        RX bytes:111490 (108.8 KB) TX bytes:111490 (108.8 KB)
```

使用 `-r` 选项将显示内核的网络路由表。显示网络是如何被配置为从一个网络发送数据包到另一个网络。

```bash
[me@linuxbox ~]$ netstat -r
Kernel IP routing table
Destination  Gateway      Genmask        Flags  MSS  Window  irtt  Iface

192.168.1.0  *            255.255.255.0  U        0  0       0     eth0
default      192.168.1.1  0.0.0.0        UG       0  0       0     eth0
```

本例中，我们看到一个典型的在防火墙/路由器之后的局域网（*local area network* LAN）中的一台客户机上的路由表。列表第一行，显示目标 `192.168.1.0`。以零结尾的 IP 地址指的是整个网络，而非单独的主机，所以该目标意为在局域网中的任意主机。第二个字段，网关（Gateway），是用于从当前主机到目标网络的网关（路由器）的名称或 IP 地址。字段中的星号表示不需要网关。

最后一行包含了目标 `default`。这意味着发往网络的任何流量都未在表中列出。在我们的例子中，可以看到网关被定义为一个路由器，地址是 `192.168.1.1`，假定其知晓如何处理目标流量。

和 `ip` 程序一样，`netstat` 有众多的选项，而我们仅仅学了两个。请查看 `ip` 和 `netstat` 的手册页以获取完整的选项列表。

## 通过网络传输文件

网络有什么好呢？除非我们能在网上移动文件。有许多程序可以在网络上移动数据。我们将学习其中的两个，更多的将放在后续章节中。

### ftp

作为一个「经典」的程序，`ftp` 得名于其所用的协议，<u>文件传输协议</u>（*File Transfer Protocol*）。FTP 曾是通过互联网下载文件应用最广的方法。大多数，如果不是全部，浏览器支持 FTP，你也会经常看到以 `ftp://` 开头的 URI。

在浏览器之前，是 `ftp` 程序。`ftp` 程序用来和 FTP 服务器通信，FTP 服务器包含可通过网络上传或下载的文件。

FTP （在其原始状态）是不安全的，因为它以<u>明文</u>（*cleartext*）发送用户名和密码。这意味着它们未曾加密，任何嗅探网络的人都能看到。由于此，几乎所有 FTP 能在互联网上做的，都由<u>匿名 FTP 服务器</u>（*anonymous FTP servers*）完成了。匿名 FTP 服务器允许任何人以「anonymous」作为用户名和一个无意义的密码登录。

在下面的例子中，我们会展示一个用 `ftp` 程序下载位于名为 `fileserver`  的匿名 FTP 服务器中的 `/pub/cd_images/Ubuntu-18.04` 目录中 Ubuntu ISO 映像文件的典型会话：

```bash
[me@linuxbox ~]$ ftp fileserver
Connected to fileserver.localdomain.
220 (vsFTPd 2.0.1)
Name (fileserver:me): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> cd pub/cd_images/Ubuntu-18.04
250 Directory successfully changed.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-rw-r--    1    500    500    733079552    Apr 25 03:53 ubuntu-18.04-desktop-amd64.iso
226 Directory send OK.
ftp> lcd Desktop
Local directory now /home/me/Desktop
ftp> get ubuntu-18.04-desktop-amd64.iso
local: ubuntu-18.04-desktop-amd64.iso remote: ubuntu-18.04-desktopamd64.iso
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for ubuntu-18.04-desktopamd64.iso (733079552 bytes).
226 File send OK.
733079552 bytes received in 68.56 secs (10441.5 kB/s)
ftp> bye
```

表 16-1 提供了对这个会话中的所键入的命令的解释。

表 16-1：`ftp` 命令交互示例

| 命令                                 | 意义                                                         |
| ------------------------------------ | ------------------------------------------------------------ |
| `ftp fileserver`                     | 调用 `ftp`  程序并使其连接到 FTP 服务器 `fileserver`。       |
| `anonymous`                          | 登录名。在登录提示符之后，会出现一个密码提示符。有些服务器会接受一个空密码，有些则需要如电子邮件地址格式一样的密码，若如此，请尝试如 `user@example.com` 形式的密码。 |
| `cd pub/cd_images/Ubuntu-18.04`      | 在远程系统中切换目录到包含想要的文件所在的目录。注意，在大多数匿名 FTP 服务器上，供公开下载的文件的可以在 `pub`  下找到。 |
| `ls`                                 | 列出远程系统的目录内容。                                     |
| `lcd Desktop`                        | 切换本地系统的目录到 `~/Desktop`。在例子中，调用 `ftp` 时的工作目录是 `~`。该命令将工作目录切换到 `~/Desktop`。 |
| `get ubuntu-18.04-desktop-amd64.iso` | 告知远程系统传输 `ubuntu-18.04-desktop-amd64.iso` 文件到本地系统。由于本地系统的工作目录已经切换到 `~/Desktop`，文件也将被下载到该目录中。 |
| `bye`                                | 登出远程系统并终结 `ftp` 程序会话。也常用 `quit` 和 `exit` 命令。 |

在 `ftp>` 提示符下输入 `help` 会显示受支持的命令列表。在已授予足够权限的服务器上使用 `ftp`，可以通过网络执行许多普通文件管理任务的传输文件。 这很笨拙，但确实有效。

### lftp - 一个更好的 ftp

`ftp` 不是仅有的命令行 FTP 客户端。事实上有很多这样的程序。其中，一个更好（也更受欢迎）的程序，是由 Alexander Lukyanov 写的 `lftp`。它的用法很像传统的 `ftp` 程序，不过有许多额外的便利特性，包括支持多协议（包括 HTTP）、下载失败后的自动重试、后台进程、Tab 键补全路径名，等等等等。

### wget

另一个流行的用于文件下载的命令行程序是 `wget`。对于下载网络和 FTP 站点的内容来说，它很有用。它可以下载单个文件、多个文件、甚至整个站点。要下载 `linuxcommand.org` 的第一个页面，我们可以这么做：

```bash
[me@linuxbox ~]$ wget http://linuxcommand.org/index.php
--11:02:51-- http://linuxcommand.org/index.php
           => `index.php'
Resolving linuxcommand.org... 66.35.250.210
Connecting to linuxcommand.org|66.35.250.210|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: unspecified [text/html]

    [ <=>                      ] 3,120               --.--K/s

11:02:51 (161.75 MB/s) - `index.php' saved [3120]
```

程序的许多选项，允许 `wget` 递归式下载、在后台下载（允许你登出但继续下载）、完成一个已下载了部分的文件。这些特性在其优于平均水平的手册页中有很好的记述。

## 与远程主机安全通信

很多年来，类 Unix 操作系统

### ssh



### scp 和 sftp



## 总结



## 扩展阅读
