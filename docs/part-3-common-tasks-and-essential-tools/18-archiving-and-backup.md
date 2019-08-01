# 18. 存档和备份

计算机系统管理员的一个主要任务之一，是保持系统的数据安全。完成这项任务的一个途径是定时执行系统文件的备份。即便你不是一个系统管理员，在设备之间制作拷贝和移动巨大的文件集，也是很有用的。

本章中，我们将看到几个用来管理文件集的常用程序。它们是文件压缩程序：

- `gzip` 压缩或扩展文件
- `bzip2` 块排序文件压缩器

还有存档程序：

- `tar` 磁带存档工具
- `zip` 大包和压缩文件

和文件同步程序：

- `rsync` 远程文件和目录同步

## 压缩文件

在整个计算机历史中，人们一直在努力将最多的数据放入最小的可用空间中，无论该空间是内存、存储设备还是网络带宽。今天，许多我们认为理所当然的数据服务，如移动电话服务、高清电视、宽带互联网，都归功于有效的<u>数据压缩</u>（*data compression*）技术。

数据压缩是一个移除<u>冗余</u>（*redundancy*）数据的进程。来考虑一个虚拟场景。假设我们有一个全黑的图片文件，尺寸是 `100*100` 像素。就数据存储而言（假设是 24 位，或者每像素 3 字节），则该图片将占据 30,000 字节的存储空间（`100*100*3 = 30,000`）。

一个单一颜色的图像全都是冗余数据。如果我们聪明点，可以用一种方法编码数据，以便简单地描述「有 10,000 个黑像素块」这个事实。所以，取代存储包含有 30,000 个零的数据块（黑色在图像文件中通常被表述为 0），我们可以压缩数据到 10,000，跟随一个零，来表述我们的数据。这种数据压缩方案名为游程编码或行程长度编码（*run-length encoding*），是一种最初级的压缩技术。当今的技术已经是更高级更复杂了，但是基本目标是一样的——去除冗余数据。

<u>压缩算法</u>（*compression algorithms*）（以数学技术实现压缩），有两个普通的类别。

- 无损（*lossless*）：无损压缩保存所有包含的数据为原始状态。这意味着当一个文件从压缩版本中恢复的时候，其所恢复的文件和原始未压缩文件一模一样。
- 有损（*lossy*）：有损压缩，在另一面，当执行压缩时移除了数据，以允许应用更多的压缩。当恢复一个有损文件时，不会符合原始版本，相反，它是一个近似值。有损压缩的示例，是 JPEG 图像和 MP3 音乐。

在我们的讨论中，我们将专门研究无损压缩，因为计算机上的大多数数据都不能容忍任何数据丢失。

### gzip

`gzip` 程序用来压缩一个或多个文件。当执行后，会用压缩版本替换原始文件。相应的 `gunzip` 程序则用来恢复压缩文件为原始文件。这里是个例子：

```bash
[me@linuxbox ~]$ ls -l /etc > foo.txt
[me@linuxbox ~]$ ls -l foo.*
-rw-r--r-- 1 me    me    15738    2018-10-14 07:15 foo.txt
[me@linuxbox ~]$ gzip foo.txt
[me@linuxbox ~]$ ls -l foo.*
-rw-r--r-- 1 me    me    3230    2018-10-14 07:15 foo.txt.gz
[me@linuxbox ~]$ gunzip foo.txt
[me@linuxbox ~]$ ls -l foo.*
-rw-r--r-- 1 me    me    15738    2018-10-14 07:15 foo.txt
```

本例中，我们用一个目录中的文件列表创建了一个文件，名为 `foo.txt`。然后运行 `gzip`，它用一个压缩版本 `foo.txt.gz` 替换了原始文件。在目录中列出 `foo.*`，我们看到原始文件已经被压缩版本取代，而压缩文件大约只有原始文件五分之一的尺寸。还可以看到压缩文件具有相同的权限和时间戳。

