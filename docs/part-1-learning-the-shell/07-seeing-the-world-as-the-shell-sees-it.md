# 7. 如 Shell 般看世界

本章我们讲领略一些当我们按下 `Enter` 按键的时候发生在命令行中的魔法。我们会检查一些有趣且复杂的 Shell 特征，所有这一切，我们仅用到一个新命令。

- `echo` 显示一行文本

## 扩展

每当我们键入一个命令并按下 `Enter` 键，`bash` 会在完成我们的命令之前执行几次文本替换。我们已经看到了一些案例，一个简单的字符序列，如 `*`，在 shell 中具有多种意义。让这种行为得以发生的进程，唤作<u>扩展</u>（*expansion*）。有了扩展，我们输入的东西就会在 shell 执行之前扩展成另外一些东西。要举例说明我们现在所说的意义，来看一下 `echo` 命令。`echo` 是一个内建命令，执行一个非常简单的任务：打印它的文本参数到标准输出。

```bash
[me@linuxbox ~]$ echo this is a test
this is a test
```

非常简单明了。任何传递给 `echo` 的参数都会被显示出来。让我们试试别的：

```bash
[me@linuxbox ~]$ echo *
Desktop Documents ls-output.txt Music Pictures Public Templates
Videos
```

发生了什么？为什么不是打印出 `*`？当我们回想通配符，`*` 字符意味着匹配文件名中的任意字符，但在我们最初的讨论中没有看到的 shell 怎么做。简单的答案是，在 shell 执行 `echo` 命令之前，将 `*` 扩展成了其它一些东西（在这个实例中，是当前工作目录中的文件名）。当 `Enter` 键被按下时，shell 在完成命令之前，自动扩展命令行上符合条件的字符，所以 `echo` 命令不会看到 `*`，而是仅仅看到它已经扩展后的结果。了解到这一点，我们可以看到 `echo` 的行为是符合预期的。

### 路径扩展

通配符的工作机制唤作<u>路径扩展</u>（*pathname expansion*）。如果我们尝试一些前几章采用过的技术，来看下它们真正的扩展。一个家目录看来是这样的：

```bash
[me@linuxbox ~]$ ls
Desktop   ls-output.txt  Pictures  Templates
Documents  Music         Public    Videos
```

我们可以完成这样的扩展：

```bash
[me@linuxbox ~]$ echo D*
Desktop Documents
```

和这个：

```bash
[me@linuxbox ~]$ echo *s
Documents Pictures Templates Videos
```

甚至是这个：

```bash
[me@linuxbox ~]$ echo [[:upper:]]*
Desktop Documents Music Pictures Public Templates Videos
```

来看看我们家目录之外的：

```bash
[me@linuxbox ~]$ echo /usr/*/share
/usr/kerberos/share /usr/local/share
```

> **隐藏文件的路径扩展**
>
> 我们知道，以 `.` 开始的文件名是被隐藏起来的。路径扩展也会尊重这个行为。像下面这个扩展是不会显示隐藏文件的：
>
> `echo *`
>
> 初看起来，下面这个以 `.` 开头的命令扩展能包含隐藏文件了：
>
> `echo .*`
>
> 似乎完美了。但是我们仔细检查结果，会看到 `.` 和 `..` 包含在结果中。因为这些名字指向当前工作目录及其上级目录，使用这个模式会导致错误的结果，我们可以看下面这个命令：
>
> `ls -d .* | less`
>
> 在这种情况下，我们采用一个更特殊的模式来执行更佳的路径扩展：
>
> `echo .[!.]*`
>
> 这个模式能扩展到每个以单个 `.` 开头且跟随有其它任意字符的文件名。能正确显示大多数隐藏文件（尽管仍然不能包括多个句点开头的文件名）。`ls` 命令的 `-A` 选项（almost all）将提供一个正确的隐藏文件的清单。
>
> `ls -A`

### 波浪号扩展

回忆一下 `cd` 命令，波浪号（`~`）具有特殊含义。当用在单词开始的时候，它扩展为命名用户的家目录，如果没有指定用户，就指向当前用户的家目录。

```bash
[me@linuxbox ~]$ echo ~
/home/me
```

如果用户 `foo` 有一个帐号，下面的命令就扩展为：

```bash
[me@linuxbox ~]$ echo ~foo
/home/foo
```

### 算术扩展

Shell 允许扩展执行四则运算。就能让我们把 shell 提示符当作计算器来使用。

```bash
[me@linuxbox ~]$ echo $((2 + 2))
4
```

算术扩展的用法是：

`$((expression))`

