# 5. 用命令工作

目前为止，我们看到了一系列迷一样的命令，每个命令各自有一套迷一样的选项和参数。在本章，我们会尝试抹去一些谜团，甚至会创建我们自己的命令。将在本章中介绍的命令有：

- `type` 指示如何解释命令名称
- `which` 显示哪一个可执行程序将被执行
- `help` 得到 shell 内建的帮助信息
- `man` 显示一个命令的<u>手册页</u>（*manual page*）
- `apropos` 显示适用的命令列表
- `info` 显示一个命令的信息
- `whatis` 显示一行手册页描述
- `alias` 创建一个命令的别名

## 命令究竟是什么？

一个命令可以是下列四种不同事物中的一个：

1. **一个可执行程序**。如我们在 `/usr/bin` 中所见到的那些文件。在那个目录中，用 C 和 C++ 所写的，或者那些用<u>脚本语言</u>（*scripting language* 如 shell，Perl，Python，Ruby 等等）所写的程序可以被编译成二进制文件。
2. **一个 shell 内建的命令**。`bash` 内部支持的一些命令，称为 <u>shell 内建命令</u>（*shell builtins*），例如 `cd` 就是其中一个。
3. **一个 shell 函数**。shell 函数是被纳入<u>环境</u>（*environment*）中的微型 shell 脚本。我们将在后续章节中学习如何配置环境、如何编写 shell 函数。现在我们只要知道它们存在就行了。
4. **一个别名**。别名是我们自己从其它命令定义的命令。

## 识别命令

知道所用的命令是四者中的哪个类型，通常是很有用的，而 Linux 也提供了一些途径去识别。

### type - 显示命令类型

`type` 命令是一个 shell 内建命令，用以根据给出的命令名称，显示 shell 将执行的命令的类型。用法如下：

```
type command
```

「command」是我们想要检查的命令的名称。下面是一些范例：

```bash
[me@linuxbox ~]$ type type
type is a shell builtin
[me@linuxbox ~]$ type ls
ls is aliased to `ls --color=tty'
[me@linuxbox ~]$ type cp
cp is /bin/cp
```

这里我们看到三种不同的命令。注意 `ls`（上例取自 Fedora 系统），`ls` 实际上是 `ls` 命令外加 `--color=tty` 选项的一个别名。现在我们知道了为什么 `ls` 输出结果的时候会显示不同的颜色了！

### which - 显示可执行程序的位置

系统中有时候会安装一个可执行程序的多个版本。这在桌面系统中不常见，但是在大型服务器上却不罕见。要确定所给出的可执行程序的确切位置，就要用到 `which` 命令。

```bash
[me@linuxbox ~]$ which ls
/bin/ls
```

`which` 仅仅为可执行程序服务，不为内建命令、也不为替代实际可执行程序的别名服务。当我们尝试用 `which` 检查一个内建命令如 `cd` 时，要么没有反馈，要么得到一个错误信息。

```bash
[me@linuxbox ~]$ which cd
/usr/bin/which: no cd in (/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games)
```

这样的反馈是一种「找不到命令」的花哨的说法。

## 得到命令的文档

通过对命令的了解，我们现在可以搜索可用于各种命令的文档。

### help - 得到内建命令的帮助文档

`bash` 为每个 shell 内建命令提供了一个内建的帮助文档。键入`help` 后跟内建命令的名称，即可得到。示例如下：

```bash
[me@linuxbox ~]$ help cd
cd: cd [-L|[-P [-e]] [-@]] [dir]
    Change the shell working directory.

    Change the current directory to DIR...（译者从略）
```

**关于符号的注释**：一个命令的句法中如果出现<u>方括号</u>（square brackets `[]`），意指为可选项（optional items） 。<u>竖条字符</u>（vertical bar character `|`）意指互斥项（mutually exclusive items）。如上 `cd` 命令中所示：

```bash
cd [-L|[-P[-e]]] [dir]
```

这个符号是说，`cd` 后可选择性的跟随 `-L` 或 `-P`，更进一步，如果指定了 `-P` 选项，也可以包含 `-e` 选项，随后是一个可选的参数 `dir`。

`cd` 命令的 `help` 输出的内容简洁准确，但如我们看到的，这并非入门教程，还有很多内容我们没有提到。别担心，会有的。

### --help - 显示用法信息

许多可执行程序支持 `--help` 选项以显示关于命令所支持的句法和选项。例如：

```bash
[me@linuxbox ~]$ mkdir --help
Usage: mkdir [OPTION] DIRECTORY...
Create the DIRECTORY(ies), if they do not already exist.

  -Z, --context=CONTEXT (SELinux) set security context to CONTEXT Mandatory arguments to long options are mandatory for short options too. （译者从略）
```

有些程序并不支持 `--help` 选项，不过无论有没有，都试试。通常，一些错误信息里也会提供同样用途的信息。

### man - 显示程序的手册页

大多数可执行程序会为命令行提供一个正规的文档，叫做「手册页」（*manual* 或 *man page*）。一个专门的分页程序 `man` 用来查看这些文档。用法如下：

```bash
man program
```

其中 `program` 即要查看的命令的名称。



