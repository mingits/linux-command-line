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

现在我们要对闪存盘用 Linux 原生系统重新格式化，不要用现在的 FAT32 系统了。这涉及到两个步骤。

1. （可选的）创建一个新的分区布局，如果现存的布局不合适的话。
2. 在磁盘上创建一个新的空白的文件系统。

> **警告！**在下面的练习中，我们将格式化一个闪存盘。请用一个不含重要内容的盘，因为格式化将清除所有文件！还有，**确保你所指定的设备名在你的系统中是正确的，不一定和下面的文本中保持一致。忽略此警告将导致你格式化（擦除）错误的驱动器！**

### 用 fdisk 处理分区

`fdisk` 是许多能让我们直接和磁盘类设备（如硬盘和闪存盘）在低级别进行交互的（命令行和图形界面的）程序之一。 我们可以通过这个工具在设备上编辑、删除和创建新分区。要在闪存盘上工作，首先要卸载它（如果必要的话），然后调用 `fdisk` 程序：

```bash
[me@linuxbox ~]$ sudo umount /dev/sdb1
[me@linuxbox ~]$ sudo fdisk /dev/sdb
```

注意，我们所指定的必须是整个设备，而非某个分区。当程序启动后，会看到下列提示符：

```bash
Command (m for help):
```

键入 `m` 将显示程序菜单。

```bash
Command action
  a    toggle a bootable flag
  b    edit bsd disklabel
  c    toggle the dos compatibility flag
  d    delete a partition
  l    list known partition types
  m    print this menu
  n    add a new partition
  o    create a new empty DOS partition table
  p    print the partition table
  q    quit without saving changes
  s    create a new empty Sun disklabel
  t    change a partition's system id
  u    change display/entry units
  v    verify the partition table
  w    write table to disk and exit
  x    extra functionality (experts only)

Command (m for help):
```

首先要做的是检查现存的分区布局，我们键入一个 `p` 来打印设备的分区表。

```bash
Command (m for help): p

Disk /dev/sdb: 16 MB, 16006656 bytes
1 heads, 31 sectors/track, 1008 cylinders
Units = cylinders of 31 * 512 = 15872 bytes

Device Boot    Start    End   Blocks    Id    System
/dev/sdb1          2   1008   15608+     b    W95 FAT32
```

本例中，我们看到一个 16MB 的设备，仅有一个分区 (1)，使用了 1008 个磁盘柱面中的 1006 个，被识别为 Windows 95 的 FAT32 分区。有些程序会以这个识别来限制操作的种类，不过多数时候改变它并不重要。这次演示的重点，我们要将它改变为一个 Linux 分区。为此，必须首先找到 Linux 分区所用的标识符。在之前的列表中，我们看到 `ID b` 用来指定现有分区。要查看可用的分区类型列表，我们看一下程序菜单。可以看到下列选项：

```bash
l    list known partition types
```

若在提示符中键入 `l`，会显示很长的一个列表。其中，我们看到 `b` 是现有分区类型，而 `83` 是 Linux 分区类型。

回到菜单，我们看到该选项用来改变分区的 ID：

```bash
t    change a partition's system id
```

我们键入 `t` 并输入新的 ID：

```bash
Command (m for help): t
Selected partition 1
Hex code (type L to list codes): 83
Changed system type of partition 1 to 83 (Linux)
```

这样，我们所要的变更都已经完成了。目前为止，设备还不受影响（所有的变更都存储在内存中，而非在物理设备上），所以我们要将变更的分区表写入设备，然后退出。我们在提示符处输入 `w`。

```bash
Command (m for help): w
The partition table has been altered!
Calling ioctl() to re-read partition table.
WARNING: If you have created or modified any DOS 6.x partitions, please see the fdisk manual page for additional information.
Syncing disks.
[me@linuxbox ~]$
```

如果我们想不对设备做变更而退出，可以在提示符中输入 `q`，以便不写入变更而退出。可以直接忽略那些听起来很可怕的警告信息。

### 用 mkfs 创建新文件系统

我们已经编辑了分区表（虽然它可能是轻量级的），是时候在闪存盘上创建新的文件系统了。我们用 `mkfs`（即 **m**a**k**e **f**ile **s**ystem 的缩写）命令，可以用多种格式创建文件系统。要创建一个 ext4 文件系统，我们用 `-t` 选项来指定 ext4 系统类型，后面跟着要创建该格式分区的设备名。