算术表达式（expression）是由数值和算术运算符组成的。

算术扩展仅支持整数（没有小数点的），不过能支持不少操作符。表 7-1 列举了一些允许操作的运算符。

表 7-1：算术运算符

| 操作符 | 描述                                             |
| ------ | ------------------------------------------------ |
| `+`    | 加法                                             |
| `-`    | 减法                                             |
| `*`    | 乘法                                             |
| `/`    | 除法（因为扩展仅支持整数，所以结果也就是整数）。 |
| `%`    | 模，简单地说，就是「余数」。                     |
| `**`   | 乘方                                             |

空格在算术运算符中并不重要，表达式也能嵌套。例如 3 乘以 5 的平方，我们可以这样写：

```bash
[me@linuxbox ~]$ echo $(($((5**2)) * 3))
75
```

单个括号可以用来组合多个子表达式。用这个技术，我们可以用单个扩展替换两个扩展来重写上面这个例子，并得到相同的结果。

```bash
[me@linuxbox ~]$ echo $(((5**2) * 3))
75
```

下面是一个简单的除法和余数操作。注意是整除：

```bash
[me@linuxbox ~]$ echo Five divided by two equals $((5/2))
Five divided two equals 2
[me@linuxbox ~]$ echo with $((5/2)) left over.
with 1 left over.
```

算术表达式在第 34 章有更深入的介绍。

### 花括号扩展

或许，最神奇的扩展要属<u>花括号扩展</u>（*brace expansion*）了。通过它，我们能用含花括号的模式创建多个文本字符串。来看下这个例子：

```bash
[me@linuxbox ~]$ echo Front-{A,B,C}-Back
Front-A-Back Front-B-Back Front-C-Back
```

用在花括号扩展的模式可以包含唤作<u>前言</u>（*preamble*）的开头一部分和被称为<u>后记</u>（*postscript*）的结束一部分。花括号表达式自身能包含逗号分隔的字符串列表、整数序列、单个字符。模式不能包含没有被引号包含的空白字符。下例用到了一个整数序列：

```bash
[me@linuxbox ~]$ echo Number_{1..5}
Number_1 Number_2 Number_3 Number_4 Number_5
```

在 4.0 或更高版本的 `bash` 中，整数可以使用<u>前置零</u>（*zero-padded*）：

```bash
[me@linuxbox ~]$ echo {01..15}
01 02 03 04 05 06 07 08 09 10 11 12 13 14 15
[me@linuxbox ~]$ echo {001..15}
001 002 003 004 005 006 007 008 009 010 011 012 013 014 015
```

下面是一串倒序排列的字符：

```bash
[me@linuxbox ~]$ echo {Z..A}
Z Y X W V U T S R Q P O N M L K J I H G F E D C B A
```

花括号表达式能被嵌套。

```bash
[me@linuxbox ~]$ echo a{A{1,2},B{3,4}}b
aA1b aA2b aB3b aB4b
```

那么，这有什么好用处？最常用的应用是创建文件或目录列表。例如，如果我们是摄影师，我们想要按年月来组织所收藏的大量图片，那我们首先要做的事情可能是创建一个「年-月」格式的目录序列。目录需要按时间顺序排列。我们可以手动敲出一个完整的目录清单，但这是一个费事的工作，也很容易出错。反之，我们这么操作：

```bash
[me@linuxbox ~]$ mkdir Photos
[me@linuxbox ~]$ cd Photos
[me@linuxbox Photos]$ mkdir {2007..2009}-{01..12}
[me@linuxbox Photos]$ ls
2007-01 2007-07 2008-01 2008-07 2009-01 2009-07
2007-02 2007-08 2008-02 2008-08 2009-02 2009-08
2007-03 2007-09 2008-03 2008-09 2009-03 2009-09
2007-04 2007-10 2008-04 2008-10 2009-04 2009-10
2007-05 2007-11 2008-05 2008-11 2009-05 2009-11
2007-06 2007-12 2008-06 2008-12 2009-06 2009-12
```

顺畅之极！

### 参数扩展

这里我们仅简单介绍一下参数扩展，后续章节中会详述。比起直接在命令行中使用，这个扩展在 shell 脚本中更有用。它的许多功能都与系统存储小块数据并为每个块命名的能力有关。许多这么的小块数据，更恰当的称呼是<u>变量</u>（*variables*），可以供我们检查。例如，名为 `USER` 的变量包含我们的用户名。要调用参数扩展并探查 `USER` 变量的内容，我们这么做：

```bash
[me@linuxbox ~]$ echo $USER
me
```

下面的命令可以查看可用的变量清单：

