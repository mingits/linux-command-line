# 6. 重定向

本章中，我们将释放可能是命令行中最佳的特性。<u>输入输出重定向</u>（*I/O redirection*）。「I/O」代表 *input/output*，伴随这个功能，我们能从文件输入命令，也能把命令的输出重定向到文件，我们可以将多个命令连接成一个命令<u>管道</u>（*pipelines*）。要展示这个功能，我们来介绍下面几个命令：

- `cat` 连接文件
- `sort` 对文本行排序
- `uniq` 报告或忽略重复的行
- `grep` 打印匹配字符模式的行
- `wc` 打印文件的行数、词数和字节数
- `head` 输出文件的开头部分
- `tail` 输出文件的结尾部分
- `tee` 从标准输入读取文件，写入标准输出和文件

## 标准输入，标准输出，错误

目前为止我们所接触到的很多程序都会产生某种输出。输出通常有两种类型组成：

- 程序的结果，即程序按设计所产生的数据
- 状态及错误信息，用以告知我们如何使用程序

如果我们看到 `ls` 命令，我们可以看到它在屏幕上显示结果和错误信息。

坚持 Unix 「一切皆文件」的主题，像 `ls` 这样的程序实际上将他们的计算结果送到了一个叫<u>标准输出</u>（*standard output* 经常写成 *stdout*）的特殊文件中，将他们的状态信息送到另一个叫<u>标准错误</u>（*standard error* 经常写成 *stderr*）的文件中。默认情况下，两者都被连接到屏幕而非磁盘文件。

另外，许多程序从<u>标准输入</u>（*standard input* 经常写成 *stdin*）的设备中取得输入，默认情况下，附加到键盘。

I/O 重定向允许我们改变输出的去向和输入的来源。正常情况下，输出到屏幕、输入自键盘，但运用重定向，我们能改变这种情况。

## 重定向标准输出

I/O 重定向允许我们重新定义标准输出的去向。要重定向标准输出到另一个文件而非屏幕，我们用 `>` 重定向操作符，并跟随文件名。为何我们要这样做呢？通常，存储一个命令的输出为一个文件非常有用。例如，我们告诉 shell，将 `ls` 命令的输出送到 `ls-output.txt` 文件而非屏幕中：

```bash
[me@linuxbox ~]$ ls -l /usr/bin > ls-output.txt
```

这里我们创建了一个 `/usr/bin` 目录中的文件清单，并将结果保存到了文件 `ls-output.txt` 中。让我们检查重定向命令：

```bash
[me@linuxbox ~]$ ls -l ls-output.txt
-rw-rw-r-- 1 me me 167878 2018-02-01 15:07 ls-output.txt
```

很好，一个很大的文本文件。如果我们用 `less` 查看文件，我们可以看到文件确实包含了之前 `ls` 命令的结果。

```bash
[me@linuxbox ~]$ less ls-output.txt
```

现在来重复我们的重定向测试，但是这次来点绕的，我们把目录名改成一个并不存在的目录：

```bash
[me@linuxbox ~]$ ls -l /bin/usr > ls-output.txt
ls: cannot access /bin/usr: No such file or directory
```

我们收到了一个错误信息。这是合理的，因为我们指定了一个不存在的目录 `/bin/usr`，但是为何错误信息是显示在屏幕上而非显示在重定向文件 `ls-output.txt` 中呢？答案是，`ls` 程序没有将错误信息传送到标准输出。类似大多数优秀的 Unix 程序，它把错误信息传送到了标准错误。我们随后将看到如何重定向标准错误，但是首先让我们看下输出文件：

```bash
[me@linuxbox ~]$ ls -l ls-output.txt
-rw-rw-r-- 1 me me 0 2018-02-01 15:08 ls-output.txt
```

