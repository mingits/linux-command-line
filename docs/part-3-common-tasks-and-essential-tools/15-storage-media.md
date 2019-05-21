# 15. 存储媒体

前几章中，我们了解了在文件层面处理数据。本章中，我们将考虑在设备层面的数据。Linux 具有惊人的处理存储设备的能力，无论物理存储，如硬盘、网络存储，还是虚拟存储设备，如 RAID（Redundant Array of Independent Disks 独立磁盘冗余阵列）和 LVM（Logical Volume Manager 逻辑卷管理器）。

然而，因为本书不是关于系统管理的，我们不会很深入的讨论这个主题。我们将尝试的是介绍一些管理存储设备的概念和主要命令。

要完成本章的练习，我们会用到一个 USB 闪存和一个 CD-RW（光盘烧录设备）。

我们将了解下列命令：

- `mount` 加载一个文件系统
- `umount` 卸载一个文件系统
- `fsck` 检查并修复文件系统
- `fdisk` 处理磁盘分区表
- `mkfs` 创建一个文件系统
- `dd` 转换并复制一个文件
- `genisoimage` (`mkisofs`) 创建一个 ISO 9660 的映像文件
- `wodim` (`cdrecord`) 将数据刻录到光存储媒体中
- `md5sum` 计算 MD5 校验和

## 加载和卸载存储设备

Linux 桌面的最新进展，已经使得存储设备的管理对桌面用户而言变得极其简单。大部分情况下，我们加载一个设备到系统中，它会「立刻工作」。在早些日子（说的是 2004 年），这项工作不得不手动完成。在非桌面系统（如服务器），由于服务器经常有大量的存储需求和复杂的配置要求，这仍然是一个很大程度上手动操作的过程。