接下来，我们运行 `gunzip` 程序以解压缩文件。然后，我们可以看到压缩版本的文件被原始文件所替换，权限和时间戳再次保留了下来。

`gzip` 具有许多选项，如表 18-1 所描述。

表 18-1：gzip 选项

| 选项      | 长选项                             | 描述                                                         |
| --------- | ---------------------------------- | ------------------------------------------------------------ |
| `-c`      | `--stdout`<br />`--to-stdout`      | 输出到标准输出，保留原始文件。                               |
| `-d`      | `--decompress`<br />`--uncompress` | 解压。功效同 `gunzip`。                                      |
| `-f`      | `--force`                          | 强行压缩，即使原始文件的压缩版本已经存在。                   |
| `-h`      | `--help`                           | 显示帮助信息。                                               |
| `-l`      | `--list`                           | 列出每个压缩文件的压缩统计数据。                             |
| `-r`      | `--recursive`                      | 如果命令行上的一个或多个参数是目录，则递归压缩其中包含的文件。 |
| `-t`      | `--test`                           | 测试一个压缩文件的完整性。                                   |
| `-v`      | `--verbose`                        | 压缩时显示详细信息。                                         |
| `-number` |                                    | 设定压缩量。*number* 是一个从 1 （最快，最少的压缩量）到 9 （最慢，最多的压缩量）的整数。数字还可以用 `---fast` 和 `--best` 来表述。默认值为 6。 |

回到更早的例子。

```bash
[me@linuxbox ~]$ gzip foo.txt
[me@linuxbox ~]$ gzip -tv foo.txt.gz
foo.txt.gz:   OK
[me@linuxbox ~]$ gzip -d foo.txt.gz
```

这里，我们用一个压缩版本的 `foo.txt.gz` 替换了文件 `foo.txt`。然后用 `-t` 和 `-v` 选项测试了压缩版本的完整性。最后，解压文件为原始形式。

`gzip` 还可以通过标准输入和输出，用起来更有趣。

```bash
[me@linuxbox ~]$ ls -l /etc | gzip > foo.txt.gz
```

该命令创建了一个目录列表的压缩版本。

解压 `gzip` 文件的 `gunzip` 程序，假设压缩文件名以 `.gz` 扩展名结尾，所以就不必指定文件的扩展名，只要指定的名称与现有的未压缩文件不冲突即可。

```bash
[me@linuxbox ~]$ gunzip foo.txt
```

如果仅仅是要查看压缩的文本文件的内容，可以这样做：

```bash
[me@linuxbox ~]$ gunzip -c foo.txt | less
```

可替换的，由 `gzip` 提供的一个程序，名为 `zcat`，等价于带 `-c` 选项的 `gunzip` 程序。它用起来像 `cat` 命令一样，用在 `gzip` 压缩文件上。

```bash
[me@linuxbox ~]$ zcat foo.txt.gz | less
```

> **提示：**还有一个 `zless` 程序。执行和上面这个管道命令一样的功能。

### bzip2

`bzip2` 程序由 Julian Seward 编写，与 `gzip` 类似，但是使用不同的压缩算法，以牺牲压缩速度来得到更高级别的压缩。大多数情况下，其工作方式和 `gzip` 一样。由 `bzip2` 压缩的文件得到 `.bz2` 的扩展名。

```bash
[me@linuxbox ~]$ ls -l /etc > foo.txt
[me@linuxbox ~]$ ls -l foo.txt
-rw-r--r-- 1 me    me    15738 2018-10-17 13:51 foo.txt
[me@linuxbox ~]$ bzip2 foo.txt
[me@linuxbox ~]$ ls -l foo.txt.bz2
-rw-r--r-- 1 me    me     2792 2018-10-17 13:51 foo.txt.bz2
[me@linuxbox ~]$ bunzip2 foo.txt.bz2
```