文件现在是 0 字节长度了！这是因为我们用 `>` 重定向了输出，而目标文件总是从文件开头被重写。因为我们的 `ls` 命令没有生成结果，而仅仅得到一个错误信息，重定向操作开始重写文件，随即因错误而停止，导致文件被截断。实际上，如果我们需要截断一个文件（或创建一个新的空文件），我们可以用这样的技巧：

```bash
[me@linuxbox ~]$ > ls-output.txt
```

没有前置命令，简单的使用重定向操作符，将清空一个已存在的文件，或创建一个新的空文件。

那我们怎么能把重定向结果附加到现有文件而不是从文件开始处覆盖文件呢？我们用 `>>` 重定向操作符：

```bash
[me@linuxbox ~]$ ls -l /usr/bin >> ls-output.txt
```

使用 `>>` 操作符将结果附加到一个文件中。如果该文件不存在，将直接创建文件。来测试一下：

```bash
[me@linuxbox ~]$ ls -l /usr/bin >> ls-output.txt
[me@linuxbox ~]$ ls -l /usr/bin >> ls-output.txt
[me@linuxbox ~]$ ls -l /usr/bin >> ls-output.txt
[me@linuxbox ~]$ ls -l ls-output.txt
-rw-rw-r-- 1 me me 503634 2018-02-01 15:45 ls-output.txt
```

我们重复了三次命令，导致输出文件是一次输出文件的三倍大小。

## 重定向标准错误

重定向标准错误没有一个专用的重定向操作符。要重定向标准错误，我们必须要参考其<u>文件描述符</u>（*file descriptor*）。一个程序可以在几个编号的文件流中的任何一个上产生输出。我们所说的标准输入、标准输出、标准错误，是这些文件流的前三个，shell 内部分别将它们的文件描述符编号定为 0，1，2。Shell 使用文件描述符编号为重定向文件提供一个记号。因为标准错误的文件描述符编号为 2，我们就可以用这个记号重定向标准错误：

```bash
[me@linuxbox ~]$ ls -l /bin/usr 2> ls-error.txt
```

文件描述符「2」放在重定向操作符之前，以便将标准错误重定向到文件 `ls-error.txt`。

### 重定向标准输出和标准错误到一个文件中

我们可能想要把一个命令的输出捕捉到一个单个文件中。这样，我们必须同时重定向标准输出和标准错误。有两种方法可以做到。这里展示一个传统方法，用于老版本的 shell：

```bash
[me@linuxbox ~]$ ls -l /bin/usr > ls-output.txt 2>&1
```

我们用这种方式执行两个重定向。首先重定向标准输出到 `ls-output.txt` 文件，然后用 `2>&1` 重定向文件描述符 2（即标准错误）到文件描述符 1（标准输出）。

> **注意：重定向的顺序很重要。**标准错误的重定向必须总是在重定向标准输出<u>之后</u>发生，否则无效。下面的例子将重定向标准错误到 `ls-output.txt` 文件：
>
> ```bash
> >ls-output.txt 2>&1
> ```
>
> 但是，如果顺序改成下面这样，标准错误就直接显示到屏幕了。
>
> ```bash
> 2>&1 >ls-output.txt
> ```

最近版本的 `bash` 提供第二种方法，更精简地执行这个重定向：

```bash
[me@linuxbox ~]$ ls -l /bin/usr &> ls-output.txt
```

上例中，我们用 `&>` 重定向标准输出和标准错误两者到 `ls-output.txt` 文件中。我们还可以附加标准输出和标准错误文件到一个文件中：

```bash
[me@linuxbox ~]$ ls -l /bin/usr &>> ls-output.txt
```

### 处理不想得到的输出

所谓「沉默是金」，有时候我们不想从一个命令中输出什么，而只是想抛开它。这要用到特别的错误和状态信息。系统提供了一个叫 `/dev/null` 的特殊文件来重定向输出。这是个被称为<u>比特桶</u>（*bit bucket*）的系统设备，用以接收输入且对该输入不作任何处理。这样做：

```bash
[me@linuxbox ~]$ ls -l /bin/usr 2> /dev/null
```

