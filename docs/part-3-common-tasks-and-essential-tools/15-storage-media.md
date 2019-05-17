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

| 字段 | 内容         | 详述 |
| ---- | ------------ | ---- |
| 1    | 设备         |      |
| 2    | 加载点       |      |
| 3    | 文件系统类型 |      |
| 4    | 选项         |      |
| 5    | 频率         |      |
| 6    | 顺序         |      |



### 查看已加载的文件系统清单



### 确定设备名称



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