可以看到，`bzip2` 的用法和 `gzip` 一样。所有在 `gzip` 中讨论过的的选项（除了 `-r`）都可以用在 `bzip2` 中。但请注意，压缩级别选项（`-number`）与 `bzip2` 的含义有些不同。`bzip2` 附带了 `bunzip2` 和 `bzcat` 来解压缩文件。

`bzip2` 还带来了 `bzip2recover` 程序，可以尝试恢复那些损坏的 `.bz2` 文件。

> **不要强制压缩**
>
> 我偶尔看到人们尝试压缩一个已经由有效压缩算法压缩过的文件，如这样的：
>
> **`gzip picture.jpg`**
>
> 不要这么做。你就是在浪费时间和磁盘空间！如果你压缩一个已经压缩过的文件，通常情况下你会得到一个更大的文件。因为所有压缩技术都涉及一些添加到文件中以描述压缩的开销。如果你试图压缩一个已经没有冗余信息的文件，压缩行为通常不会导致任何节省以抵消额外的开销。

## 存档文件

常见的文件管理任务通常会把压缩和<u>存档</u>（*archiving*）联系在一起。存档是一个将许多文件收集起来捆绑在一个单独的大文件中的进程。存档通常作为系统备份的一部分，也用在将老旧数据从系统中移动到某些长期存储设备中时。

### tar

在类 Unix 软件世界中，`tar` 程序是一款经典的存档工具。它得名自 *tape archive* 的缩写，揭示其根源是一款制作磁带备份的工具。它仍然用于传统的任务，同时也适用于其它存储设备。我们经常可以看到以 `tar` 或 `tgz` 的扩展名结尾的文件，分别指示其为一个「普通」的 tar 存档文件或一个 gzip 压缩存档。tar 存档可以包含一组单独的文件，一个或多个目录层次结构，或两者的混合。命令句法如下：

```bash
tar mode[options] pathname...
```

这里，*mode* 是下表所列的操作模式之一（表 18-2 仅列出部分，可以查看 `tar` 手册页获取完整列表）。

表 18-2：tar 模式

| 模式 | 描述                               |
| ---- | ---------------------------------- |
| `c`  | 从一个文件或目录列表创建一个存档。 |
| `x`  | 释放一个存档。                     |
| `r`  | 附加指定路径名到一个存档的末尾。   |
| `t`  | 列出一个存档的内容。               |

`tar` 使用了一种稍微有点奇怪的方法来表示选项，所以我们需要一些例子来展示其如何工作。首先，重建一个游戏场，如上章所述。

```bash
[me@linuxbox ~]$ mkdir -p playground/dir-{001..100}
[me@linuxbox ~]$ touch playground/dir-{001..100}/file-{A..Z}
```

其次，创建一个整个游戏场的 tar 存档。

```bash
[me@linuxbox ~]$ tar cf playground.tar playground
```

该命令创建了一个名为 `playground.tar` 的 tar 存档，包含了整个游戏场目录层次。可以看到模式（`c`）和用来指定 tar 存档文件名的 `f` 选项，二者可以连接在一起，也不需要一个前置短横（`-`）。记住，必须先指定模式，模式总是在其他选项之前。

要列出存档内容，可以这么做：

```bash
[me@linuxbox ~]$ tar tf playground.tar
```

要看更详细的清单，可以加个 `v` 选项（verbose）。

```bash
[me@linuxbox ~]$ tar tvf playground.tar
```

现在，让我们解压游戏场到一个新处所。来创建一个新的目录，`foo`，切换到该目录然后解压 tar 存档。

```bash
[me@linuxbox ~]$ mkdir foo
[me@linuxbox ~]$ cd foo
[me@linuxbox foo]$ tar xf ../playground.tar
[me@linuxbox foo]$ ls
playground
```