> **在 Unix 文化中的 /dev/null **
>
> 比特桶是一个古老的 Unix 概念，因为其普遍性，在 Unix 文化的各个方面都有所体现。现在你应该知道，如果有人说，他把你的评论送到 `/dev/null` 去了的含义了吧。更多案例，可以参考维基百科的 `/dev/null` 条目：http://en.wikipedia.org/wiki//dev/null

## 重定向标准输入

目前为止，我们没遇到要用到标准输入的命令（实际上有过，但是我们要晚点来透露这个惊喜），所以，来介绍一个。

### cat - 连接文件

`cat` 命令读取一个或多个文件，然后复制到标准输出：

```bash
cat [file...]
```

大多数情况下，我们把 `cat` 类比做 DOS 中的 `TYPE` 命令。可以用来不分页的显示文件。例如下面的命令将显示 `ls-output.txt` 的文件内容：

```bash
[me@linuxbox ~]$ cat ls-output.txt
```

`cat` 经常用来显示短小的文本文件。因为 `cat` 命令能接收多个文件作为参数，它也能用来将文件连接在一起。我们下载了一个被分割成多个部分的文件（Usenet 上的多媒体文件通常会被如此分割），我们需要将这些被分割的文件重新合并起来，如果文件是这样命名的：

`movie.mpeg.001` `movie.mpeg.002` `movie.mpeg.099`

我们就能用下面的命令将它们合并起来：

```bash
cat movie.mpeg.0* > movie.mpeg
```

由于通配符总是按顺序扩展，所以这个参数能按正确的顺序排列文件。

都挺好的。但是跟标准输入有啥关系？现在还没有，但让我们试试别的。如果我们不带任何参数输入 `cat`，会发生什么？

```bash
[me@linuxbox ~]$ cat
```

什么都没发生，不过光标在那挂着。它可能看起来像这样，但它确实正在做它应该做的事情。

如果不给任何参数，`cat` 将从标准输入中读取，默认情况下是等待我们从键盘输入。试着输入些文本然后按 `Enter`：

```bash
[me@linuxbox ~]$ cat
The quick brown fox jumped over the lazy dog.
```

然后按 `Ctrl-d`（按住 Ctrl 键，然后按 d），告诉 `cat` 标准输入已经到了<u>文件结束</u>（EOF *end of file*）。

```bash
[me@linuxbox ~]$ cat
The quick brown fox jumped over the lazy dog.
The quick brown fox jumped over the lazy dog.
```

由于缺少文件名参数，`cat` 将标准输入复制到了标准输出，所以我们看到我们的文本重复出现一行。我们能用这种行为创建短小的文本文件。来用刚才的文本创建一个名为 `lazy_dog.txt` 的文件。

```bash
[me@linuxbox ~]$ cat > lazy_dog.txt
The quick brown fox jumped over the lazy dog.
```

命令后紧跟着我们所需的文件，记住用 `Ctrl-d` 结束。我们用这个命令行实现了世界上最笨的文字处理！用 `cat` 再次将文件复制到标准输出：

```bash
[me@linuxbox ~]$ cat lazy_dog.txt
The quick brown fox jumped over the lazy dog.
```

既然我们知道 `cat` 如何接收标准输入，除了文件名参数外，来尝试一下重定向标准输入。

```bash
[me@linuxbox ~]$ cat < lazy_dog.txt
The quick brown fox jumped over the lazy dog.
```

使用 `<` 重定向操作符，我们把标准输入的来源从键盘改变成 `lazy_dog.txt` 文件。我们看到结果与仅仅传递一个文件名参数是相同的。与传递文件名参数相比，这不是特别有用，但它用于演示使用文件作为标准输入的来源。 我们很快就会看到其他命令可以更好地利用标准输入。

在继续我们的课程前，为 `cat` 检查一下手册页，会发现几个有趣的选项。

## 管道

