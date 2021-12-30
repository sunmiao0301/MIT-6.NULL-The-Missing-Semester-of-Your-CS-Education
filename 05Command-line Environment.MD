### 命令行环境

*—— 课后练习未全部完成*

*—— 见Supplement from video*

当您使用 shell 进行工作时，可以使用一些方法改善您的工作流，本节课我们就来讨论这些方法。我们已经使用 shell 一段时间了，但是到目前为止我们的关注点主要集中在使用不同的命令上面。现在，我们将会学习：

- 如何同时执行多个不同的进程并追踪它们的状态
- 如何停止或暂停某个进程以及如何使进程在后台运行。

- 我们还将学习一些能够改善您的 shell 及其他工具的工作流的方法，这主要是通过：
  - 定义别名或基于配置文件对其进行配置来实现的。这些方法都可以帮您节省大量的时间。
  - 例如，仅需要执行一些简单的命令，我们就可以在所有的主机上使用相同的配置。
  - 学习如何使用 SSH 操作远端机器。

### 任务控制

某些情况下我们需要中断正在执行的任务，比如当一个命令需要执行很长时间才能完成时（假设我们在使用 `find` 搜索一个非常大的目录结构）。大多数情况下，我们可以使用 `Ctrl-C` 来停止命令的执行。但是它的工作原理是什么呢？为什么有的时候会无法结束进程？

### 结束进程

您的 shell 会使用 UNIX 提供的信号机制执行进程间通信。当一个进程接收到信号时，它会停止执行、处理该信号并基于信号传递的信息来改变其执行。就这一点而言，信号是一种*软件中断*。

在上面的例子中，当我们输入 `Ctrl-C` 时，shell 会发送一个`SIGINT` 信号到进程。

下面这个 Python 程序向您展示了捕获信号`SIGINT` 并忽略它的基本操作，它并不会让程序停止。为了停止这个程序，我们需要使用`SIGQUIT` 信号，通过输入`Ctrl-\`可以发送该信号。

```python
#!/usr/bin/env python
import signal, time

def handler(signum, time):
    print("\nI got a SIGINT, but I am not stopping")

signal.signal(signal.SIGINT, handler)
i = 0
while True:
    time.sleep(.1)
    print("\r{}".format(i), end="")
    i += 1
```

如果我们向这个程序发送两次 `SIGINT` ，然后再发送一次 `SIGQUIT`，程序会有什么反应？

注意 `^` 是我们在终端输入`Ctrl` 时的表示形式：

```bash
$ python sigint.py
24^C
I got a SIGINT, but I am not stopping
26^C
I got a SIGINT, but I am not stopping
30^\[1]    39913 quit       python sigint.pyƒ
```

尽管 `SIGINT` 和 `SIGQUIT` 都常常用来发出和终止程序相关的请求。`SIGTERM` 则是一个更加通用的、也更加优雅地退出信号。为了发出这个信号我们需要使用 [`kill`](https://www.man7.org/linux/man-pages/man1/kill.1.html) 命令, 它的语法是： `kill -TERM <PID>`。

**P.S. 我们编写一个loop**

```shell
  1  #!/bin/bash
  2  while :
  3  do
  4         date
  5         sleep 1
  6  done
```

```bash
# 执行 然后
# ctrl c 可以退出 
# ctrl \ 无法退出
msun@ceph57 ~ $ ./loop
Wed Dec 22 10:08:00 CST 2021
Wed Dec 22 10:08:01 CST 2021
Wed Dec 22 10:08:02 CST 2021
^\./loop: line 6: 19893 Quit                    (core dumped) sleep 1
Wed Dec 22 10:08:02 CST 2021
Wed Dec 22 10:08:03 CST 2021
^C
msun@ceph57 ~ $ 
# zhp给出的解释是：
# ctrl+\ 这个信号是发送给了 sleep 1 这个"程序"，使得当前运行的 sleep 程序退出
# 然后继续“脚本”循环
# ctrl+c 是发送给“脚本”，使得“脚本”退出

# 如果是 直接sleep 100 无论哪种都可以直接退出
msun@ceph57 ~ $ sleep 100
^C
msun@ceph57 ~ $ sleep 100
^\Quit (core dumped)
```

### 暂停和后台执行进程

信号可以让进程做其他的事情，而不仅仅是终止它们。例如，`SIGSTOP` 会让进程暂停。在终端中，键入 `Ctrl-Z` 会让 shell 发送 `SIGTSTP` 信号。

我们可以使用 [`fg`](https://www.man7.org/linux/man-pages/man1/fg.1p.html) 或 [`bg`](http://man7.org/linux/man-pages/man1/bg.1p.html) 命令恢复暂停的工作。它们分别表示在前台继续或在后台继续。

```bash
# 执行sleep 100 并用Ctrl + z 暂停 
# 用fg在前台恢复
msun@ceph57 ~ $ sleep 100
^Z
[1]+  Stopped                 sleep 100
msun@ceph57 ~ $ fg
sleep 100
^C
msun@ceph57 ~ $ 

# 执行sleep 100 并用Ctrl + z 暂停 
# 用bg在后台恢复
msun@ceph57 ~ $ sleep 100
^Z
[1]+  Stopped                 sleep 100
msun@ceph57 ~ $ bg
[1]+ sleep 100 &
# 找到后台的程序 Ctrl + c 终止
msun@ceph57 ~ $ jobs
[1]+  Running                 sleep 100 &
msun@ceph57 ~ $ %1
sleep 100
^C
```

[`jobs`](http://man7.org/linux/man-pages/man1/jobs.1p.html) 命令会列出当前终端会话中尚未完成的全部任务。您可以使用 pid 引用这些任务（也可以用 [`pgrep`](https://www.man7.org/linux/man-pages/man1/pgrep.1.html) 找出 pid）。更加符合直觉的操作是您可以使用百分号 + 任务编号（`jobs` 会打印任务编号）来选取该任务。如果要选择最近的一个任务，可以使用 `$!` 这一特殊参数。

```bash
msun@ceph57 ~ $ sleep 100
^Z
[1]+  Stopped                 sleep 100
msun@ceph57 ~ $ jobs
[1]+  Stopped                 sleep 100
msun@ceph57 ~ $ %1
sleep 100
^C
```

还有一件事情需要掌握，那就是命令中的 `&` 后缀可以让命令在直接在后台运行，这使得您可以直接在 shell 中继续做其他操作，不过它此时还是会使用 shell 的标准输出，这一点有时会比较恼人（这种情况可以使用 shell 重定向处理）。

```bash
## 如果不写入文件 就会一直使用shell的标准输出 显示在shell上
msun@ceph57 ~ $ ./loop &
[1] 6413
msun@ceph57 ~ $ Wed Dec 22 10:38:38 CST 2021
Wed Dec 22 10:38:39 CST 2021
Wed Dec 22 10:38:40 CST 2021
Wed Dec 22 10:38:41 CST 2021
Wed Dec 22 10:38:42 CST 2021
[1]+  Running                 ./loop &
msun@ceph57 ~ $ Wed Dec 22 10:38:49 CST 2021
Wed Dec 22 10:38:50 CST 2021
Wed Dec 22 10:38:51 CST 2021
bg
-bash: bg: job 1 already in background
msun@ceph57 ~ $ Wed Dec 22 10:38:52 CST 2021
Wed Dec 22 10:38:53 CST 2021
Wed Dec 22 10:38:54 CST 2021
Wed Dec 22 10:38:55 CST 2021
Wed Dec 22 10:38:56 CST 2021
^C
msun@ceph57 ~ $ 