如果我们检查 `~/foo/playground` 的内容，可以看到存档已经得以成功地安置，创建了一个原始文件的精确的副本。这里有个警告。除非我们正以超级用户的身份在操作，否则从存档中解压的文件和目录的属主就会改成执行恢复操作的用户，而非原始的属主。

另一个有趣的 tar 的行为，是其处理存档中的路径名。默认的路径名是相对路径而非绝对路径。tar 仅仅简单地在创建存档时，将前置的 `/` 从路径名中删掉。来演示一下，首先将重建存档，这次则指定了绝对路径。

```bash
[me@linuxbox foo]$ cd
[me@linuxbox ~]$ tar cf playground2.tar ~/playground
```

记住 `~/playground` 会在我们按下 `Enter` 键后扩展为 `/home/me/playground`，所以我们会得到在演示中得到一个绝对路径。接下来，我们来解压这个存档，看看会发生什么。

```bash
[me@linuxbox ~] cd foo
[me@linuxbox foo] tar xf ../playground2.tar
[me@linuxbox foo]$ ls
home playground
[me@linuxbox foo]$ ls home
me
[me@linuxbox foo]$ ls home/me
playground
```

这里我们可以看到，当我们解压第二个存档时，它在相对于我们的当前工作目录 `~/foo` 而不是相对于根目录，重新创建了 `home/me/playground` 目录，就像和绝对路径一样。这种工作方式可能看起来很奇怪，但是实际上这种方式会更有用，因为这允许我们解压存档到任意位置，而非被强制解压到它们的原始位置。用包含详细信息选项 `v` 来重复一下这个练习，会更清楚地了解发生了什么。

来考虑一个假象的不过也是实际的 tar 行为。想象我们想要从一个系统中复制家目录及其内容到另一个系统中，我们有一个大容量 USB 硬盘可以用来传输。在现代 Linux 系统中，该驱动器是「自动」被加载到 `/media` 目录中的。同时，假设在加载时，该磁盘的卷标是 `BigDisk`。要制作 tar 存档的话，我们可以这么做：

```bash
[me@linuxbox ~]$ sudo tar cf /media/BigDisk/home.tar /home
```

在写入 tar 文件之后，卸载驱动器，并加载到第二台计算机中。U 盘有又一次被加载为 `/media/BigDisk`。要解压这个存档，可以这么做：

```bash
[me@linuxbox2 ~]$ cd /
[me@linuxbox2 /]$ sudo tar xf /media/BigDisk/home.tar
```

这里重要的是，我们首先要切换到 `/`，以便解压是相对于根目录的，因为存档中的所有路径都是相对的。

当解压一个存档时，可以限制哪些文件从存档中解压出来。例如，假设我们想从存档中解压出单个文件，可以这样去完成：

```bash
tar xf archive.tar pathname
```

通过添加尾随的 `pathname`，tar 就会仅仅恢复那个指定的文件。还可以指定多个路径名。注意，路径名必须是存储在存档中完整准确的相对路径名。当指定路径名时，通配符是不受支持的，然而，GNU 版本的 tar （大多数常见 Linux 发行版中的版本）通过 `--wildcards` 选项可以支持通配符。这里可以用之前的游戏场文件举例如下：

```bash
[me@linuxbox ~]$ cd foo
[me@linuxbox foo]$ tar xf ../playground2.tar --wildcards 'home/me/playground/dir-*/file-A'
```

这条命令会仅仅解压匹配指定路径名中包含通配符 `dir-*` 的文件。

tar 经常被用来和 `find` 结合使用来制作存档。在下例中，将用到 `find` 来产生一个文件集，并包含到一个存档中。

```bash
[me@linuxbox ~]$ find playground -name 'file-A' -exec tar rf playground.tar '{}' '+'
```

这里，我们用 `find` 匹配所有在 `playground` 中的 `file-A` 文件，然后用 `-exec` 行为用附加模式（`r`）调用 `tar`，添加匹配的文件到 `playground.tar` 存档中。

 和 `find` 一起使用 `tar` 是一个创建增量备份的好方法，无论是对一个目录树还是对整个系统而言。通过 `find` 找到比某个时间戳文件更新的文件，我们可以创建一个存档，仅包含那些比上次存档更新的文件。假定那个时间戳文件在每次存档后得以正确地更新。