<u>管道</u>（*pipelines*）是 shell 提供的一个利器，用以读取一个命令的标准输入，并发送到标准输出。使用管道操作符 `|`（竖条符），一个命令的标准输出能通过管道成为另一个命令的标准输入。

```bash
command1 | command2
```

要完整的演示这个，我们需要几个命令。还记得我们说过已经学过一个用来接收标准输入的命令了吗？那就是 `less`。我们用 `less` 分页显示任何命令的标准输出：

```bash
[me@linuxbox ~]$ ls -l /usr/bin | less
```

简直太方便了！使用这个技术，我们能方便的检查任何命令所产生的输出。

> **`>` 和 `|` 的区别**
>
> 初看之下，很难理解管道符号（`|`）和重定向符号（`>`）所执行的重定向的区别。简单地说，重定向操作符以一个文件连接一个命令，而管道符号将第一个命令的输出用作第二个命令的输入。
>
> ```bash
> command1 > file1
> command1 | command2
> ```
>
> 很多人在学习管道的时候想尝试下面这个操作，让我们来看看会发生什么：
>
> ```bash
> command1 > command2
> ```
>
> 答案是：有时候会相当糟糕。
>
> 这里有个 Linux 系统管理员在管理服务器应用时提交的一个实例。作为一个超级用户，他输入了：
>
> ```bash
> # cd /usr/bin
> # ls > less
> ```
>
> 第一个命令让他进入存放着大量程序的目录，第二个命令告诉 shell 用 `ls` 命令的输出覆盖 `less` 命令。因为 `/usr/bin` 目录已经存在一个名为 `less` 的文件（`less` 程序），所以第二个命令用 `ls` 命令所产生的文本覆盖了 `less` 程序就这样，毁掉了他系统中的 `less` 程序。
>
> 这个教训是，重定向操作符会静默地创建或覆盖文件，所以操作时需要万分谨慎地对待它。

### 过滤器

管道经常用来执行复杂数据的操作。可以将多个命令放在一个管道中。用这种方式使用的命令，也经常被成为<u>过滤器</u>（*filters*）。过滤器取得输入，做了某些改变，然后再输出。首先我们来试一下 `sort`。想象一下，我们要合并 `/bin` 和 `/usr/bin` 目录中的全部可执行程序的清单，并在结果中按序排列：

```bash
[me@linuxbox ~]$ ls /bin /usr/bin | sort | less
```

因为我们指定了两个目录（`/bin` 和 `/usr/bin`），`ls` 输出的是由两个各自已经排序好的列表所组成的。由于在管道中加入了 `sort`，我们将数据改变成了一份排序好的列表。

### uniq - 报告或忽略重复行

`uniq` 命令经常和 `sort` 连在一起用。`uniq` 接收一个完成排序的数据清单，并在缺省情况下移除重复项。所以，要确保我们的列表中没有重复项（即在 `/bin` 和 `/usr/bin` 两个目录中有相同文件名的程序），我们要把 `uniq` 加到我们的管道中。

```bash
[me@linuxbox ~]$ ls /bin /usr/bin | sort | uniq | less
```

本例中，我们用 `uniq` 移除了 `sort` 命令所输出的全部重复项。如果我们想看的是重复项，我们需要给 `uniq` 加一个 `-d` 选项：

```bash
[me@linuxbox ~]$ ls /bin /usr/bin | sort | uniq -d | less
```

### wc - 打印行数、字数和字节数

`wc` 命令（word count）用来显示文件中的行数、字数和字节数。例如：

```bash
[me@linuxbox ~]$ wc ls-output.txt
 7902 64566 503634 ls-output.txt
```

本例中，打印了 `ls-output.txt` 文件的行数，字数和字节数三个数字。如我们之前的命令，如果执行的是不带任何参数的命令，`wc` 接收标准输入。`-l` 选项限制其输出为仅显示行数。把它加到管道中，能很方便的计数。来一下我们排序好的列表的项目数：

```bash
[me@linuxbox ~]$ ls /bin /usr/bin | sort | uniq | wc -l
2728
```