## 使用重定向 安静
msun@ceph57 ~ $ ./loop > loop.txt &
[1] 3354
msun@ceph57 ~ $ jobs
[1]+  Running                 ./loop > loop.txt &
msun@ceph57 ~ $ fg
./loop > loop.txt
^C
msun@ceph57 ~ $ cat loop.txt
Wed Dec 22 10:36:47 CST 2021
Wed Dec 22 10:36:48 CST 2021
Wed Dec 22 10:36:49 CST 2021
Wed Dec 22 10:36:50 CST 2021
```

让已经在运行的进程转到后台运行，您可以键入`Ctrl-Z` ，然后紧接着再输入`bg`。

**注意，后台的进程仍然是您的终端进程的子进程，一旦您关闭终端（会发送另外一个信号`SIGHUP`），这些后台的进程也会终止。**

为了防止这种情况发生，您可以使用 [`nohup`](https://www.man7.org/linux/man-pages/man1/nohup.1.html) (一个用来忽略 `SIGHUP` 的封装) 来运行程序。针对已经运行的程序，可以使用`disown` 。除此之外，您可以使用终端多路复用器来实现，下一章节我们会进行详细地探讨。

下面这个简单的会话中展示来了些概念的应用。

```bash
$ sleep 1000
^Z
[1]  + 18653 suspended  sleep 1000

$ nohup sleep 2000 &
[2] 18745
appending output to nohup.out

$ jobs
[1]  + suspended  sleep 1000
[2]  - running    nohup sleep 2000

$ bg %1
[1]  - 18653 continued  sleep 1000

$ jobs
[1]  - running    sleep 1000
[2]  + running    nohup sleep 2000

$ kill -STOP %1
[1]  + 18653 suspended (signal)  sleep 1000

$ jobs
[1]  + suspended (signal)  sleep 1000
[2]  - running    nohup sleep 2000

$ kill -SIGHUP %1
[1]  + 18653 hangup     sleep 1000

$ jobs
[2]  + running    nohup sleep 2000

$ kill -SIGHUP %2

$ jobs
[2]  + running    nohup sleep 2000

$ kill %2
[2]  + 18745 terminated  nohup sleep 2000

$ jobs
```

`SIGKILL` 是一个特殊的信号，它不能被进程捕获并且它会马上结束该进程。不过这样做会有一些副作用，例如留下孤儿进程。

您可以在 [这里](https://en.wikipedia.org/wiki/Signal_(IPC)) 或输入 [`man signal`](https://www.man7.org/linux/man-pages/man7/signal.7.html) 或使用 `kill -t` 来获取更多关于信号的信息。

### 终端多路复用

当您在使用命令行时，您通常会希望同时执行多个任务。举例来说，您可以想要同时运行您的编辑器，并在终端的另外一侧执行程序。尽管再打开一个新的终端窗口也能达到目的，使用终端多路复用器则是一种更好的办法。

像 [`tmux`](https://www.man7.org/linux/man-pages/man1/tmux.1.html) 这类的终端多路复用器可以允许我们基于面板和标签分割出多个终端窗口，这样您便可以同时与多个 shell 会话进行交互。

不仅如此，终端多路复用使我们可以分离当前终端会话并在将来重新连接。

这让您操作远端设备时的工作流大大改善，避免了 `nohup` 和其他类似技巧的使用。

现在最流行的终端多路器是 [`tmux`](https://www.man7.org/linux/man-pages/man1/tmux.1.html)。`tmux` 是一个高度可定制的工具，您可以使用相关快捷键创建多个标签页并在它们间导航。

`tmux` 的快捷键需要我们掌握，它们都是类似 `<C-b> x` 这样的组合，即需要先按下`Ctrl+b`，松开后再按下 `x`。`tmux` 中对象的继承结构如下：

- 会话  每个会话都是一个独立的工作区，其中包含一个或多个窗口
  - `tmux` 开始一个新的会话
  - `tmux new -s NAME` 以指定名称开始一个新的会话
  - `tmux ls` 列出当前所有会话
  - 在 `tmux` 中输入 `<C-b> d` ，将当前会话分离
  - `tmux a` 重新连接最后一个会话。您也可以通过 `-t` 来指定具体的会话
- 窗口  相当于编辑器或是浏览器中的标签页，从视觉上将一个会话分割为多个部分
  - `<C-b> c` 创建一个新的窗口，使用 `<C-d>`关闭
  - `<C-b> N` 跳转到第 *N* 个窗口，注意每个窗口都是有编号的
  - `<C-b> p` 切换到前一个窗口
  - `<C-b> n` 切换到下一个窗口
  - `<C-b> ,` 重命名当前窗口
  - `<C-b> w` 列出当前所有窗口
- 面板  像 vim 中的分屏一样，面板使我们可以在一个屏幕里显示多个 shell
  - `<C-b> "` 水平分割
  - `<C-b> %` 垂直分割
  - `<C-b> <方向>` 切换到指定方向的面板，<方向> 指的是键盘上的方向键
  - `<C-b> z` 切换当前面板的缩放
  - `<C-b> [` 开始往回卷动屏幕。您可以按下空格键来开始选择，回车键复制选中的部分
  - `<C-b> <空格>` 在不同的面板排布间切换

