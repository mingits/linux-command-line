# 10. 进程

现代操作系统通常是<u>多任务</u>（*multitasking*）的，意味着它们通过迅速地从一个正在执行的程序切换到另一个程序，来创造一种计算机在同一时间内能做多个事务的错觉。Linux 内核用<u>进程</u>（*processes*）管理多任务。进程是 Linux 如何组织各个不同程序以等待 CPU 处理它们的程序。

有时，计算机会变得很迟钝，或者一个应用程序会停止响应。在本章中，我们将看到命令行中一些可用的工具，让我们检查程序正在做什么，并终结那些错误行为的进程。

本章将介绍下列命令：

- `ps` 报告当前所有进程的快照
- `top` 显示任务
- `jobs` 列出活动的工作
- `bg` 将一个工作置于后台
- `fg` 将一个工作置于前台
- `kill` 发送信号给进程
- `killall` 通过名称杀掉进程
- `shutdown` 关闭或重启系统

## 一个进程如何工作

当系统启动，内核发起一些自己的活动作为进程，并启动一个名为 `init` 的程序。`init` 反过来运行一系列被称为初始脚本（*init scripts*）的 shell 脚本（位于 `/etc`）以启动所有的系统服务。许多服务被当作<u>守护程序</u>（*daemon programs*）来执行，即程序仅仅在后台呆着，做着自己的工作，没有任何用户界面。所以，即使我们没有登录，系统已经在忙于开展一些例行事务了。

一个程序能运行其它程序，在进程方案中被表述为一个<u>父进程</u>（*parent process*）制造了一个<u>子进程</u>（*child process*）。

内核维护着关于每个进程的信息，以帮助保持良好的组织性。例如，每个进程被分配到一个称为<u>进程 ID</u>（*process ID* 即 *PID*）的数字。PID 按升序分配，`init` 进程总是得到 1 号 PID。内核还会跟踪分配给每个进程的内存，以及进程恢复执行的准备情况。和文件一样，进程也拥有属主和用户 ID、有效用户 ID 等。

## 查看进程

查看进程的几个命令中，最常用的一个是 `ps`。`ps` 程序有很多选项，但是最近单的使用形式是这样的：

```bash
[me@linuxbox ~]$ ps
  PID    TTY          TIME    CMD
 5198    pts/1    00:00:00    bash
10129    pts/1    00:00:00    ps
```

上例结果中列出了两个进程，分别是 `5198` 号的 `bash` 和 `10129` 号的 `ps`。我们可以看到，默认情况下 `ps` 不会显示给我们很多，而仅仅显示与当前终端会话关联的进程。要查看更多进程，我们需要加一些选项，不过首先让我们看一下 `ps` 输出的其它各个字段。`TTY` 是「teletype」的简写，指进程的<u>控制终端</u>（*controlling terminal*）。Unix 在这里显示该进程的寿命。`TIME` 字段是该进程消耗 CPU 的时长。如我们所见，没有进程让计算机工作得很辛苦。

如果我们加个选项，我们就能得到一张系统正在干嘛的更大的图片。

```bash
[me@linuxbox ~]$ ps x
  PID TTY STAT TIME  COMMAND
 2799 ?   Ssl  0:00  /usr/libexec/bonobo-activation-server –ac
 2820 ?   Sl   0:01  /usr/libexec/evolution-data-server-1.10 --
15647 ?   Ss   0:00  /bin/sh /usr/bin/startkde
15751 ?   Ss   0:00  /usr/bin/ssh-agent /usr/bin/dbus-launch --
15754 ?   S    0:00  /usr/bin/dbus-launch --exit-with-session
15755 ?   Ss   0:01  /bin/dbus-daemon --fork --print-pid 4 –pr
15774 ?   Ss   0:02  /usr/bin/gpg-agent -s –daemon
15793 ?   S    0:00  start_kdeinit --new-startup +kcminit_start
15794 ?   Ss   0:00  kdeinit Running...
15797 ?   S    0:00  dcopserver –nosid
and many more...
```

加了「x」选项（注意，没有前置的连字符）告诉 `ps` 显示所有进程，不论这些进程由哪个终端控制。`TTY` 字段的 `?` 指示没有控制终端。使用这个选项，我们看到了属于自己的全部进程。

### 用 top 动态查看进程



## 控制进程



### 中断一个进程



### 将一个进程置于后台



### 将一个进程带回前台



### 终止（暂停）一个进程



## 信号



### 用 kill 发送信号给进程



### 用 killall 发送信号给多个进程



## 关闭系统



## 更多进程相关的命令



## 总结