```bash
[me@linuxbox ~]$ sudo mkfs -t ext4 /dev/sdb1
mke2fs 2.23.2 (12-Jul-2011)
Filesystem label=
OS type: Linux
Block size=1024 (log=0)
Fragment size=1024 (log=0)
3904 inodes, 15608 blocks
780 blocks (5.00%) reserved for the super user
First data block=1
Maximum filesystem blocks=15990784
2 block groups
8192 blocks per group, 8192 fragments per group
1952 inodes per group
Superblock backups stored on blocks:
    8193

Writing inode tables: done
Creating journal (1024 blocks): done
Writing superblocks and filesystem accounting information: done
This filesystem will be automatically checked every 34 mounts or
180 days, whichever comes first. Use tune2fs -c or -i to override.
[me@linuxbox ~]$
```

当选择 ext4 作为文件系统时，程序会显示一堆信息。要将其重新格式化为 FAT32 文件系统，则需要指定 `vfat` 作为文件系统类型。

```bash
[me@linuxbox ~]$ sudo mkfs -t vfat /dev/sdb1
```

当附加的存储设备被加载到系统中时，随时可以开始分区和格式化进程。我们已经操作了一个小小的闪存盘。同样的，也可以对内部硬盘和其他可移动存储设备作相同的操作。

## 测试并修复文件系统

在早先关于 `/etc/fstab` 文件的讨论中，我们看到在一些每行末尾都有些神秘的数字。每次系统启动时，在加载文件系统前会例行检查其完整性。该操作由 `fsck` 程序完成（**f**ile **s**ystem **c**hec**k** 的缩写）。每条 `fstab` 末的数字指示了检查设备的顺序。在前例中，我们看到首先检查的是根文件系统，然后是 `home` 和 `boot` 文件系统。末尾数字为零的设备不在例行检查之列。

在检查系统完整性之外，`fsck` 还可以根据损坏程度修复损坏的文件系统，并取得不同程度的成功。在类 Unix 系统中，文件的恢复部分被安置在每个文件系统的根目录下的 `lost+fount` 目录中。

要检查我们的闪存盘（首先要卸载它），我们可以按下面的命令来：

```bash
[me@linuxbox ~]$ sudo fsck /dev/sdb1
fsck 1.40.8 (13-Mar-2016)
e2fsck 1.40.8 (13-Mar-2016)
/dev/sdb1: clean, 11/3904 files, 1661/15608 blocks
```

现在，除非硬件问题（如磁盘损坏），文件系统损坏是相当少见了。在多数系统中，启动时检查文件系统错误会导致系统停止，并在继续运行前引导你运行 `fsck`。

> **fsck 什么？**
>
> 在 Unix 文件中，*fsck* 这个词经常用来替代包含这三个字母的常用词。这是特别合适的，因为如果你发现自己处于被迫运行 `fsck` 的情况，你可能会说出上述单词。

## 直接在设备之间移动数据

我们通常认为计算机中的数据被组织为众多文件，不过它也可以被认为是「原始」形式。如果我们查看一个磁盘驱动，例如，我们看到它由大量的数据「簇」组成，而操作系统则将其看成一个个文件和目录。无论如何，如果我妈吧一个磁盘简单地看作一个大型的数据簇的集合，就可以执行一些有用的任务，如克隆设备。

`dd` 程序执行这个克隆设备的任务。它会将数据簇从一个地方复制到另一个地方。（由于历史原因）它使用一个单一的句法，通常这样用：

```bash
dd if=input_file of=output_file [bs=block_size [count=blocks]]
```

> **警告！**`dd` 命令功能强大。尽管它的得名源自「data definition 数据定义」，有时候也会由于用户错误地输入 `if` 或 `of` 指定项而被称为「destroy disk 毁灭磁盘」。**永远要在按回车键之前再三检查输入输出的指定项！**

假设我们有两块相同容量的 USB 闪存盘，且要完整准确地将第一块盘中的数据复制到第二个之中。如果我们将两个设备都加载到计算机中，它们分别被分配为 `/dev/sdb` 和 `/dev/sdc`，就可以用下列命令将第一块磁盘上的所有东西复制到第二块盘中：

```bash
dd if=/dev/sdb of=/dev/sdc
```

作为替代方案，如果计算机仅加载了第一个设备，我们可以将其内容复制到一个普通文件，以供稍后恢复或拷贝。

```bash
dd if=/dev/sdb of=flash_drive.img
```

## 创建 CD-ROM 映像

烧录一张可写 CD-ROM（CD-R 或 CD-RW）包含两步：

1. 构建一个 <u>iso 映像文件</u>（*iso image file*），即 CD-ROM 的文件系统映像。
2. 将映像文件写入 CD-ROM 媒体。