### grep - 显示符合模式的行

`grep` 是一个非常强大的用来查找文件中文本模式的程序。用法如下：

```bash
grep pattern [file...]
```

当 `grep` 遭遇一个文件中的「模式」，会打印出包含该模式的行。`grep` 能匹配非常复杂的模式，但是现在我们将集中在简单的文本匹配。我们将在第 19 章「正则表达式」介绍高级模式。

现在我们想要找出程序清单中所有包含 `zip` 的文件。这样的检索，让我们想到的是，系统中某些文件压缩程序。我们这样做：

```bash
[me@linuxbox ~]$ ls /bin /usr/bin | sort | uniq | grep zip
bunzip2
bzip2
gunzip
gzip
unzip
zip
```

有一些很方便的选项：

- `-i` 导致 `grep` 忽略大小写（正常情况下是大小写敏感的）
- `-v` 告诉 `grep` 仅打印不匹配模式的行。

### head/tail - 打印文件的头/尾部分

有时我们不想要看到一个命令的全部输出，而仅需要看到文件的头部或尾部的若干行。`head` 命令显示一个文件的前十行，`tail` 命令显示一个文件的末十行。默认情况下，两个命令都会显示十行，但是可以通过 `-n` 选项调整。

```bash
[me@linuxbox ~]$ head -n 5 ls-output.txt
total 343496
-rwxr-xr-x 1 root root 31316 2007-12-05 08:58 [
-rwxr-xr-x 1 root root 8240 2007-12-09 13:39 411toppm
-rwxr-xr-x 1 root root 111276 2007-11-26 14:27 a2p
-rwxr-xr-x 1 root root 25368 2006-10-06 20:16 a52dec
[me@linuxbox ~]$ tail -n 5 ls-output.txt
-rwxr-xr-x 1 root root 5234 2007-06-27 10:56 znew
-rwxr-xr-x 1 root root 691 2005-09-10 04:21 zonetab2pot.py
-rw-r--r-- 1 root root 930 2007-11-01 12:23 zonetab2pot.pyc
-rw-r--r-- 1 root root 930 2007-11-01 12:23 zonetab2pot.pyo
lrwxrwxrwx 1 root root 6 2016-01-31 05:22 zsoelim -> soelim
```

这些也能用在管道中：

```bash
[me@linuxbox ~]$ ls /usr/bin | tail -n 5
znew
zonetab2pot.py
zonetab2pot.pyc
zonetab2pot.pyo
zsoelim
```

`tail` 有一个选项，允许我们实时查看文件，用来观察正在被写入的日志文件时，这个选项很有用。下例中，我们能看到在 `/var/log/messages` 文件（或者 `/var/log/syslog` 文件）。在某些发行版中，这个操作需要超级用户权限，因为 `/var/log/messages` 文件可能包含安全信息：

```bash
[me@linuxbox ~]$ tail -f /var/log/messages
Feb 8 13:40:05 twin4 dhclient: DHCPACK from 192.168.1.1
Feb 8 13:40:05 twin4 dhclient: bound to 192.168.1.4 -- renewal in
1652 seconds.
Feb 8 13:55:32 twin4 mountd[3953]: /var/NFSv4/musicbox exported to
both 192.168.1.0/24 and twin7.localdomain in
192.168.1.0/24,twin7.localdomain
```

使用 `-f` 选项，`tail` 持续监控文件，当新行被添加进来后，会立刻显示出来，直到我们按 `ctrl-c` 停止。

### tee - 从标准输入读取，输出到标准输出和文件

与管道的隐喻相匹配，Linux 提供一个 `tee` 的命令创造了一个「三通」来适配我们的「管道」。`tee` 程序读取标准输入，复制到标准输出（允许数据继续在管道中）和一个或多个文件。对于捕捉即时处理中的管道中的内容会很有用。来重复一个之前的示例，这次用 `tee` 来捕捉整个目录