# 3. 探索系统

既然现在我们知道如何在文件系统中来回移动，是时候导览一下 Linux 系统了。在开始之前，我们要学几个命令，在导览时会很受用：

- `ls` 列出目录内容（List directory contents）
- `file` 确定文件类型（Determine file type）
- `less` 查看文件内容（View file contents）

## 更有趣的 ls

`ls` 命令大概是最常用的命令了。 通过它，我们能查看目录的内容、确定各种重要文件和目录的属性。如我们之前所见，我们能简单的输入 `ls` 查看当前工作目录中的文件和子目录清单：

```bash
[me@linuxbox ~]$ ls
Desktop Documents Music Pictures Public Templates Videos
```

除了当前工作目录，我们可以指定目录来列出清单，如：

```bash
me@linuxbox ~]$ ls /usr
bin games kerberos libexec sbin src
etc include lib local share tmp
```

甚至可以指定多个目录。下例中，我们将列出用户家目录（用 `~` 表示）和 `usr` 目录。

```bash
[me@linuxbox ~]$ ls ~ /usr
/home/me:
Desktop Documents Music Pictures Public Templates Videos

/usr:
bin games kerberos libexec sbin src
etc include lib local share tmp
```

我们也能改变输出格式来展示更多细节：

```bash
[me@linuxbox ~]$ ls -l
total 56
drwxrwxr-x 2 me me 4096 2007-10-26 17:20 Desktop
drwxrwxr-x 2 me me 4096 2007-10-26 17:20 Documents
drwxrwxr-x 2 me me 4096 2007-10-26 17:20 Music
drwxrwxr-x 2 me me 4096 2007-10-26 17:20 Pictures
drwxrwxr-x 2 me me 4096 2007-10-26 17:20 Public
drwxrwxr-x 2 me me 4096 2007-10-26 17:20 Templates
drwxrwxr-x 2 me me 4096 2007-10-26 17:20 Videos
```

通过在命令中加入 `-l`，我们能改变输出为长格式。

### 选项和参数

从上述命令中，我们得到一个关于命令如何工作的重点。命令通常会通过有一个或若干个<u>选项</u>（*options*）——甚者——带上一个或更多<u>参数</u>（*arguments*）来修整它们的行为。所以，大多数命令长得看起来像这样：

```bash
command -options arguments
```

大多数命令的选项由一个字母跟随一个<u>短横线</u>（dash）例如 `-l` 组成，不过还有很多命令，包括那些来自 GNU 项目的命令，支持由两个短横线和一个单词组成的<u>长选项</u>（*long options*）。