```bash
[me@linuxbox ~]$ printenv | less
```

或许你已经注意到，在其它类型的扩展中，如果我们打错了一个模式，扩展将不会发生，`echo` 命令会简单的显示错误输入的模式。在参数扩展中，如果我们拼写错了一个变量名，扩展将继续执行，但返回的结果会是一个空字符串。

```bash
[me@linuxbox ~]$ echo $SUER

[me@linuxbox ~]$
```

### 命令替换

命令替换能让我们用一个命令的输出作为一个扩展。

```bash
[me@linuxbox ~]$ echo $(ls)
Desktop Documents ls-output.txt Music Pictures Public Templates Videos
```

我个人最喜欢的其中一个是这样的：

```bash
[me@linuxbox ~]$ ls -l $(which cp)
-rwxr-xr-x 1 root root 71516 2007-12-05 08:58 /bin/cp
```

这里我们把 `which cp` 的结果作为参数传递给 `ls` 命令，所以我们可以不用知道 `cp` 程序的完整路径就可以将其列出。不限于简单的命令，整个管道中都能用到（这里仅仅显示部分输出）：

```bash
[me@linuxbox ~]$ file $(ls -d /usr/bin/* | grep zip)
/usr/bin/bunzip2:      symbolic link to `bzip2'
/usr/bin/bzip2:        ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.9, stripped
```

本例中，管道的结果成为 `file` 命令的参数列表。这里，命令替换是一个替代句法，在老旧的 shell 程序中也能得到支持。不过要用反引号替代美元符号和括号。

```bash
[me@linuxbox ~]$ ls -l `which cp`
-rwxr-xr-x 1 root root 71516 2007-12-05 08:58 /bin/cp
```

## 引用

既然我们已经知道了 shell 执行扩展的多种方法，是时候学习一下如何控制它了。例：

```bash
[me@linuxbox ~]$ echo this is a    test
this is a test
```

或这个：

```bash
[me@linuxbox ~]$ echo The total is $100.00
The total is 00.00
```

第一个例子中，shell 的文字分隔功能移除了 `echo` 命令的参数列表中多余的空白字符。第二个例子中，参数扩展用一个空白字符串替换了 `$1` 变量的值，因为这个变量没有被定义。Shell 提供了一个叫<u>引用</u>（*quoting*）的机制来取缔不想要的扩展。

### 双引号

第一类的引用，我们将看到「双引号」（*double quotes*）。如果我们将文本放在双引号之间，则所有用在 shell 中有特殊意义的字符将失去其特殊意义，而被看作普通字符。例外情况是 <code>$  \  `</code>。意味着文字分隔、路径扩展、波浪号扩展、花括号扩展都将被取缔，但是参数扩展、算术扩展和命令替代将继续被执行。使用双引号，我们能处理嵌入空格的文件名。假设我们是名为 <code>two words.txt</code> 文件的不幸受害者。如果我们想在命令行中用到这个文件，文字分隔将导致这个文件被看作两个参数，而非一个参数。

```bash
[me@linuxbox ~]$ ls -l two words.txt
ls: cannot access two: No such file or directory
ls: cannot access words.txt: No such file or directory
```

使用双引号，我们阻止了文字分隔，得到了想要的结果；更进一步，我们可以修复这一问题。

```bash
[me@linuxbox ~]$ ls -l "two words.txt"
-rw-rw-r-- 1 me me 18 2016-02-20 13:03 two words.txt
[me@linuxbox ~]$ mv "two words.txt" two_words.txt
```

就那样！我们现在不用输入那些讨厌的双引号了。

记住，参数扩展，算术扩展和命令替换，在双引号中仍将起作用。

```bash
[me@linuxbox ~]$ echo "$USER $((2+2)) $(cal)"
me  4  February 2019
Su Mo Tu We Th Fr Sa
                1  2
 3  4  5  6  7  8  9
10 11 12 13 14 15 16
17 18 19 20 21 22 23
24 25 26 27 28 29
```

我们应该花点时间来看一下命令替换中的双引号的效果。首先，更深入的看一下文字分隔是如何工作的。在之前的案例中，我们看到文字分隔如何移除我们文本中的多余的空白字符。

```bash
[me@linuxbox ~]$ echo this is a    test
this is a test
```

默认情况下，文字分隔查找空格、制表符和换行符，将他们看作单词间的<u>分隔符</u>（*delimiters*）。这意味着没有被引号包含的空格、制表符、换行符都不会被认为是文本的一部分，仅仅被用作分隔符。由于他们将单词分隔为不同的参数，我们的样例中，命令之后跟随着四个不同的参数。如果我们加了双引号：