扩展阅读： [这里](https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/) 是一份 `tmux` 快速入门教程， [而这一篇](http://linuxcommand.org/lc3_adv_termmux.php) 文章则更加详细，它包含了 `screen` 命令。您也许想要掌握 [`screen`](https://www.man7.org/linux/man-pages/man1/screen.1.html) 命令，因为在大多数 UNIX 系统中都默认安装有该程序。

### 别名

输入一长串包含许多选项的命令会非常麻烦。因此，大多数 shell 都支持设置别名。shell 的别名相当于一个长命令的缩写，shell 会自动将其替换成原本的命令。例如，bash 中的别名语法如下：

```
alias alias_name="command_to_alias arg1 arg2"
```

**注意， `=`两边是没有空格的，因为 [`alias`](https://www.man7.org/linux/man-pages/man1/alias.1p.html) 是一个 shell 命令，它只接受一个参数。且注意后面是“”在内**

别名有许多很方便的特性:

```bash
# 创建常用命令的缩写
alias ll="ls -lh"

# 能够少输入很多
alias gs="git status"
alias gc="git commit"
alias v="vim"

# 手误打错命令也没关系
alias sl=ls

# 重新定义一些命令行的默认行为
alias mv="mv -i"           # -i prompts before overwrite
alias mkdir="mkdir -p"     # -p make parent dirs as needed
alias df="df -h"           # -h prints human readable format

# 别名可以组合使用
alias la="ls -A"
alias lla="la -l"

# 在忽略某个别名
\ls
# 或者禁用别名
unalias la

# 获取别名的定义
alias ll
# 会打印 ll='ls -lh'
```

**值得注意的是，在默认情况下 shell 并不会保存别名。为了让别名持续生效，您需要将配置放进 shell 的启动文件里，像是`.bashrc` 或 `.zshrc`，下一节配置文件中我们就会讲到。**

### 配置文件

![1224tanshe1.png](https://raw.githubusercontent.com/sunmiao0301/Public-Pic-Bed/main/1224tanshe1.png)

很多程序的配置都是通过纯文本格式的被称作***点文件***的配置文件来完成的（之所以称为点文件，是因为它们的文件名以 `.` 开头，例如 `~/.vimrc`。也正因为此，它们默认是隐藏文件，`ls`并不会显示它们）。

shell 的配置也是通过这类文件完成的。在启动时，您的 shell 程序会读取很多文件以加载其配置项。根据 shell 本身的不同，您从登录开始还是以交互的方式完成这一过程可能会有很大的不同。关于这一话题，[这里](https://blog.flowblok.id.au/2013-02/shell-startup-scripts.html) 有非常好的资源。（如果你真的有条理，你会把所有的点文件放在某个地方的存储库中，这样你就可以在你工作的所有机器上保持你的设置同步。）

**对于 `bash`来说，在大多数系统下，您可以通过编辑 `.bashrc` 或 `.bash_profile` 来进行配置。在文件中您可以添加需要在启动时执行的命令，例如上文我们讲到过的别名，或者是您的环境变量。**

**实际上，很多程序都要求您在 shell 的配置文件中包含一行类似 `export PATH="$PATH:/path/to/program/bin"` 的命令，这样才能确保这些程序能够被 shell 找到。**

还有一些其他的工具也可以通过*点文件*进行配置：

- `bash` - `~/.bashrc`, `~/.bash_profile`
- `git` - `~/.gitconfig`
- `vim` - `~/.vimrc` 和 `~/.vim` 目录
- `ssh` - `~/.ssh/config`
- `tmux` - `~/.tmux.conf`

我们应该如何管理这些配置文件呢，它们应该在它们的文件夹下，并使用版本控制系统进行管理，然后通过脚本将其 **符号链接** 到需要的地方。这么做有如下好处：

- **安装简单**: 如果您登录了一台新的设备，在这台设备上应用您的配置只需要几分钟的时间；
- **可以执行**: 您的工具在任何地方都以相同的配置工作
- **同步**: 在一处更新配置文件，可以同步到其他所有地方
- **变更追踪**: 您可能要在整个程序员生涯中持续维护这些配置文件，而对于长期项目而言，版本历史是非常重要的

配置文件中需要放些什么？您可以通过在线文档和[帮助手册](https://en.wikipedia.org/wiki/Man_page)了解所使用工具的设置项。另一个方法是在网上搜索有关特定程序的文章，作者们在文章中会分享他们的配置。还有一种方法就是直接浏览其他人的配置文件：您可以在这里找到无数的[dotfiles 仓库](https://github.com/search?o=desc&q=dotfiles&s=stars&type=Repositories) —— 其中最受欢迎的那些可以在[这里](https://github.com/mathiasbynens/dotfiles)找到（我们建议您不要直接复制别人的配置）。

**[这里](https://dotfiles.github.io/) 也有一些非常有用的资源。**

本课程的老师们也在 GitHub 上开源了他们的配置文件： [Anish](https://github.com/anishathalye/dotfiles), [Jon](https://github.com/jonhoo/configs), [Jose](https://github.com/jjgo/dotfiles).

### 可移植性

配置文件的一个常见的痛点是它可能并不能在多种设备上生效。例如，如果您在不同设备上使用的操作系统或者 shell 是不同的，则配置文件是无法生效的。或者，有时您仅希望特定的配置只在某些设备上生效。

有一些技巧可以轻松达成这些目的。如果配置文件 if 语句，则您可以借助它针对不同的设备编写不同的配置。例如，您的 shell 可以这样做：

```shell
if [[ "$(uname)" == "Linux" ]]; then {do_something}; fi

# 使用和 shell 相关的配置时先检查当前 shell 类型
if [[ "$SHELL" == "zsh" ]]; then {do_something}; fi

# 您也可以针对特定的设备进行配置
if [[ "$(hostname)" == "myServer" ]]; then {do_something}; fi
```

`~/.gitconfig` 可以这样编写：

```shell
[include]
    path = ~/.gitconfig_local
```

然后我们可以在日常使用的设备上创建配置文件 `~/.gitconfig_local` 来包含与该设备相关的特定配置。您甚至应该创建一个单独的代码仓库来管理这些与设备相关的配置。

如果您希望在不同的程序之间共享某些配置，该方法也适用。例如，如果您想要在 `bash` 和 `zsh` 中同时启用一些别名，您可以把它们写在 `.aliases` 里，然后在这两个 shell 里应用：

```shell
# Test if ~/.aliases exists and source it
if [ -f ~/.aliases ]; then
    source ~/.aliases
fi
```

### 远端设备

对于程序员来说，在他们的日常工作中使用远程服务器已经非常普遍了。如果您需要使用远程服务器来部署后端软件或您需要一些计算能力强大的服务器，您就会用到**安全 shell（SSH）**。

和其他工具一样，SSH 也是可以高度定制的，也值得我们花时间学习它。 —— I agree with it.

通过如下命令，您可以使用 `ssh` 连接到其他服务器：

```bash
ssh foo@bar.mit.edu
```

以上就是我们尝试以用户名 `foo` 登录服务器 `bar.mit.edu`。服务器可以通过 URL 指定（例如`bar.mit.edu`），也可以使用 IP 指定（例如`foobar@192.168.1.42`）。后面我们会介绍如何修改 ssh 配置文件使我们可以用类似 `ssh bar` 这样的命令来登录服务器。

### 执行命令 —— 远程

`ssh` 的一个经常被忽视的特性是它可以直接远程执行命令。 `ssh foobar@server ls` 可以直接在用foobar的命令下执行 `ls` 命令。 想要配合管道来使用也可以， `ssh foobar@server ls | grep PATTERN` 会在本地查询远端 `ls` 的输出而 `ls | ssh foobar@server grep PATTERN` 会在远端对本地 `ls` 输出的结果进行查询。

```bash
# 在本地查询远端ls的输出 
# ssh root@ceph57 ls -l
total 248
-rw-------.  1 root root   1632 Nov 30 14:09 anaconda-ks.cfg
-rw-r--r--   1 root root      1 Dec  3 16:54 br_netfilter
-rw-r--r--   1 root root 217613 Dec  8 11:41 calico.yaml
-rw-r--r--   1 root root      0 Dec 10 13:40 cluster.yaml
-rw-r--r--   1 root root      0 Dec 10 16:21 common.yaml
drwxr-xr-x  12 root root   4096 Dec  8 14:55 rook
-rw-r--r--   1 root root    270 Dec 15 14:33 sa-db-cb.yaml
-rw-r--r--   1 root root    107 Dec 15 14:35 sa-db.yaml
-rwxr-xr-x   1 root root     83 Dec 11 13:46 set-proxy
drwxr-xr-x   3 root root     38 Dec  9 15:34 summiao
-rwxr-xr-x   1 root root     39 Dec 11 13:46 unset-proxy
drwxr-xr-x   9 root root   4096 Dec 22 11:29 zhp

# 在远端对本地 `ls` 输出的结果进行查询
# ls | ssh root@ceph57
msun@ceph57 ~ $ ls -l| ssh root@ceph57
Pseudo-terminal will not be allocated because stdin is not a terminal.
root@ceph57's password: 
-bash: line 1: total: command not found
-bash: line 2: drwxrwxr-x: command not found
-bash: line 3: -rw-rw-r--: command not found
-bash: line 4: -rwxrw-r--: command not found
-bash: line 5: -rw-rw-r--: command not found
-bash: line 6: -rw-rw-r--: command not found
-bash: line 7: -rw-rw-r--: command not found
-bash: line 8: -rwxr-xr-x: command not found
-bash: line 9: -rw-rw-r--: command not found
-bash: line 10: -rw-rw-r--: command not found
-bash: line 11: -rw-rw-r--: command not found

## 我们应该给远端一个命令显示本地的ls结果
## 如下即可
## 下面的指令的意思是：
## 在本地ls查询 -> 传到远端 cat读取 -> 读取的结果传回本地显示
msun@ceph57 ~ $ ls | ssh root@ceph57 cat
root@ceph57's password: 
d_for_find
dotfiles.md
loop
loop.txt
ls.txt
show-args.sh
spa ce.html
ssh.log
xargs.html

# PS 这一部分还是迷糊
# 尝试使用ssh -t -t(或简称ssh -tt)强制进行伪tty分配，即使stdin不是终端。
# 另请参见：终止由bash脚本执行的ssh会话
# 从ssh手册页：
# -T      Disable pseudo-tty allocation.
# -t      Force pseudo-tty allocation.  This can be used to execute arbitrary
#        screen-based programs on a remote machine, which can be very useful,
#        e.g. when implementing menu services.  Multiple -t options force tty
#        allocation, even if ssh has no local tty.
msun@ceph57 ~ $ ls | ssh -tt root@ceph57
d_for_find
dotfiles.md
loop
loop.txt
ls.txt
show-args.sh
spa ce.html
ssh.log
xargs.html
Last failed login: Wed Dec 22 14:17:32 CST 2021 from 10.10.8.57 on ssh:notty
There was 1 failed login attempt since the last successful login.
Last login: Wed Dec 22 14:09:34 2021 from 10.10.8.57
root@ceph57 ~ # d_for_find
-bash: d_for_find: command not found
root@ceph57 ~ # dotfiles.md
-bash: dotfiles.md: command not found
root@ceph57 ~ # loop
-bash: loop: command not found
root@ceph57 ~ # loop.txt
-bash: loop.txt: command not found
root@ceph57 ~ # ls.txt
-bash: ls.txt: command not found
root@ceph57 ~ # show-args.sh
-bash: show-args.sh: command not found
root@ceph57 ~ # spa ce.html
-bash: spa: command not found
root@ceph57 ~ # ssh.log
-bash: ssh.log: command not found
root@ceph57 ~ # xargs.html
-bash: xargs.html: command not found
```

### SSH密钥

基于密钥的验证机制使用了密码学中的公钥，我们只需要向服务器证明客户端持有对应的私钥，而不需要公开其私钥。这样您就可以避免每次登录都输入密码的麻烦了秘密就可以登录。不过，私钥(通常是 `~/.ssh/id_rsa` 或者 `~/.ssh/id_ed25519`) 等效于您的密码，所以一定要好好保存它。

### 密钥生成

使用 [`ssh-keygen`](http://man7.org/linux/man-pages/man1/ssh-keygen.1.html) 命令可以生成一对密钥：

```
ssh-keygen -o -a 100 -t ed25519 -f ~/.ssh/id_ed25519
```

您可以为密钥设置密码，防止有人持有您的私钥并使用它访问您的服务器。您可以使用 [`ssh-agent`](https://www.man7.org/linux/man-pages/man1/ssh-agent.1.html) 或 [`gpg-agent`](https://linux.die.net/man/1/gpg-agent) ，这样就不需要每次都输入该密码了。

如果您曾经配置过使用 SSH 密钥推送到 GitHub，那么可能您**已经完成了[这里](https://help.github.com/articles/connecting-to-github-with-ssh/) 介绍的这些步骤**，并且已经有了一个可用的密钥对。要检查您是否持有密码并验证它，您可以运行 `ssh-keygen -y -f /path/to/key`.

##### 介绍一下[这里](https://help.github.com/articles/connecting-to-github-with-ssh/) 的步骤 — 以GitHub为例

使用 SSH 协议，您可以连接并验证远程服务器和服务。使用 SSH 密钥，您可以连接到 GitHub，而无需在每次访问时提供您的用户名和个人访问令牌。

**对于GitHub而言，具体步骤是：**

- SSH 密钥生成

[检查现有的SSH密钥](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/checking-for-existing-ssh-keys)

[生成新的SSH密钥](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#generating-a-new-ssh-key)

如果您不想在每次使用 SSH 密钥时重新输入密码，您可以将您的密钥添加到 SSH 代理，它管理您的 SSH 密钥并记住您的密码。见[将您的 SSH 密钥添加到 ssh-agent](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#adding-your-ssh-key-to-the-ssh-agent)

- 将新的 SSH 密钥添加到您的 GitHub 帐户

[将新的 SSH 密钥添加到您的 GitHub 帐户](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)

- 测试您的 SSH 连接

[测试您的 SSH 连接](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/testing-your-ssh-connection)

- 使用 SSH 密钥密码

[略]()

使用 SSH 密钥，如果有人可以访问您的计算机，他们也可以访问使用该密钥的每个系统。要添加额外的安全层，您可以向 SSH 密钥添加密码。您可以使用`ssh-agent`安全地保存您的密码，这样您就不必重新输入它。

**总体来说的ssh登录流程是：**

- 远程服务器持有公钥，当有用户进行登录，服务器就会随机生成一串字符串，然后发送给正在进行登录的用户。
- 用户收到远程服务器发来的字符串，使用与**远程服务器公钥配对的私钥**对字符串进行加密，再发送给远程服务器。
- 服务器使用公钥对用户发来的加密字符串进行解密，得到的解密字符串如果与第一步中发送给客户端的随机字符串一样，那么判断为登录成功。

### 基于密钥的认证机制

`ssh` 会查询 `.ssh/authorized_keys` 来确认那些用户可以被允许登录。您可以通过下面的命令将一个公钥拷贝到这里：

```bash
cat .ssh/id_ed25519.pub | ssh foobar@remote 'cat >> ~/.ssh/authorized_keys'
```

如果支持 `ssh-copy-id` 的话，可以使用下面这种更简单的解决方案：

```bash
ssh-copy-id -i .ssh/id_ed25519.pub foobar@remote
```

### 通过 SSH 复制文件

使用 ssh 复制文件有很多方法：

- `ssh+tee`, 最简单的方法是执行 `ssh` 命令，然后通过这样的方法利用标准输入实现 `cat localfile | ssh remote_server tee serverfile`。回忆一下，[`tee`](https://www.man7.org/linux/man-pages/man1/tee.1.html) 命令会将标准输出写入到一个文件；

```bash
TEE(1)                                   User Commands                                   TEE(1)

NAME
       tee - read from standard input and write to standard output and files
```

```bash
## 在本地机上
msun@ceph57 ~ $ cat hello.txt
hello
msun@ceph57 ~ $ cat hello.txt | ssh root@ceph57 tee hello.txt
root@ceph57's password: 
hello

# 在远端机上
root@ceph57 ~ # cat hello.txt
hello

## 如上 就完成了将本地机的一个文件的内容 写入远端机的文件内
```

- [`scp`](https://www.man7.org/linux/man-pages/man1/scp.1.html) ：当需要拷贝大量的文件或目录时，使用`scp` 命令则更加方便，因为它可以方便的遍历相关路径。语法如下：`scp path/to/local_file remote_host:path/to/remote_file`；
- p.s. scp = ssh cp

```bash
msun@ceph57 ~ $ scp /home/msun/loop.txt root@10.10.8.57:/
root@10.10.8.57's password: 
loop.txt                                                                              100% 1102     1.4MB/s   00:00  

root@ceph57 / # ls
bin   clustre.yaml  etc   lib    loop.txt  mnt  proc  run   srv  tmp  var
boot  dev           home  lib64  media     opt  root  sbin  sys  usr

# 说明 
# 1.如果远程服务器防火墙有为scp命令设置了指定的端口，我们需要使用 -P 参数来设置命令的端口号，命令格式如下：
#scp 命令使用端口号 4588
# scp -P 4588 remote@www.runoob.com:/usr/local/sin.sh /home/administrator

# 2.使用scp命令要确保使用的用户具有可读取远程服务器相应文件的权限，否则scp命令是无法起作用的。
```

- [`rsync`](https://www.man7.org/linux/man-pages/man1/rsync.1.html) 对 `scp` 进行了改进，它可以检测本地和远端的文件以防止重复拷贝。它还可以提供一些诸如符号连接、权限管理等精心打磨的功能。甚至还可以基于 `--partial`标记实现断点续传。`rsync` 的语法和`scp`类似；

### 端口转发 — 重要

很多情况下我们都会遇到软件需要监听特定设备的端口。如果是在您的本机，可以使用 `localhost:PORT` 或 `127.0.0.1:PORT`。但是如果需要监听远程服务器的端口该如何操作呢？这种情况下远端的端口并不会直接通过网络暴露给您。

此时就需要进行 *端口转发*。端口转发有两种，一种是本地端口转发和远程端口转发（参见下图，该图片引用自这篇[StackOverflow 文章](https://unix.stackexchange.com/questions/115897/whats-ssh-port-forwarding-and-whats-the-difference-between-ssh-local-and-remot)）中的图片。

常见的情景是使用本地端口转发，即远端设备上的服务监听一个端口，而您希望在本地设备上的一个端口建立连接并转发到远程端口上。例如，我们在远端服务器上运行 Jupyter notebook 并监听 `8888` 端口。 然后，建立从本地端口 `9999` 的转发，使用 `ssh -L 9999:localhost:8888 foobar@remote_server` 。这样只需要访问本地的 `localhost:9999` 即可。

### SSH配置

我们已经介绍了很多参数。为它们创建一个别名是个好想法，我们可以这样做：

```bash
alias my_server="ssh -i ~/.id_ed25519 --port 2222 -L 9999:localhost:8888 foobar@remote_server
```

不过，更好的方法是使用 `~/.ssh/config`.

```bash
Host vm
    User foobar
    HostName 172.16.174.141
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
    LocalForward 9999 localhost:8888

# 在配置文件中也可以使用通配符
Host *.mit.edu
    User foobaz
```

这么做的好处是，使用 `~/.ssh/config` 文件来创建别名，类似 `scp`、`rsync`和`mosh`的这些命令都可以读取这个配置并将设置转换为对应的命令行选项。

注意，`~/.ssh/config` 文件也可以被当作配置文件，而且一般情况下也是可以被导入其他配置文件的。不过，如果您将其公开到互联网上，那么其他人都将会看到您的服务器地址、用户名、开放端口等等。这些信息可能会帮助到那些企图攻击您系统的黑客，所以请务必三思。

服务器侧的配置通常放在 `/etc/ssh/sshd_config`。您可以在这里配置免密认证、修改 shh 端口、开启 X11 转发等等。 您也可以为每个用户单独指定配置。

### 杂项

连接远程服务器的一个常见痛点是遇到由关机、休眠或网络环境变化导致的掉线。如果连接的延迟很高也很让人讨厌。[Mosh](https://mosh.org/)（即 mobile shell ）对 ssh 进行了改进，它允许连接漫游、间歇连接及智能本地回显。

有时将一个远端文件夹挂载到本地会比较方便， [sshfs](https://github.com/libfuse/sshfs) 可以将远端服务器上的一个文件夹挂载到本地，然后您就可以使用本地的编辑器了。

### Shell & 框架

在 shell 工具和脚本那节课中我们已经介绍了 `bash` shell，因为它是目前最通用的 shell，大多数的系统都将其作为默认 shell。但是，它并不是唯一的选项。

例如，`zsh` shell 是 `bash` 的超集并提供了一些方便的功能：

- 智能替换, `**`
- 行内替换/通配符扩展
- 拼写纠错
- 更好的 tab 补全和选择
- 路径展开 (`cd /u/lo/b` 会被展开为 `/usr/local/bin`)

**框架** 也可以改进您的 shell。比较流行的通用框架包括[prezto](https://github.com/sorin-ionescu/prezto) 或 [oh-my-zsh](https://ohmyz.sh/)。还有一些更精简的框架，它们往往专注于某一个特定功能，例如[zsh 语法高亮](https://github.com/zsh-users/zsh-syntax-highlighting) 或 [zsh 历史子串查询](https://github.com/zsh-users/zsh-history-substring-search)。 像 [fish](https://fishshell.com/) 这样的 shell 包含了很多用户友好的功能，其中一些特性包括：

- 向右对齐
- 命令语法高亮
- 历史子串查询
- 基于手册页面的选项补全
- 更智能的自动补全
- 提示符主题

需要注意的是，使用这些框架可能会降低您 shell 的性能，尤其是如果这些框架的代码没有优化或者代码过多。您随时可以测试其性能或禁用某些不常用的功能来实现速度与功能的平衡。

### 终端模拟器

和自定义 shell 一样，花点时间选择适合您的 **终端模拟器**并进行设置是很有必要的。有许多终端模拟器可供您选择（这里有一些关于它们之间[比较](https://anarc.at/blog/2018-04-12-terminal-emulators-1/)的信息）

您会花上很多时间在使用终端上，因此研究一下终端的设置是很有必要的，您可以从下面这些方面来配置您的终端：

- 字体选择
- 彩色主题
- 快捷键
- 标签页/面板支持
- 回退配置
- 性能（像 [Alacritty](https://github.com/jwilm/alacritty) 或者 [kitty](https://sw.kovidgoyal.net/kitty/) 这种比较新的终端，它们支持GPU加速）。

# HOMEWORK

## 任务控制

1. 我们可以使用类似 `ps aux | grep` 这样的命令来获取任务的 pid ，然后您可以基于pid 来结束这些进程。但我们其实有更好的方法来做这件事。在终端中执行 `sleep 10000` 这个任务。然后用 `Ctrl-Z` 将其切换到后台并使用 `bg`来继续允许它。现在，使用 [`pgrep`](https://www.man7.org/linux/man-pages/man1/pgrep.1.html) 来查找 pid 并使用 [`pkill`](https://www.man7.org/linux/man-pages/man1/pgrep.1.html) 结束进程而不需要手动输入pid。(提示：: 使用 `-af` 标记)。

   ```bash
   # PID 是 进程号
   msun@ceph57 ~ $ sleep 10000
   ^Z
   [1]+  Stopped                 sleep 10000
   msun@ceph57 ~ $ jobs
   [1]+  Stopped                 sleep 10000
   msun@ceph57 ~ $ bg
   [1]+ sleep 10000 &
   msun@ceph57 ~ $ jobs
   [1]+  Running                 sleep 10000 &
   msun@ceph57 ~ $ pgrep sleep
   18430
   18431
   18432
   18433
   18434
   18435
   18436
   18437
   20010
   20342
   msun@ceph57 ~ $ pkill -ef sleep
   pkill: killing pid 18430 failed: Operation not permitted
   pkill: killing pid 18431 failed: Operation not permitted
   pkill: killing pid 18432 failed: Operation not permitted
   pkill: killing pid 18433 failed: Operation not permitted
   pkill: killing pid 18434 failed: Operation not permitted
   pkill: killing pid 18435 failed: Operation not permitted
   pkill: killing pid 18436 failed: Operation not permitted
   pkill: killing pid 18437 failed: Operation not permitted
   sleep killed (pid 20010)
   pkill: killing pid 21223 failed: Operation not permitted
   [1]+  Terminated              sleep 10000
   msun@ceph57 ~ $ jobs
   
   #
   msun@ceph57 ~ $ sleep 10000
   ^Z
   [1]+  Stopped                 sleep 10000
   msun@ceph57 ~ $ bg
   [1]+ sleep 10000 &
   msun@ceph57 ~ $ jobs
   [1]+  Running                 sleep 10000 &
   msun@ceph57 ~ $ sudo pkill -ef sleep
   [sudo] password for msun: 
   sleep killed (pid 18430)
   sleep killed (pid 18431)
   sleep killed (pid 18432)
   sleep killed (pid 18433)
   sleep killed (pid 18434)
   sleep killed (pid 18435)
   sleep killed (pid 18436)
   sleep killed (pid 18437)
   sleep killed (pid 21512)
   sudo killed (pid 21988)
   sleep killed (pid 21991)
   [1]+  Terminated              sleep 10000
   ```

2. 如果您希望某个进程结束后再开始另外一个进程， 应该如何实现呢？在这个练习中，我们使用 `sleep 60 &` 作为先执行的程序。一种方法是使用 [`wait`](http://man7.org/linux/man-pages/man1/wait.1p.html) 命令。尝试启动这个休眠命令，然后待其结束后再执行 `ls` 命令。

   ```bash
   sleep 60 | wait && ls
   
   #
   msun@ceph57 ~ $ sleep 5 | wait & ls
   [1] 4666
   d_for_find  dotfiles.md  hello.txt  loop  loop.txt  ls.txt  show-args.sh  spa ce.html  ssh.log  xargs.html
   ```

   但是，如果我们在不同的 bash 会话中进行操作，则上述方法就不起作用了。因为 `wait` 只能对子进程起作用。之前我们没有提过的一个特性是，`kill` 命令成功退出时其状态码为 0 ，其他状态则是非0。`kill -0` 则不会发送信号，但是会在进程不存在时返回一个不为0的状态码。请编写一个 bash 函数 `pidwait` ，它接受一个 pid 作为输入参数，然后一直等待直到该进程结束。您需要使用 `sleep` 来避免浪费 CPU 性能。

## 终端多路复用

1. 请完成这个 `tmux` [教程](https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/) 参考[这些步骤](https://www.hamvocke.com/blog/a-guide-to-customizing-your-tmux-conf/)来学习如何自定义 `tmux`。

## 别名

1. 创建一个 `dc` 别名，它的功能是当我们错误的将 `cd` 输入为 `dc` 时也能正确执行。

   ```bash
   msun@ceph57 ~ $ alias dc="cd"
   msun@ceph57 ~ $ dc dir
   msun@ceph57 ~/dir $ 
   
   # 尽管"在默认情况下 shell 并不会保存别名"
   # 我们试试直接禁用
   msun@ceph57 ~ $ unalias dc
   msun@ceph57 ~ $ dc dir
   dc: Will not attempt to process directory dir
   ```

2. 执行 `history | awk '{$1="";print substr($0,2)}' | sort | uniq -c | sort -n | tail -n 10` 来获取您最常用的十条命令，尝试为它们创建别名。注意：这个命令只在 Bash 中生效，如果您使用 ZSH，使用`history 1` 替换 `history`。

   ```bash
   msun@ceph57 ~ $ history | awk '{$1="";print substr($0,2)}' | sort | uniq -c | sort -n | tail -n 10
        11 unzip html.zip
        11 vim ls.txt
        12 sleep 100
        13 man xargs
        18 history
        18 man find
        22 cd
        24 jobs
        27 la
       141 ls
   ...
   ```

## 配置文件

让我们帮助您进一步学习配置文件：

1. 为您的配置文件新建一个文件夹，并设置好版本控制

   ```bash
   msun@ceph57 ~ $ mkdir dotfiles
   ```

2. 在其中添加至少一个配置文件，比如说您的 shell，在其中包含一些自定义设置（可以从设置 `$PS1` 开始）。

3. 建立一种在新设备进行快速安装配置的方法（无需手动操作）。最简单的方法是写一个 shell 脚本对每个文件使用 `ln -s`，也可以使用[专用工具](https://dotfiles.github.io/utilities/)

   ```bash
   #!/bin/bash
   files="bashrc vimrc "
   
   for file in $files; do
    # ln - make links between files
      ln -s ~/dotfiles/$file ~/.$file
   done
   ```

   分为[硬链接和软链接](https://chinese.freecodecamp.org/news/the-linux-commands-handbook/)

   ```bash
   # 硬链接
   ln <源文件路径> <链接路径>
   ln recipes.txt newrecipes.txt
   
   # 硬链接现在很少使用。它有一些弱点：你无法链接到目录，也无法链接到外部文件系统（磁盘驱动器）。
   # 现在，如果你编辑这对文件中的任何一个，另一个文件的内容也会随之更新。
   # 即使你删除了源文件，链接文件仍然会包含源文件的内容。那是因为直到创建硬链接时，源文件还没有被删除。
   
   # 如下测试
   msun@ceph57 ~ $ echo s > s.txt
   
   # 注意 创建链接的时候 被链接文件夹不需要存在
   msun@ceph57 ~ $ ls
   s.txt
   msun@ceph57 ~ $ cat s.txt
   s
   
   # 创建硬链接
   msun@ceph57 ~ $ ln s.txt l.txt
   msun@ceph57 ~ $ ls
   l.txt  s.txt
   msun@ceph57 ~ $ cat l.txt
   s
   msun@ceph57 ~ $ rm -f s.txt
   
   # 即使你删除了源文件，链接文件仍然会包含源文件的内容。那是因为直到创建硬链接时，源文件还没有被删除
   msun@ceph57 ~ $ cat l.txt
   s
   ```

   **一言以蔽之，硬链接用的不多**

   ```bash
   # 软链接
   # 你可以使用 ln 命令的 -s 参数创建一个软链接
   ln -s <源文件路径> <链接路径>
   ln -s recipes.txt newrecipes.txt
   
   # 软链接则有所不同，它更为强大一些，你可以用它链接到其他的文件系统和目录。但请注意，当你删除源文件时，这个链接会断掉。
   # 测试如下
   msun@ceph57 ~ $ echo s > s.txt
   msun@ceph57 ~ $ ls
   dotfiles  loop  show-args.sh  sm-vim  s.txt
   msun@ceph57 ~ $ ln -s s.txt l.txt
   msun@ceph57 ~ $ ls
   dotfiles  loop  l.txt  show-args.sh  sm-vim  s.txt
   msun@ceph57 ~ $ 
   
   # 并且在软链接情况下，当你用 ls -al 列出文件时，你可以看见标出了链接关系
   msun@ceph57 ~ $ ls -al
   lrwxrwxrwx  1 msun msun     5 Dec 30 17:35 l.txt -> s.txt
   
   # 现在，如果你删除源文件，链接就会断掉，如果你在终端里尝试访问它，Shell 会提示 "No such file or directory" （没有这个文件或目录）
   msun@ceph57 ~ $ rm -f s.txt
   msun@ceph57 ~ $ ls
   dotfiles  loop  l.txt  show-args.sh  sm-vim
   msun@ceph57 ~ $ cat l.txt
   cat: l.txt: No such file or directory
   # 如果开了终端颜色，还可以看到l.txt由浅蓝色变成红色
   ```

   **由以上可知，我们创建点文件的软链接的时候 应该是：**

   ```bash
   ln -s ~/dotfiles/$file ~/.$file
   ```

   **也就是把dotfile文件内的源文件，链接，给到系统的配置文件。**

   做到这里，我感觉这个方法不太方便，不如直接如下

   ![1230scporlink.png](https://raw.githubusercontent.com/sunmiao0301/Public-Pic-Bed/main/1230scporlink.png)

4. 在新的虚拟机上测试该安装脚本。

5. 将您现有的所有配置文件移动到项目仓库里。

6. 将项目发布到GitHub。

## 远端设备 — 重要

进行下面的练习需要您先安装一个 Linux 虚拟机（如果已经安装过则可以直接使用），如果您对虚拟机尚不熟悉，可以参考[这篇教程](https://hibbard.eu/install-ubuntu-virtual-box/) 来进行安装。

**出于一举两得的考量，我们就用本地windows连接Linux服务器，最终实现从Windows到Linux服务器的免密登录**

**预备知识**，先在服务器下检查有没有 home/user/.ssh，没有的话，```ssh locanhost```来生成一个(`~/.ssh` 是在运行第一次 `ssh` 命令的时候自动创建的。)

```bash
msun@ceph57 ~ $ ssh msun@ceph57
The authenticity of host 'ceph57 (10.10.8.57)' can't be established.
ECDSA key fingerprint is SHA256:HUtJVYsRZKvKhlFe2TARyNgyfbgsnD4DEsBcRP3jObs.
ECDSA key fingerprint is MD5:07:f3:a4:67:19:fc:e5:98:8b:70:0a:36:d2:62:b4:25.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'ceph57,10.10.8.57' (ECDSA) to the list of known hosts.
msun@ceph57's password: 
Last login: Thu Dec 23 09:17:33 2021 from 10.12.3.1
msun@ceph57 ~ $ cd ~
msun@ceph57 ~ $ ls -a
.              .bash_logout   d_for_find   .emacs     .lesshst  ls.txt        .ssh      xargs.html
..             .bash_profile  dir          .emacs.d   loop      show-args.sh  .viminfo
.bash_history  .bashrc        dotfiles.md  hello.txt  loop.txt  spa ce.html   .vimrc
msun@ceph57 ~ $ cd .ssh
msun@ceph57 ~/.ssh $
```

前往 `~/.ssh/` 并查看是否已经存在 SSH 密钥对。如果不存在，请使用`ssh-keygen`来创建一个。建议为密钥设置密码然后使用`ssh-agent`，更多信息可以参考 [这里](https://www.ssh.com/ssh/agent)；

```bash
# 在Windows PowerShell 生成密钥
PS C:\Users\hfut_> ssh-keygen
Generating public/private rsa key pair.
# 选择生成密钥的位置 如果是第一次 直接默认地址就行
Enter file in which to save the key (C:\Users\hfut_/.ssh/id_rsa):
# 第一次输入密码
Enter passphrase (empty for no passphrase):
# 再次输入密码
Enter same passphrase again:
# 私钥完毕
Your identification has been saved in C:\Users\hfut_/.ssh/id_rsa.
# 公钥完毕
Your public key has been saved in C:\Users\hfut_/.ssh/id_rsa.pub.
# 指纹
# 以下是 公钥密钥指纹以及它的的可视化，公布它应该是安全的
The key fingerprint is:
SHA256:—————————————————————————————————————— hfut_@DESKTOP-HA2C4BG
The key's randomart image is:
+---[RSA 3072]----+
|          +O=.oo+|
|         ++.++. o|
|        .oooo  =.|
|       ..  o  +.*|
|        S . ...*E|
———————————————————
———————————————————
|     = + o .o  . |
|    o.o   .      |
+----[SHA256]-----+
```

**关于ssh-agent 如下**

*The `ssh-agent` is a helper program that keeps track of user's [identity keys](https://www.ssh.com/ssh/identity-key) and their [passphrases](https://www.ssh.com/ssh/passphrase). The agent can then use the keys to log into other servers without having the user type in a password or passphrase again. This implements a form of single sign-on (SSO).*

在`.ssh/config`加入下面内容：

```bash
# Port 字段指定远程主机端口号，默认为 22 。
# Host 字段配置了登录别名，比如 ssh docker 表示使用别名为 docker 的配置登录远端的服务器。
# 这里需要注意的是，Host 是支持通配符的， * 代表0～n个非空白字符，? 代表一个非空白字符，! 表示例外通配。

Host msun
	# 指定登录的用户名
    User msun
    # 指定远程主机名，可以直接使用 IP 地址。
    # 如果这个字段中包含 ‘%h’ ，则实际使用时会被命令行中的主机名替换。
    # 比如 demo 中如果设置 HostName 0.0.0.0.%h
    # 那么在运行 ssh docker 后，会把主机名解析为 0.0.0.0.docker。
    HostName 10.10.8.57
    # 私钥路径 —— 如果只有一个私钥 就不需要 多个私钥 还是加上吧
    IdentityFile ~/.ssh/id_rsa
    ## 此外，因为我用了跳板机 所以这里得加一个 跳板机@ip地址:端口号
    ProxyJump msun@202.38.95.226:2222
    
    # 为了接下来在本机访问服务器
    # 用到了本地端口转发LocalForward
    LocalForward 9999:localhost:8888
```

关于上面的LocalForward，zhp推荐Dynamic 可以实现本地一个端口对接服务器多个端口，这里需要单独开一篇[动态端口转发]()来单独详细介绍了

然后使用 `ssh-copy-id vm` 将您的 ssh 密钥拷贝到服务器。

```bash
# 命令如下
cat ~/.ssh/id_rsa.pub | ssh -J msun@202.38.95.226:2222 root@10.10.8.5 tee .ssh/authorized_keys

# 或者直接如下
ssh-copy-id root@10.10.8.57

# 或者直接拷贝到跳板机和服务器
# 跳板机
msun@ubuntu:~/.ssh$ vim authorized_keys
# 服务器
msun@ceph57 ~/.ssh $ vim authorized_keys
```

此时可以先测试一下 我们在Windows PowerShell上运行

```bash
# 在本地机上配置好config之后 直接ssh msun(也就是Host别名即可)
PS C:\Users\hfut_> ssh msun
Last login: Fri Dec 24 09:58:54 2021 from 10.12.3.1
root@ceph57 ~ #

# p.s. 如果debug是文件权限问题，权限修改一下
msun@ceph57 /etc/ssh $ chmod 700 ~/.ssh
msun@ceph57 /etc/ssh $ chmod 600 ~/.ssh/authorized_keys
```

使用`python -m http.server 8888` 在您的虚拟机中启动一个 Web 服务器并通过本机的`http://localhost:9999` 访问虚拟机上的 Web 服务器

```bash
0## 注意 这里的命令的 http.server 属于python3 如下
# If Python version returned above is 3.X
python3 -m http.server

# 所以我们应该运行的是 SimpleHTTPServer
# If Python version returned above is 2.X
root@ceph57 ~ # python -m SimpleHTTPServer 8888
Serving HTTP on 0.0.0.0 port 8888 ...

## 然后我们已经在本地运行了一个本地端口转发
## 此时我们在本地 Windows PowerShell 连接上服务器
PS C:\Users\hfut_> ssh root57
Last login: Fri Dec 24 12:34:02 2021 from 10.12.3.1
root@ceph57 ~ #

## 并且我们已经在config中配置了本地转发端口如下
LocalForward 9999:localhost:8888
## 此时就可以直接在本地浏览器打开 
http://localhost:9999/
## 就可以看到如下结果
Directory listing for /
.aws/
.bash_history
.bash_logout
.bash_profile
.bashrc
.cache/
.config/
.cshrc
.docker/
.emacs
.emacs.d/
.gitconfig
.kube/
.lesshst
.minikube/
.pki/
.redhat/
.ssh/
.tcshrc
.tldr/
.viminfo
.vimrc
.vscode-server/
anaconda-ks.cfg
br_netfilter
calico.yaml
cluster.yaml
common.yaml
hello.txt
rook/
set-proxy
summiao/
unset-proxy
zhp/
```

使用`sudo vim /etc/ssh/sshd_config` 编辑 SSH 服务器配置，通过修改`PasswordAuthentication`的值来禁用密码验证。通过修改`PermitRootLogin`的值来禁用 root 登录。然后使用`sudo service sshd restart`重启 `ssh` 服务器，然后重新尝试。

(附加题) 在虚拟机中安装 [`mosh`](https://mosh.org/) 并启动连接。然后断开服务器/虚拟机的网络适配器。mosh可以恢复连接吗？

(附加题) 查看`ssh`的`-N` 和 `-f` 选项的作用，找出在后台进行端口转发的命令是什么？

**在做之前需要明确的：对于我们的情境，ssh是用于本地机的指令。**

```bash
# 原本 我们是
PS C:\Users\hfut_> ssh root57
Last login: Mon Dec 27 09:29:48 2021 from 10.12.3.1
root@ceph57 ~ # 

-N      Do not execute a remote command.  This is useful for just forwarding ports.

-f      Requests ssh to go to background just before command execution.  This is useful if ssh is going to ask for passwords or passphrases, but the user wants it in the background.  This implies -n.  The recommended way to start X11 programs at a remote site is with something like ssh -f host xterm.

If the ExitOnForwardFailure configuration option is set to ``yes'', then a client started with -f will wait for all remote port forwards to be successfully established before placing itself in the background.

# 由上可知 -N 使得不执行远端命令 对端口转发生效
# -f 使得在后台执行
# 所以如果想要本地机在后台进行端口转发的命令 应该是
PS C:\Users\hfut_> ssh -fN root57
```



## Supplement from video

```bash
# 建立一个在后台的
msun@ceph57 ~ $ nohup sleep 2000 &
[3] 28099
[2]   Terminated              sleep 1000
msun@ceph57 ~ $ nohup: ignoring input and appending output to ‘nohup.out’
jobs
[3]+  Running                 nohup sleep 2000 &
msun@ceph57 ~ $ jobs
[3]+  Running                 nohup sleep 2000 &
msun@ceph57 ~ $ sleep 1000
^Z
[4]+  Stopped                 sleep 1000
msun@ceph57 ~ $ jobs
[3]-  Running                 nohup sleep 2000 &
[4]+  Stopped                 sleep 1000
msun@ceph57 ~ $ bg %4
[4]+ sleep 1000 &
msun@ceph57 ~ $ jobs
[3]-  Running                 nohup sleep 2000 &
[4]+  Running                 sleep 1000 &
msun@ceph57 ~ $ kill -STOP %4

[4]+  Stopped                 sleep 1000
msun@ceph57 ~ $ jobs
[3]-  Running                 nohup sleep 2000 &
[4]+  Stopped                 sleep 1000
msun@ceph57 ~ $ kill -HUP %4
[4]+  Hangup                  sleep 1000
msun@ceph57 ~ $ jobs
[3]+  Running                 nohup sleep 2000 &
msun@ceph57 ~ $ kill -HUP %3
msun@ceph57 ~ $ jobs
[3]-  Running                 nohup sleep 2000 &

msun@ceph57 ~ $ kill -KILL %3
msun@ceph57 ~ $ jobs

# bg 与 fg

## tmux
- Sessions
  - Windows
    - panes

# 别名的用处之 —— 为一些关键的 flag 加上默认执行
# 如下 -i 会询问是否overwrite 如此写会
root@ceph57 ~ # alias mv="mv -i"
root@ceph57 ~ #mv a.sh b.sh
overwrite b.sh? (y/n [n])

# alise mv 能够查看其已经设定的别名
root@ceph57 ~ # alias mv
alias mv='mv -i'

# 但是alise的设定都会在关闭terminal之后消失 
# 所以如果需要其永久生效 我们可以编辑点文件dotfile
## 对于bash而言 点文件是 ~/.bashrc (在home文件夹下) 我们试试
root@ceph57 ~ # vim ~/.bashrc
# by miao
alias sl='ls'

# bash 或是 source ~/.bashrc
root@ceph57 ~ # bash
root@ceph57 ~ # sl
anaconda-ks.cfg  calico.yaml   common.yaml  rook       summiao      zhp
br_netfilter     cluster.yaml  hello.txt    set-proxy  unset-proxy

# 其次 还有 ~/.vimrc 

# 此外，还可以修改字体 —— font
vim ~/.config/alacritty
size: 40.0 -> 18.0


```



