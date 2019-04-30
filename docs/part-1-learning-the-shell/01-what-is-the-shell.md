# 1. 什么是 Shell

当我们说起「命令行」时，我们实际上是指称 *shell* （此单词不翻译）。Shell 是一个程序，负责将通过键盘输入的命令带入操作系统并执行命令。几乎所有 Linux 发行版都会提供一个名叫 *bash* 的 GNU 项目。*bash* 是「**B**ourne **A**gain **SH**ell」的缩略语，是原生 Unix shell 程序 *sh* （由 Steve Bourne 所写）的增强替代品。

## 终端模拟器

当我们使用图形用户界面的时候，我们需要一个叫作「终端模拟器」的程序来跟 shell 交互。如

- KDE 中的 Konsole
- GNOME 中的 gnome-terminal

此外还有许多供 Linux 使用的终端模拟器，它们基本上只做相同的一件事：将我们接入 shell。

## 第一次击键

所以，让我们开始吧，运行终端模拟器。当它启动后，我们可以看到类似这样的界面：

```bash
[me@linuxbox ~]$
```

这被称为<u>命令提示符</u>（*shell prompt*），无论何时，当 shell 准备接受输入的状态下，命令提示符就会显示在终端模拟器中，它也会有依赖于各种发行版的各种显示变化，不过通常会包含 `username@machinename` （用户名@机器名）紧跟着的是当前的工作目录（`~`），还有一个美元符号（`$`） 。

注意：如果最后一个字符是英镑符号（`#`）而非美元符号（`$`），说明当前终端会话具有<u>超级管理员权限</u>（superuser privileges），这意味着我们是以<u>根用户</u>（root）登录或者我们选择了一个具有超级管理员权限的终端模拟器。

假设到目前为止，一切状况正常，让我们在命令行中随便写一些东西：

```bash
[me@linuxbox ~]$ kaekfjaeifj
```

shell 会告诉我们这个命令毫无意义，并且会给我们再次输入的机会：

```bash
bash: kaekfjaeifj: command not found
[me@linuxbox ~]$
```

### 命令历史

当我们键入向上箭头键（↑），我们可以看到上一次输入的命令「`kaekfjaeifj`」再次出现在提示符之后，这就是<u>命令历史</u>（*command history*），大多数 Linux 发行版会记住之前输入的 500 条命令。当键入向下箭头键（↓）时，之前的命令就消失了。

### 光标的移动

按下向上箭头键，重新调出之前的命令，现在尝试使用左右箭头键，知道我们该如何把光标定位到命令行中的任意位置了吧，这样会使我们更方便地编辑命令。

> **关于鼠标和焦点的二三言**
>
> 除了键盘之外，在终端模拟器中你也可以使用鼠标。这是 X Window 系统（图形用户界面的底层引擎）的内建机制，以便支持更快捷的复制和粘贴技术。如果你按住鼠标左键并拖动它以高亮选中某些文本，这些文本将会复制到 X 系统提供的缓存中，按下鼠标中键会将这些文本粘贴到当前光标位置。试一试吧。
>
> **注意**：不要尝试使用 Ctrl-c 和 Ctrl-v 在终端窗口中复制粘贴，不会有效果的。早于 Microsoft Windows 多年，这些组合键就已经在 shell 中被分配给了其它意义。
>
> 你的图形桌面环境（大多数是 KDE 或者 GNOME）为了努力表现的像 Windows，或许会采取「点击聚焦」（click to focus）的聚焦策略（*focus policy*）。这意味着你需要点击一个窗口才能使其得到聚焦（即被激活）。这与传统的 X 窗口行为「焦点跟随鼠标」（即只需要将鼠标滑过窗口，就能使其得到聚焦）是相违背的。窗口在被点击之前是不会转到前台的，但是它还是能接受输入。将聚焦策略设置为「焦点跟随鼠标」会使得复制粘贴技术更有用。如果你可以的话，尝试一下（有些桌面环境，例如 ubuntu 的 unity 已经不再支持这一策略了）。我觉得如果你试过之后，应该为更喜欢这种策略，你可以在窗口管理的配置程序中找到这个设置。

## 尝试一些简单的命令

既然我们已经知道如何键入命令，让我们尝试一些简单的命令。第一个命令 `date`。这个命令显示当前的日期和时间。

```bash
[me@linuxbox ~]$ date
Thu Oct 25 13:51:54 EDT 2007
```

与之相关的一个命令是 `cal`，默认情况下会显示当前月份的日历。

```bash
[me@linuxbox ~]$ cal
October 2007
Su Mo Tu We Th Fr Sa
    1  2  3  4  5  6
 7  8  9 10 11 12 13
14 15 16 17 18 19 20
21 22 23 24 25 26 27
28 29 30 31
```

想要了解当前磁盘驱动器的可用空间，键入 `df` 命令：

```bash
[me@linuxbox ~]$ df
Filesystem  1K-blocks     Used  Available  Use%  Mounted on
/dev/sda2    15115452  5012392    9949716   34%  /
/dev/sda5    59631908 26545424   30008432   47%  /home
/dev/sda1      147764    17370     122765   13%  /boot
tmpfs          256856        0     256856    0%  /dev/shm
```

同样，要显示当前可用内存容量，键入 `free` 命令：

```bash
[me@linuxbox ~]$ free
         total       used    free  shared  buffers  cached
Mem:    513712     503976    9736       0     5312  122916
-/+ buffers/cache: 375748  137964
Swap:  1052248     104712  947536
```

## 结束终端会话

要结束一个终端会话，我们可以关闭终端模拟器或者在 shell 提示符后键入 `exit` 命令：

```bash
[me@linuxbox ~]$ exit
```

> **幕后控制台**
>
> 即便我们没有运行终端模拟器，还是会有几个终端会话持续运行在图形桌面之后，那就是<u>虚拟终端</u>（*virtual terminals*）或<u>虚拟控制台</u>（*virtual consoles*），在多数 Linux 发行版中，这些会话可以通过按键 Ctrl-Alt-F1 到 Ctrl-Alt-F6 接入。当接入到一个会话后，它会提供一个登录提示符，让我们键入用户名和密码。可以通过按键 Alt-F1 到 Alt-F6 来切换虚拟控制台。要回到图形桌面，请按 Alt-F7。

## 总结

作为本书的起始章节，我们接触到了 shell 并第一次看到了命令行，学习了如何开启并结束一个终端会话。我们也了解到如何提交一下简单的命令，并执行一些小的命令行内编辑。看起来不怎么可怕是吧？

## 扩展阅读

- 如需更深入了解 Steve Bourne 和 Bourne Shell，请访问维基百科 [https://en.wikipedia.org/wiki/Steve_Bourne](https://en.wikipedia.org/wiki/Steve_Bourne)
- 这里是一篇关于 shells 的文章 [https://en.wikipedia.org/wiki/Shell_(computing)](https://en.wikipedia.org/wiki/Shell_(computing))