```bash
[me@linuxbox ~]$ echo "this is a    test"
this is a    test
```

文字分隔被取缔，而内嵌的空格没有被作为分隔符而是被当作参数的一部分来看待。一旦加了双引号，我们的命令行中就仅有一个参数跟随着命令了。

实际上，换行符被文字分隔机制看作分隔符，会引起有趣——尽管很微妙——的命令替换效果。看下例：

```bash
[me@linuxbox ~]$ echo $(cal)
February 2019 Su Mo Tu We Th Fr Sa 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29
[me@linuxbox ~]$ echo "$(cal)"
   February 2019
Su Mo Tu We Th Fr Sa
                1  2
 3  4  5  6  7  8  9
10 11 12 13 14 15 16
17 18 19 20 21 22 23
24 25 26 27 28 29
```

第一个例子中，没有被引号包括的命令替换导致一个命令包含了 38 个参数。而在第二例的结果中，一个命令后就只有一个嵌入了空格和换行符的参数。

### 单引号

如果我们要取缔全部扩展，应该用<u>单引号</u>（*single quotes*）。下面是没有引号、双引号和单引号的比较：

```bash
[me@linuxbox ~]$ echo text ~/*.txt {a,b} $(echo foo) $((2+2)) $USER
text /home/me/ls-output.txt a b foo 4 me
[me@linuxbox ~]$ echo "text ~/*.txt {a,b} $(echo foo) $((2+2)) $USER"
text ~/*.txt {a,b} foo 4 me
[me@linuxbox ~]$ echo 'text ~/*.txt {a,b} $(echo foo) $((2+2)) $USER'
text ~/*.txt {a,b} $(echo foo) $((2+2)) $USER
```

我们能看到，随着引号级别的继承，扩展被一层层的取缔。

### 转义字符

有时候我们想引用的仅仅是一个字符。我们可以在这个字符前加一个反斜杠，在这个语境中，叫做<u>转义字符</u>（*escape character*）。通常，会在双引号内选择性的预防扩展。

```bash
[me@linuxbox ~]$ echo "The balance for user $USER is: \$5.00"
The balance for user me is: $5.00
```

使用转义字符来消除文件名中字符的特殊含义，也很常见。例如文件名中包含在 shell 中具有特殊意义的字符是有可能的。如 `$ ! &` 和空格等等。对于一个包含了特殊字符的文件名，我们可以：

```bash
[me@linuxbox ~]$ mv bad\&filename good_filename
```

用 `\\` 来转义反斜杠符号。注意，在单引号中的反斜杠会失去其特殊意义而被当作普通字符。

> **反斜杠转义序列**
>
> 除了作为转义字符的角色之外，反斜杠还用作表示某些称为控制代码的特殊字符的符号的一部分。ASCII 编码序列中的前 32 个字符用作电传设备的传输命令。有些编码是我们熟悉的（制表符、退格、换行、回车），有些则不是（空值 null、传输结束 end-of-transmission、确认 acknowledge）。
>
> | 转义序列 | 意义                               |
> | -------- | ---------------------------------- |
> | `\a`     | 铃响（引起计算机嘟嘟叫的一种警报） |
> | `\b`     | 退格                               |
> | `\n`     | 换行，在类 Unix 系统是换行符。     |
> | `\r`     | 回车                               |
> | `\t`     | 制表                               |
>
> 上表中列出了一些常用的反斜杠转义序列。在这种表示法之后的理念，源自 C 编程语言，已经被许多包含 shell 的语言所适应。
>
> 在 `echo` 命令中加 `-e` 选项，能解释转义序列。也可以把他们放在 `$' '` 之间。现在，使用 `sleep` 命令，一个等待指定数字的秒数后退出的简单程序，我们可以创造一个简陋的倒计时器：
>
> ```bash
> sleep 10; echo -e "Time's up\a"
> ```
>
> 也可以这样：
>
> ```bash
> sleep 10; echo "Time's up" $'\a'
> ```

## 总结

随着我们使用 shell，会发现扩展和引用会越来越频繁的被用到，所以需要对它们的工作方法有个很好的认知。实际上，可以说它们是了解 shell 的最重要的主题。如果没有对扩展的正确理解，shell 将永远是一个神秘和混乱的源头，它的大部分潜在力量都被浪费掉了。

## 扩展阅读

- `bash` 的手册页有一个关于扩展和引用的主要章节，以一种更正式的态度来讨论本章主题。
- *Bash Reference Manual* 也包含了扩展和引用的章节：http://www.gnu.org/software/bash/manual/bashref.html