### 从 CD-ROM 创建一个映像拷贝

如果要从一张现有的 CD-ROM 制作一个 ISO 映像，可以用 `dd` 来读取 CD-ROM 的数据簇，并将其复制为一个本地文件。假设我们有一张 Ubuntu CD，我们想要制作一个 ISO 文件以便日后用来制作更多的拷贝件。在插入 CD 并确定其设备名（我们假定其为 `/dev/cdrom`）之后，就可以制作 iso 文件了：

```bash
dd if=/dev/cdrom of=ubuntu.iso
```

此项技术对数据 DVD 盘同样有效，但是对音频 CD 无效，因为音频 CD 没有使用文件系统来存储。对音频 CD，要用 `cdrdao` 命令。

### 从文件集创建一个映像

要创建一个包含一个目录内容的 ISO 映像文件，要用 `genisoimage` 程序。首先，创建一个目录，包含所有我们想要放到映像文件中的文件，然后执行 `genisoimage` 命令创建映像文件。例如，如果我们已经创建了一个名为 `~/cd-rom-files` 的目录，并已经填充了为 CD-ROM 所需的文件，可以用下列命令创建一个名为 `cd-rom.iso` 的映像文件：

```bash
genisoimage -o cd-rom.iso -R -J ~/cd-rom-files
```

`-R` 选项为 Rock Ridge 扩展添加元数据，以便允许使用长文件名和 POSIX 风格的文件许可。同样的，`-J` 选项激活了 Joliet 扩展，以便在 Windows 系统中允许长文件名。

> **任何其他名称的程序……**
>
> 如果你看了创建和烧录光学介质（如 CD DVD）的在线教程，会经常遇见两个程序，`mkisofs` 和 `cdrecord`。这些程序是一个名为 `cdrtools` 的流行程序包的组件，作者是 Jörg Schilling。在 2006 年夏天，Schilling 先生变更了部分 `cdrtools` 程序包的授权许可，在 Linux 社区中创建了一个与 GNU GPL 不兼容的许可证。结果是，启动了 `cdrtools` 项目的一个分支，现在包括了 `cdrecord` 和 `mkisofs` 程序的替代品，分别是 `wodim` 和 `genisoimage`。

## 写入 CD-ROM 映像

当我们有了一个映像文件之后，就可以烧录到光学介质上了。下面要讨论的大多数程序，都可以用到 CD-ROM 和 DVD 两种介质上。

### 直接挂载 ISO 映像

有一个技巧，当 ISO 映像文件还存在硬盘上时，就可以如已经烧录到光学介质上一般挂载它。通过 `-o loop` 选项挂载（同时也需要 `-t iso9660` 文件系统类型选项），我们可以像挂载一个设备一样挂载映像文件，将其附加到文件系统树中。

```bash
mkdir /mnt/iso_image
mount -t iso9660 -o loop image.iso /mnt/iso_image
```

上例中，我们创建一个名为 `/mnt/iso_image` 的挂载点并挂载了 `image.iso` 的映像文件。当映像被挂载后，可以被看作在真实的 CD-ROM 或 DVD。<u>记得当不再需要的时候卸载映像文件</u>。

### 清空一张可擦写 CD-ROM

可擦写 CD-RW 介质在重用前需要被擦除或<u>清空</u>（*blanked*）。要做到这点，要用 `wodim`，可以指定要写入的设备名和清空操作的类型。`wodim` 程序提供了几种类型，最小（也最快）的是 `fast` 类型。

```bash
wodim dev=/dev/cdrw blank=fast
```

### 写入一个映像

要写入一个映像，我们还是会用到 `wodim`，指定光学介质写入设备和映像文件的名称。

```bash
wodim dev=/dev/cdrw image.iso
```

除了设备名和映像文件名，`wodim` 还支持一大堆选项。两个常用的选项，一个是 `-v`，供详细的输出，一个是 `-dao`，以便在<u>光盘一次刻录</u>（*disc-at-once*）模式中写盘。如果你准备用于商业复制的光盘，则应使用此模式。`wodim` 的默认模式是<u>轨道一次刻录</u>（*track-at-once*），在录制音频轨道时很有用。

## 总结

本章中我们学习了基本的存储管理任务。当然还有更多。Linux 支持广大的存储设备和文件系统方案。还提供了与其它系统的可交互操作性的众多特性。

## 扩展阅读

阅读一下我们已经学过的命令的手册页。其中的一些命令支持大量的选项和操作。同时，查找一些加载硬盘和光盘的在线教程。

## 额外加分

验证已经下载的 ISO 文件的完整性，通常很有用。