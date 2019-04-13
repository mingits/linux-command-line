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