`tar` 还可以使用标准输入输出。这里是个详细的例子：

```bash
[me@linuxbox foo]$ cd
[me@linuxbox ~]$ find playground -name 'file-A' | tar cf - --files-from=- | gzip > playground.tgz
```

本例中，用 `find` 程序产生了一个匹配文件清单，并管道输出到 `tar`。如果指定了文件名 `-`，则根据需要将其视为标准输入或输出。（顺便说句，这种使用 `-` 表示标准输入输出的惯例也被许多其他程序使用）。`--files-from` 选项（它还可以写成 `-T`）导致 `tar` 从一个文件而非命令行中读取路径名列表。最后，由 `tar` 产生的存档被管道输出到 `gzip` 创建了一个压缩存档包 `playground.tgz`。`.tgz` 扩展名通常是代表由 gzip 压缩后的 tar 文件。有时也用 `.tar.gz` 来表示。

当我们使用外部 `gzip` 程序制作压缩存档时，现代版本的 GNU `tar` 同时支持 `gzip` 和 `bzip2`，分别通过 `z` 和 `j` 选项直接调用。通过之前的示例为基础，我们可以将其简化为：

```bash
[me@linuxbox ~]$ find playground -name 'file-A' | tar czf playground.tgz -T -
```

如果想用 `bzip2` 压缩，可以用下面的语句：

```bash
[me@linuxbox ~]$ find playground -name 'file-A' | tar cjf playground.tbz -T -
```

简单的从 `z` 改到 `j`（同时将输出文件的扩展明改成 `.tbz` 来指示其为 `bzip2` 压缩文件），就激活了 `bzip2` 压缩文件了。

另外一个用 tar 命令标准输入输出的有趣的用途，涉及到通过网络在系统间传输文件。想象我们有两台类 Unix 机器，都安装有 `tar` 和 `ssh`。在这个场景中，我们可以从一个远程系统（名为 `remote-sys`）中传输一个目录到我们的本地系统中。

```bash
[me@linuxbox ~]$ mkdir remote-stuff
[me@linuxbox ~]$ cd remote-stuff
[me@linuxbox remote-stuff]$ ssh remote-sys 'tar cf - Documents' | tar xf -
me@remote-sys’s password:
[me@linuxbox remote-stuff]$ ls
Documents
```

这里，我们可以从远程系统中复制一个名为 `Documents` 的目录到本地系统中的 `remote-stuff` 目录中。怎样做到的呢？首先，通过 `ssh`  运行远程系统中的 `tar` 程序。我们使用 `ssh` 通过网络远程执行一个程序，并在本地系统中「查看」结果——远程系统中的标准输出被送到本地系统中供查看。我们可以通过让 `tar` 创建一个存档（`c` 模式）并将其发送到标准输出，而不是文件（带有短横参数的 `f` 选项）从而通过 `ssh` 提供的加密隧道传输存档到本地系统。在本地系统中，我们从标准输入（还是带有短横参数的 `f` 选项）执行 `tar` 解压一个存档（`x` 模式）。

### zip

`zip` 程序既是压缩工具也是存档工具。程序所用来读写的文件格式是 Windows 用户所熟悉的 `.zip`。然而在 Linux 中，`gzip` 是优越的压缩程序，`bzip2` 则紧跟其后。

最常见的基本用法，`zip` 是这样被调用的：

```bash
zip options zipfile file...
```

例如，要制作一个游戏场的 zip 存档，是这样做的：

```bash
[me@linuxbox ~]$ zip -r playground.zip playground
```