管理存储设备的第一步，是将设备附加到文件系统树中，这个过程，被称为<u>加载</u>（*mounting*），以允许该设备与操作系统交互。回忆一下第二章，类 Unix 系统，如 Linux，维护着单一的文件系统树，设备都附加在不同的加载点上。对比其它操作系统，如 MS-DOS 和 Windows，维护的是每个设备相互分离的文件系统树（如 `C:\`、`D:\` 等）

一个名为 `/etc/fstab`（「file system table」文件系统表的缩写）列出了在启动时被加载的设备（通常是硬盘分区）。这里有个来自 Fedora 系统的例子：

```bash
LABEL=/12        /         ext4    defaults         1 1
LABEL=/home      /home     ext4    defaults         1 2
LABEL=/boot      /boot     ext4    defaults         1 2
tmpfs            /dev/shm  tmpfs   defaults         0 0
devpts           /dev/pts  devpts  gid=5,mode=620   0 0
sysfs            /sys      sysfs   defaults         0 0
proc             /proc     proc    defaults         0 0
LABEL=SWAP-sda3  swap      swap    defaults         0 0
```

上例中所列的大多数文件系统是虚拟的，我们不会讨论。对我们有用的，是前三个：

```bash
LABEL=/12        /         ext4    defaults         1 1
LABEL=/home      /home     ext4    defaults         1 2
LABEL=/boot      /boot     ext4    defaults         1 2
```

这些是硬盘分区。每行由六个字段组成，在表 15-1 中详述。

表 15-1：`/etc/fstab` 字段

| 字段 | 内容         | 详述                                                         |
| ---- | ------------ | ------------------------------------------------------------ |
| 1    | 设备         | 传统上该字段包含与物理设备关联的设备文件的实际名称，如 `/dev/sda1`（检测到的第一块硬盘的第一个分区）。但在当下的计算机中，由于有众多的设备是热插拔的（如 USB 硬盘），许多现代 Linux 发行版会用一个文本标签替代。标签（在格式化存储设备之时被添加上的）可以是简单的文本标签，伙食一个随即生成的<u>通用唯一标识符</u>（UUID: Universally Unique Identifier）该标签在设备加载到系统之时由操作系统读取。不论哪个设备文件被分配到实际的物理设备，系统依然能正确地识别它。 |
| 2    | 加载点       | 设备被附加到文件系统树上的目录位置。                         |
| 3    | 文件系统类型 | Linux 允许加载多种不同的文件系统类型。多数原生 Linux 文件系统是第四代扩展文件系统（ext4），不过还支持许多类型，如 FAT16（msdos），FAT32（vfat），NTFS（ntfs），CD-ROM（iso9660）等等。 |
| 4    | 选项         | 文件系统可以通过各种选项得以加载。如以只读方式加载文件系统，或者可以阻止从该文件系统中执行所有程序（这是对可移动媒体的一个有用的安全特性）。 |
| 5    | 频率         | 一个数字，指定是否以及何时使用 `dump` 命令备份文件系统。     |
| 6    | 顺序         | 一个数字，用于指定应使用 `fsck` 命令检查文件系统的顺序。     |

### 查看已加载的文件系统清单

`mount` 命令用来加载文件系统。不带任何参数的键入命令，将显示当前被加载的文件系统列表：

```bash
[me@linuxbox ~]$ mount
/dev/sda2 on / type ext4 (rw)
proc on /proc type proc (rw)
sysfs on /sys type sysfs (rw)
devpts on /dev/pts type devpts (rw,gid=5,mode=620)
/dev/sda5 on /home type ext4 (rw)
/dev/sda1 on /boot type ext4 (rw)
tmpfs on /dev/shm type tmpfs (rw)
none on /proc/sys/fs/binfmt_misc type binfmt_misc (rw)
sunrpc on /var/lib/nfs/rpc_pipefs type rpc_pipefs (rw)
fusectl on /sys/fs/fuse/connections type fusectl (rw)
/dev/sdd1 on /media/disk type vfat (rw,nosuid,nodev,noatime,uhelper=hal,uid=500,utf8,shortname=lower)
twin4:/musicbox on /misc/musicbox type nfs4 (rw,addr=192.168.1.4)
```

列表格式如下：<u>设备</u>（*device*） on <u>加载点</u>（*mount_point*） type 文件系统类型（*file_system_type*）（选项 *option*）。如第一行，显示设备 `/dev/sda2` 被加载到文件系统的根分区，类型是 `ext4`，是可读可写的（选项为 `rw`）。列表中的最后两条很有意思。最后第二条显示一个 2GB 大小的 SD 存储卡被加载在 `/media/disk`，最后一条是一个被加载到 `/misc/musicbox` 的网络驱动器。

作为首次体验，我们会加载一个 CD-ROM。首先，我们看一下插入光盘前的系统：

```bash
[me@linuxbox ~]$ mount
/dev/mapper/VolGroup00-LogVol00 on / type ext4 (rw)
proc on /proc type proc (rw)
sysfs on /sys type sysfs (rw)
devpts on /dev/pts type devpts (rw,gid=5,mode=620)
/dev/sda1 on /boot type ext4 (rw)
tmpfs on /dev/shm type tmpfs (rw)
none on /proc/sys/fs/binfmt_misc type binfmt_misc (rw)
sunrpc on /var/lib/nfs/rpc_pipefs type rpc_pipefs (rw)
```

这是一个来自 CentOS 系统的列表，使用了 LVM（逻辑卷管理 Logical Volume Manager）创建其根文件系统。一如多数现代 Linux 发行版，系统将尝试自动加载 CD-ROM。当我们插入盘片之后，会看到：

```bash
[me@linuxbox ~]$ mount
/dev/mapper/VolGroup00-LogVol00 on / type ext4 (rw)
proc on /proc type proc (rw)
sysfs on /sys type sysfs (rw)
devpts on /dev/pts type devpts (rw,gid=5,mode=620)
/dev/hda1 on /boot type ext4 (rw)
tmpfs on /dev/shm type tmpfs (rw)
none on /proc/sys/fs/binfmt_misc type binfmt_misc (rw)
sunrpc on /var/lib/nfs/rpc_pipefs type rpc_pipefs (rw)
/dev/sdc on /media/live-1.0.10-8 type iso9660 (ro,noexec,nosuid,nodev,uid=500)
```

在插入盘片之后，我们可以看到和之前相同的列表下多了一条。在列表末，我们看到的是已经加载到 `/media/live-1.0.10-8` 的 CD-ROM，类型是 `iso9660`。出于试验的目的，我们感兴趣的是设备的名称。当你自己做这个试验的时候，设备名称很可能不同。

> **警告：**在下面的例子中，你需要特别关注的是，要使用你系统中实际的设备名称，**不要使用在这里出现的名称**，这点至关重要！
>
> 同时注意音频 CD不同于 CD-ROM。音频 CD不包含文件系统，所以不能以通常的意义加载。

我们现在已经有了 CD-ROM 的设备名称，让我们卸载这个盘片，然后重新加载到文件系统树中的另一个位置。要做到这点，我们要成为超级用户（使用适合我们系统的命令）然后用 `umount` 命令（注意拼写）卸载盘片。

```bash
[me@linuxbox ~]$ su -
Password:
[root@linuxbox ~]# umount /dev/sdc
```

下一步是为这个盘片创建一个新的<u>加载点</u>（*mount point*）。一个加载点就是在文件系统树中的一个简单的目录。没什么特别的，甚至不必是一个空目录。如果将设备加载到一个非空目录中，那么在加载期间你将看不到这个目录原有的内容。对我们而言，将创建一个新目录。

```bash
[root@linuxbox ~]# mkdir /mnt/cdrom
```

最后我们将 CD-ROM 加载到新的加载点。`-t` 选项用来指定文件系统类型。

```bash
[root@linuxbox ~]# mount -t iso9660 /dev/sdc /mnt/cdrom
```

随后，通过新加载点检查一下 CD-ROM 的内容：

```bash
[root@linuxbox ~]# cd /mnt/cdrom
[root@linuxbox cdrom]# ls
```

注意当我们卸载 CD-ROM 时会发生什么：

```bash
[root@linuxbox cdrom]# umount /dev/sdc
umount: /mnt/cdrom: device is busy
```

为什么是这样？理由是当设备被某个用户或某个进程使用时，不能卸载设备。本案中，我们将当前工作目录切换到了 CD-ROM 的加载点，就导致了设备处于工作中。我们可以轻易地修复这个问题，仅需将工作目录切换到其它目录即可：

```bash
[root@linuxbox cdrom]# cd
[root@linuxbox ~]# umount /dev/sdc
```

现在设备成功卸载了。

> **为何卸载很重要**
>
> 如果你看一下 `free` 命令的输出，就是内存使用率的统计信息，你会看到一个叫 `buffers` 的统计。计算机系统被设计为尽可能快的运行。影响系统速度的一个因素就是慢设备。打印机是一个很好的例子。即便是最快的打印机，在计算机标准中也是相当慢的。如果计算机不得不停下来等待打印机完成打印一页纸，它会变得非常慢。在早期 PC 时代（实现多任务之前），这是个真实的问题。如果你正在做一个电子表格或者一个文本文档。计算机会以打印机可以接受的速度将数据发送到打印机，但由于打印机打印速度不是很快，因此速度非常慢。这个问题随着<u>打印机缓冲区</u>（*printer buffer*）的来临而最终得以解决，一个在计算机和打印机之间的包含有 RAM 存储的设备。由于有了打印机缓冲区，计算机可以将输出到打印机的内容发送到缓冲区，缓冲区会将其存储到快速 RAM，所以计算机就不用等待打印而继续工作了。同时，打印机缓冲区会以打印机可以接受的速度从缓冲区内存中慢慢将数据假脱机到打印机。
>
> 缓冲的理念广泛应用在计算机，以使其更快速。不要让偶然地从慢设备中读写数据的需求阻碍系统的速度。操作系统在实际必须与较慢的设备交互之前，尽可能多地存储已经从存储器中读取并将被写入存储设备的数据。在 Linux 系统中，例如，你将注意到系统看起来比它所用掉的内存更多。这并不意味着 Linux 「用」完了所有的内存，这意味着 Linux 正在利用所有可用内存来尽可能多地进行缓冲。缓冲可以让写入存储设备变得非常快，因为写入物理设备被延期到了将来的某个时间。同时，要写入设备的数据在内存中堆积起来。最终，操作系统会把数据写入到物理设备中。
>
> 卸载设备意味着要将剩余的数据写入设备，以便将其安全移除。如果在移除前没有卸载，对该设备的延时的数据就不会被完全传输。在某些案例中，数据可能包含至关重要的目录更新，将会导致<u>文件系统损坏</u>（*file system corruption*），这是对于计算机而言最坏的事件之一。

### 确定设备名称

有时候，确定一个设备的名称是有些困难的。在以前，还不是很难。一个设备总是在同一个地方，也不会有什么变化。类 Unix 系统喜欢这种方式。当 Unix 发展之后，「改变一个磁盘驱动器」涉及到的是使用叉车去计算机房中移除一个洗衣机大小的设备。近年来，主要的桌面硬件配置变得非常动态，Linux 已经进化得比它的祖先们更灵活了。

在上面的例子中，我们利用现代 Linux 桌面自动加载了设备，之后再确定了它的名称。但是，如果我们正在管理服务器或其它不会发生这种情况的环境呢？该如何识别这个设备呢？

首先，看一下系统如何命名设备。如果我们列出 `/dev` 目录中的内容（所有设备都在那），我们可以看到许许多多的设备。

```bash
[me@linuxbox ~]$ ls /dev
```

内容列表揭示了一些设备命名的样式。表 15-2 给出了这些样式的一些轮廓。

表 15-2：Linux 存储设备名称

| 样式       | 设备                                                         |
| ---------- | ------------------------------------------------------------ |
| `/dev/fd*` | 软盘驱动器。                                                 |
| `/dev/hd*` | 老旧系统中的 IDE(PATA) 磁盘。典型的主板包含了两个 IDE 接口或<u>通道</u>（*channels*），每个都带有一个电缆，带有两个驱动器连接点。电缆上的第一个驱动叫<u>主</u>（*master*）设备，第二个叫<u>从</u>（*slave*）设备 。设备名按此排序，如 `/dev/hda` 指第一个通道的主设备，`/dev/hdb` 指第一个通道的从设备；`/dev/hdc` 指第二个通道的主设备，以此类推。尾随的数字是指设备上的分区编号。如 `/dev/hda1` 指系统上第一块硬盘上的第一个分区，而 `/dev/hda` 则指整个硬盘。 |
| `/dev/lp*` | 打印机。                                                     |
| `/dev/sd*` | SCSI 磁盘。在现代 Linux 系统上，内核将所有磁盘类的设备（包括 PATA/SATA 硬盘，闪存盘，如便携式音乐播放器和数字相机之类的大容量存储设备等）都当 SCSI 磁盘。其余的命名系统与老旧的 `/dev/hd*` 命名规则类似。 |
| `/dev/sr*` | 光学设备（CD/DVD 的读取和烧录设备）。                        |

另外，我们经常看到符号链接，如 `/dev/cdrom`、`/dev/dvd` 和 `/dev/floppy`，指向实际设备文件，以提供便利。

如果你工作在一个没有自动加载可移动设备的系统上，你可以使用下列技术来确定加载后的可移动设备的名称。首先，开启一个对 `/var/log/messages` 或 `/var/log/syslog` 文件的实时查看（可能需要超级管理员权限）。

```bash
[me@linuxbox ~]$ sudo tail -f /var/log/messages
```

会显示文件的最后几行，然后会暂停。然后插入可移动设备，在本例中，我们将用一个 16MB 的闪存。几乎即时地，内核注意到了设备并检测到它。

```bash
Jul 23 10:07:53 linuxbox kernel: usb 3-2: new full speed USB device using uhci_hcd and address 2
Jul 23 10:07:53 linuxbox kernel: usb 3-2: configuration #1 chosen from 1 choice
Jul 23 10:07:53 linuxbox kernel: scsi3 : SCSI emulation for USB Mass Storage devices
Jul 23 10:07:58 linuxbox kernel: scsi scan: INQUIRY result too short (5), using 36
Jul 23 10:07:58 linuxbox kernel: scsi 3:0:0:0: Direct-Access Easy Disk 1.00 PQ: 0 ANSI: 2
Jul 23 10:07:59 linuxbox kernel: sd 3:0:0:0: [sdb] 31263 512-byte hardware sectors (16 MB)
Jul 23 10:07:59 linuxbox kernel: sd 3:0:0:0: [sdb] Write Protect is off
Jul 23 10:07:59 linuxbox kernel: sd 3:0:0:0: [sdb] Assuming drive cache: write through
Jul 23 10:07:59 linuxbox kernel: sd 3:0:0:0: [sdb] 31263 512-byte hardware sectors (16 MB)
Jul 23 10:07:59 linuxbox kernel: sd 3:0:0:0: [sdb] Write Protect is off
Jul 23 10:07:59 linuxbox kernel: sd 3:0:0:0: [sdb] Assuming drive cache: write through
Jul 23 10:07:59 linuxbox kernel: sdb: sdb1
Jul 23 10:07:59 linuxbox kernel: sd 3:0:0:0: [sdb] Attached SCSI removable disk
Jul 23 10:07:59 linuxbox kernel: sd 3:0:0:0: Attached scsi generic sg3 type 0
```

显示后再次暂停，按 `Ctrl-c` 返回提示符。有趣的是输出中重复指向 [sdb] 的部分，匹配我们所期待的 SCSI 磁盘设备名。了解到这一点，这两行变得特别有启发性：

```bash
Jul 23 10:07:59 linuxbox kernel: sdb: sdb1
Jul 23 10:07:59 linuxbox kernel: sd 3:0:0:0: [sdb] Attached SCSI removable disk
```

这告诉我们整个设备的名称是 `/dev/sdb`，第一个分区是 `/dev/sdb1`。我们已经看到，用 Linux 工作充满了侦探工作的乐趣！

> **提示：**使用 `tail -f /var/log/messages` 技术可以实时观察系统正在干嘛。

有了设备名称，我们就可以加载这个闪存了。

```bash
[me@linuxbox ~]$ sudo mkdir /mnt/flash
[me@linuxbox ~]$ sudo mount /dev/sdb1 /mnt/flash
[me@linuxbox ~]$ df
Filesystem  1K-blocks        Used   Available  Use%  Mounted on
/dev/sda2    15115452     5186944     9775164   35%  /
/dev/sda5    59631908    31777376    24776480   57%  /home
/dev/sda1      147764       17277      122858   13%  /boot
tmpfs          776808           0      776808    0%  /dev/shm
/dev/sdb1       15560           0       15560    0%  /mnt/flash
```

如果设备一直保持物理连接且没有重启计算机，则该设备名将保持不变。

## 创建新的文件系统



### 用 fdisk 处理分区



### 用 mkfs 创建新文件系统



## 测试并修复文件系统



## 在设备之间直接移动数据



## 创建 CD-ROM 映像



### 从 CD-ROM 创建一个映像拷贝



### 从文件集创建一个映像



## 写入 CD-ROM 映像



### 直接加载 ISO 映像



### 清空一张可擦写 CD-ROM



### 写入一个映像



## 总结



## 扩展阅读



## 额外加分

