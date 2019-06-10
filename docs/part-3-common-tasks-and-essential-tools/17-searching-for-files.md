# 17. 搜索文件

当我们徘徊在 Linux 系统时，有一件事变得非常清楚：一个典型的 Linux 系统有一大堆文件！这引出了一个问题，「我们怎么才能找到东西？」我们已经知道 Linux 文件系统是依初代类 Unix 系统传下来的惯例组织的，但是绝对数量的文件可能带来令人生畏的问题。

本章中，我们将会看到两个用来查找系统中文件的工具。

- `locate` 用名称找文件
- `find` 在目录等级中搜索文件

我们还将看到一个处理文件结果列表的文件搜索命令。

- `xargs` 从标准输入建立和执行命令行

另外，还将介绍一对辅助我们浏览的命令。

- `touch` 改变文件的时间
- `stat` 显示文件或文件系统的状态

## locate - 简易的找文件方法

`locate` 程序执行快速路径名数据库的搜索，输出每一个匹配给定字符串的名称。例如，我们想找到所有文件名以 `zip` 开头的程序。由于我们要找的是程序，可以假定包含程序的目录会以 `bin/` 结尾。因此，我们可以尝试这样用 `locate` 找到所需的文件：

```bash
[me@linuxbox ~]$ locate bin/zip
```

`locate` 会搜索其路径数据库，输出任何包含字符串 `bin/zip` 的条目。

```bash
/usr/bin/zip
/usr/bin/zipcloak
/usr/bin/zipgrep
/usr/bin/zipinfo
/usr/bin/zipnote
/usr/bin/zipsplit
```

如果搜索的需求不是这么简单，我们可以将 `locate` 和其它工具如 `grep` 合并使用，来设计出更有趣的搜索。

```bash
[me@linuxbox ~]$ locate zip | grep bin
/bin/bunzip2
/bin/bzip2
/bin/bzip2recover
/bin/gunzip
/bin/gzip
/usr/bin/funzip
/usr/bin/gpg-zip
/usr/bin/preunzip
/usr/bin/prezip
/usr/bin/prezip-bin
/usr/bin/unzip
/usr/bin/unzipsfx
/usr/bin/zip
/usr/bin/zipcloak
/usr/bin/zipgrep
/usr/bin/zipinfo
/usr/bin/zipnote
/usr/bin/zipsplit
```

`locate` 程序