除非我们包含 `-r` 选项用以递归，否则存档将仅包含 `playground` 目录（而不含其内容）。尽管 `.zip` 扩展名是会自动添加的，我们还是显式地包含这个扩展名。

在创建 zip 存档时，`zip` 将自然显示一系列这样的信息：

```bash
adding: playground/dir-020/file-Z (stored 0%)
adding: playground/dir-020/file-Y (stored 0%)
adding: playground/dir-020/file-X (stored 0%)
adding: playground/dir-087/ (stored 0%)
adding: playground/dir-087/file-S (stored 0%)
```

这些信息显示每个文件加到存档中的状态。`zip` 用两种存储方式中的一种将文件添加到存档中：「store」则不压缩，如上所示；「deflate」则压缩。在存储方式后的数字指示压缩量。由于游戏场仅包含空文件，所以没有对其内容执行压缩。

解压 `zip` 文件，只需要简单的使用 `unzip` 程序。

```bash
[me@linuxbox ~]$ cd foo
[me@linuxbox foo]$ unzip ../playground.zip
```

关于 `zip`，（相对与 `tar`）要注意的是，如果指定的是一个存在的存档，将会更新该存档，而非替换。这意味着已存在的存档得以保留，新的文件会被添加，重名匹配的文件将会被替换。

可以对 `unzip` 指定文件名，以便有选择地将其从 zip 存档中列表和解压出来。

```bash
[me@linuxbox ~]$ unzip -l playground.zip playground/dir-087/file-Z
Archive: ../playground.zip
  Length    Date   Time    Name
--------    ----   ----    ----
       0  10-05-16 09:25   playground/dir-087/file-Z
--------                   -------
       0                   1 file
[me@linuxbox ~]$ cd foo
[me@linuxbox foo]$ unzip ../playground.zip playground/dir-087/file-Z
Archive: ../playground.zip
replace playground/dir-087/file-Z? [y]es, [n]o, [A]ll, [N]one, [r]ename: y
 extracting: playground/dir-087/file-Z
```

使用 `-l` 选项，让 `unzip` 仅仅列出存档中的内容，而不解压文件。如果没有指定文件，`unzip` 就会列出存档中的所有文件。可以加上 `-v` 选项以增加列表的详细程度。注意，当存档解压过程中与现有文件冲突时，在替换文件前会提示用户。

和 `tar` 一样，`zip` 可以使用标准输入输出，虽然它的实现有点不太有用。可以通过 `-@` 选项管道输入给 `zip` 一个文件名列表。

```bash
[me@linuxbox foo]$ cd
[me@linuxbox ~]$ find playground -name "file-A" | zip -@ file-A.zip
```

我们用 `find` 生成了一个文件列表，匹配测试 `-name "file-A"`，并将其管道输入给 `zip`，创建一个 `file-A.zip` 存档，内含所选文件。

`zip` 还支持将其输出写到标准输出，但是用处不大，因为很少有程序可以用到这类输出。不幸的是，`unzip` 程序不能接受标准输入。这可以防止 `zip` 和 `unzip` 一起使用以执行如 `tar` 般的网络文件复制功能。

然而 `zip` 可以接受标准输入，所以可以用来压缩其它程序的输出。

```bash
[me@linuxbox ~]$ ls -l /etc/ | zip ls-etc.zip -
 adding: - (deflated 80%)
```

本例中，我们将 `ls` 的输出管道输入到 `zip`。和 `tar` 一样，`zip` 将尾随的短横（`-`）解释为「将标准输入作为输入文件」。

当 `-p`（代表 pipe）被指定时，`unzip` 程序允许将其输出送到标准输出。

```bash
[me@linuxbox ~]$ unzip -p ls-etc.zip | less
```

我们接触到了一些 `zip/unzip` 能做的基本事务。两者都有许多参数，使其得以更灵活，尽管有些是特定于其它系统平台。二者的手册页都相当完美，包含了一些有用的示例。然而，这些程序的主要用途还是和 Windows 系统交换文件，在 Linux 上压缩和存档，还是 `tar` 和 `gzip` 最常用。

## 同步文件和目录

维护系统备份的一个常用策略是，保持一个或多个目录和与另一处所——无论是本地系统（通常是可移动存储）还是远程系统——的目录（或多个目录）同步。例如，我们可以有一份正在开发中的网站本地拷贝，不时地将其同步到远程网站服务器上。

在类 Unix 世界中，这类任务首选的工具是 `rsync`。该程序通过 <u>rsync 远程更新协议</u>（*rsync remote-update protocol*）同步本地和远程目录，允许 `rsync` 快速检测两处目录间的差异并执行最小量的复制需求，以实现两者同步。这使得 `rsync` 相对于其它种类的复制程序更快速而经济。

`rsync` 调用方法如下：

```bash
rsync options source destination
```

`source` 和 `destination` 是下列之一：

- 本地文件或目录
- 远程文件或目录，格式如 *[user@]host:path*
- 用 URI 指定的远程 rsync 服务器，如 *rsync://[user@]host[:port]/path*

注意，无论源或目的地必须有一个是本地文件。不支持远程到远程的复制。

来对一些本地文件尝试一下 `rsync`。首先清理一下 `foo` 目录。

```bash
[me@linuxbox ~]$ rm -rf foo/*
```

下一步，同步 `playground` 目录到 `foo` 中相应的拷贝。

```bash
[me@linuxbox ~]$ rsync -av playground foo
```

我们加了 `-a` 选项（archiving，以递归执行且保存文件属性）和 `-v` 选项（详细输出），以便在 `foo` 目录中制作一个 `playground` 目录的<u>镜像</u>（*mirror*）。当命令运行后，我们将看到文件和目录已经复制好了。最后可以看到一条摘要信息，只是执行复制的量：

```bash
sent 135759 bytes received 57870 bytes 387258.00 bytes/sec
total size is 3230 speedup is 0.02
```

如果再次运行命令，会看到不同的结果。

```bash
[me@linuxbox ~]$ rsync -av playground foo
building file list ... done

sent 22635 bytes received 20 bytes 45310.00 bytes/sec
total size is 3230 speedup is 0.14
```

注意这次没有列出文件。因为 `rsync` 检测到 `~/playground` 和 `~/foo/playground` 两者之间没有差异，所以不需要复制任何文件。如果我们修改了 `playground` 中的一个文件再运行 `rsync`：

```bash
[me@linuxbox ~]$ touch playground/dir-099/file-Z
[me@linuxbox ~]$ rsync -av playground foo
building file list ... done
playground/dir-099/file-Z
sent 22685 bytes received 42 bytes 45454.00 bytes/sec
total size is 3230 speedup is 0.14
```

我们看到 `rsync` 检测到变化，并仅仅复制了更新过的文件。

当我们指定一个 `rsync` 源时，这是一个微妙且有用的特性。来考虑两个目录。

```bash
[me@linuxbox ~]$ ls
source        destination
```

`source` 目录中包含一个 `file1` 的文件，`destination` 目录则是空的。如果执行备份：

```bash
[me@linuxbox ~]$ rsync source destination
```

然后 `rsync` 将 `source` 复制进了 `destination`。

```bash
[me@linuxbox ~]$ ls destination
source
```

然而，如果在源目录名附加一个 `/`，则 `rsync` 只会复制源目录中的内容，而不会复制目录自身。

```bash
[me@linuxbox ~]$ rsync source/ destination
[me@linuxbox ~]$ ls destination
file1
```

这就很方便，如果我们只想复制目录中的内容而不想在目的目录中创建另一层目录。我们可以将其结果想成 `source/*`，但是这种方法会复制源目录中的所有文件，也包含隐藏文件。

作为一个实际的例子，让我们考虑一下我们之前用过 `tar` 的虚拟外部硬盘。如果再次加载该驱动器到系统中的 `/media/BigDisk`，可以执行有用的系统备份，首先在外部存储中创建 `/backup` 目录，然后用 `rsync` 从系统中复制最重要的东西到外部存储。

```bash
[me@linuxbox ~]$ mkdir /media/BigDisk/backup
[me@linuxbox ~]$ sudo rsync -av --delete /etc /home /usr/local /media/BigDisk/backup
```

本例中，从系统中复制了 `/etc`、`/home`、`/usr/local` 到虚拟外部存储设备。我们加了 `--delete` 选项，以移除那些已不存在于源设备中、但存在于备份设备中的文件（当第一次备份时这无关紧要，但是在后续备份中会有用）。重复加载外部存储、运行 `rsync` 命令这个过程是一个保持小型系统备份的有用（但不理想）的方法。当然，别名也会在这里很有帮助。可以在 `.bashrc` 中创建别名以提供此功能。

```bash
alias backup='sudo rsync -av --delete /etc /home /usr/local /media/BigDisk/backup'
```

现在我们所需做的仅仅是加载外部存储并运行 `backup` 命令以执行任务了。

### 通过网络使用 rsync

`rsync` 真正漂亮的一个好处是可以通过网络复制文件。毕竟，`r` 代表的是 `remote`。可以通过两种方式实现远程复制。第一种方式是通过远程 shell 程序登录另一个安装有 `rsync` 的系统。假设在本地网络中有另一个系统，有很大的可用磁盘空间，我们想用这个远程系统替代外部存储来执行备份操作。假设其已经有一个目录 `/backup`，可供我们送达我们的文件，我们可以这么做：

```bash
[me@linuxbox ~]$ sudo rsync -av --delete --rsh=ssh /etc /home /usr/local remote-sys:/backup
```

我们做了两处改动来促成网络拷贝。首先，我们加上了 `--rsh=ssh` 选项，指导 `rsync` 使用 `ssh` 程序作为其远程 shell。通过这种方法，我们可以用 ssh 加密通道将本地系统的数据传输到远程主机。其次，我们通过在目的路径前加前缀的方法，指定了远程主机名（本例中的远程主机名是 `remote-sys`）。

使用 `rsync` 通过网络同步文件的第二种途径是使用 <u>rsync 服务</u>（*rsync server*）。可以配置 `rsync` 为一个守护进程，监听进入的同步请求。这样做通常是允许镜像一个远程系统。例如，Red Hat Software 为其 Fedora 发行版维护了一个正在开发的大型软件包仓库。在发行版发行周期的测试阶段，镜像此收藏集，会非常有用。因为在仓库中的文件经常变更（通常一天内就变动多次），希望通过定期同步来维护本地镜像，而不是通过批量复制仓库。其中一个仓库保存在杜克大学，我们可以用本地的 `rsync` 和他们的 `rsync` 服务器来实现镜像：

```bash
[me@linuxbox ~]$ mkdir fedora-devel
[me@linuxbox ~]$ rsync -av –delete rsync://archive.linux.duke.edu/fedora/linux/development/rawhide/Everything/x86_64/os/ fedora-devel
```

本例中，用到了远程 `rsync` 服务的 URI，由协议（`rsync://`）和其后的远程主机（`archive.linux.duke.edu`）及仓库路径组成。

## 总结

我们已经看到在 Linux 和类 Unix 系统中常用的压缩和存档程序。要存档文件，`tar/gzip` 组合是 Linux 上的首选方式，`zip/unzip` 则是为了和 Windows 系统互相兼容。最后，我们学习了 `rsync` 程序（一个个人爱好），非常方便跨系统高效同步文件和目录。

## 扩展阅读

- 这里所讨论的所有命令，在手册页中有非常清楚和有用的示例。另外，GNU 项目有很好的关于它的版本的 `tar` 在线手册。可以在这里找到：http://www.gnu.org/software/tar/manual/index.html