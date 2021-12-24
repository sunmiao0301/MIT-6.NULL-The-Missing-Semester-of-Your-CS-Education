### 使用Shell — 以下在Linux - centos7上进行

*—— 见Supplement from video部分*

为了充分利用计算机的能力，我们不得不回到最根本的方式，使用文字接口：Shell

我们会使用 Bourne Again SHell, 简称 “bash” 。 

这是被最广泛使用的一种 shell，它的语法和其他的 shell 都是类似的。

启动终端 可以看到

``` bash
msun@ubuntu:~$ 
```

这一段包含了如下信息：

- 您的主机名是msun
- 当然工作目录是 ~ （这代表 home）
- $ 说明您的身份不是root用户

输入```date```命令可以被解析如下：

```bash
msun@ubuntu:~$ date
Thu Dec 16 10:25:58 CST 2021
```

命令可以附带参数 如 ```echo``` 命令 参数为 ```hello```

```bash
msun@ubuntu:~$ echo hello
hello
```

如果您希望传递的参数中包含空格（例如一个名为 My Photos 的文件夹），

您要么用

- 使用单引号，双引号将其包裹起来，

- 要么使用转义符号 `\` 进行处理（`My\ Photos`）。

```bash
msun@ubuntu:~$ echo hello\ world
hello world
msun@ubuntu:~$ echo hello world
hello world
msun@ubuntu:~$ echo hello" "world
hello world
msun@ubuntu:~$ echo hello' 'world
hello world
```

Shell是一个编程环境，所以它具备变量、条件、循环和函数，当你要求Shell去执行上面的date echo命令的时候，我们实际上是在执行一个Shell可以解释执行的简短代码

如果你要求 shell 执行某个指令，但是该指令并不是 shell 所了解的编程关键字，那么它会去咨询 *环境变量* `$PATH`，它会列出当 shell 接到某条指令时，进行程序搜索的路径。当我们执行 `echo` 命令时，shell 了解到需要执行 `echo` 这个程序，随后它便会在 `$PATH` 中搜索由 `:` 所分割的一系列目录，基于名字搜索该程序。当找到该程序时便执行。

```bash
msun@ubuntu:~$ echo $PATH
/home/msun/bin:/home/msun/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
```

确定某个程序名代表的是哪个具体的程序，可以使用 `which` 程序。我们也可以绕过 `$PATH`，通过直接指定需要执行的程序的路径来执行该程序

```bash
msun@ubuntu:~$ which echo
/bin/echo

hfut_@DESKTOP-HA2C4BG MINGW64 ~
$ /usr/bin/echo $PATHmsun@ubuntu:~$ /bin/echo $PATH
/home/msun/bin:/home/msun/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
```

### 在Shell中导航

shell 中的路径是一组被分割的目录，在 Linux 和 macOS 上使用 `/` 分割，而在Windows上是 `\`。

对于Linux**路径 `/` 代表的是系统的根目录，所有的文件夹都包括在这个路径之下。**

在Windows上每个盘都有一个根目录（例如： `C:\`）。

**我们假设您在学习本课程时使用的是 Linux 文件系统。如果某个路径以 `/` 开头，那么它是一个 *绝对路径*，其他的都是 *相对路径*。 *绝对路径*就是文件的真正存在的路径，是指从硬盘的根目录(盘符)开始，进行一级级目录指向文件。**

我们知道linux里面存在从根目录/走出的如下路径：/home/usr，所以：

```bash
## 此时 我们可以往home走 走的办法既可以是 home 也可以是/home（当然/home肯定是可以的） 但是对于目前所在的根目录而言，home也可以是相对路径 如下
msun@ubuntu:/$ cd home
msun@ubuntu:/home$ 
msun@ubuntu:/$ cd /home
msun@ubuntu:/home$ 
## 然后想要直接走到home下的msun，直接用相对路径就不行了
msun@ubuntu:/$ cd msun
-bash: cd: msun: No such file or directory
## 但是可以走绝对路径
msun@ubuntu:/$ cd /home/msun
msun@ubuntu:~$ 
## home仍是相对的 所以cd home/msun还是可以的
msun@ubuntu:/$ cd home/msun
msun@ubuntu:~$ 
```

**一言蔽之，绝对路径可以到处走，相对路径局限性大一点**

**相对路径是指相对于当前工作目录的路径，当前工作目录可以使用 `pwd` 命令来获取 —— Print working directory 显示当前工作路径。**

```bash
msun@ubuntu:~$ pwd
/home/msun
```

**此外，切换目录需要使用 `cd` 命令。在路径中，`.` 表示的是当前目录，而 `..` 表示上级目录：**

那么我们可以通过```cd ..```回到上级：

```bash
msun@ubuntu:/home$ cd msun
msun@ubuntu:~$ cd ..
msun@ubuntu:/home$ 
## 使用.
msun@ubuntu:/$ cd ./home
msun@ubuntu:/home$ 
```

注意，shell 会实时显示当前的路径信息。您可以通过配置 shell 提示符来显示各种有用的信息，这一内容我们会在后面的课程中进行讨论。

一般来说，当我们运行一个程序时，如果我们没有指定路径，则该程序会在当前目录下执行。例如，我们常常会搜索文件，并在需要时创建文件。为了查看指定目录下包含哪些文件，我们使用 `ls` 命令：

```bash
msun@ubuntu:/$ ls
bin       dev    gdata   ghome?  gshare   initrd.img  lost+found  opt       public  sbin  sys  var
boot      etc    gdata?  gpai    gshare?  lib         media       proc      root    snap  tmp  vmlinuz
datasets  gbita  ghome   gpai?   home     lib64       mnt         projects  run     srv   usr
```

除非我们利用第一个参数指定目录，否则 `ls` 会打印当前目录下的文件。这个参数 /home可以打印出更加详细地列出根目目录下文件或文件夹的信息。

```bash
msun@ubuntu:/$ ls -l /home
total 180
drwxr-xr-x 24 ddcui    ddcui    4096 Dec 15 17:50 ddcui
drwxr-xr-x  4 mzhang   mzhang   4096 Jan 21  2019 dlcheng
drwxr-xr-x 31 fchang   fchang   4096 Sep 24 10:20 fchang
drwxr-xr-x  4 fou      fou      4096 Dec 15 17:51 fou
...
## 首先，本行第一个字符 d 表示 missing 是一个目录。
然后接下来的九个字符，每三个字符构成一组。 
（rwx）（r-x）（r-x）
它们分别代表了文件所有者（ddcui），用户组（users） 以及其他所有人具有的权限。
其中 - 表示该用户不具备相应的权限。
从上面的信息来看，只有文件所有者可以修改（w），ddcui 文件夹 （例如，添加或删除文件夹中的文件）。为了进入某个文件夹，用户需要具备该文件夹以及其父文件夹的“搜索”权限（以“可执行”：x）权限表示。为了列出它的包含的内容，用户必须对该文件夹具备读权限（r）。对于文件来说，权限的意义也是类似的。注意，/bin 目录下的程序在最后一组，即表示所有人的用户组中，均包含 x 权限，也就是说任何人都可以执行这些程序。
```

大多数的命令接受标记和选项（带有值的标记），它们以 `-` 开头，并可以改变程序的行为。通常，在执行程序时使用 `-h` 或 `--help` 标记可以打印帮助信息，以便了解有哪些可用的标记或选项。例如，`ls --help` 的输出如下：

```bash
msun@ubuntu:~$ ls --help
Usage: ls [OPTION]... [FILE]...
List information about the FILEs (the current directory by default).
Sort entries alphabetically if none of -cftuvSUX nor --sort is specified.

Mandatory arguments to long options are mandatory for short options too.
  -a, --all                  do not ignore entries starting with .
  -A, --almost-all           do not list implied . and ..
      --author               with -l, print the author of each file
  -b, --escape               print C-style escapes for nongraphic characters
      --block-size=SIZE      scale sizes by SIZE before printing them; e.g.,
                               '--block-size=M' prints sizes in units of
                               1,048,576 bytes; see SIZE format below
  -B, --ignore-backups       do not list implied entries ending with ~
  -c                         with -lt: sort by, and show, ctime (time of last
                               modification of file status information);
                               with -l: show ctime and sort by name;
                               otherwise: sort by ctime, newest first
  -C                         list entries by columns
      --color[=WHEN]         colorize the output; WHEN can be 'always' (default
                               if omitted), 'auto', or 'never'; more info below
  -d, --directory            list directories themselves, not their contents
  -D, --dired                generate output designed for Emacs' dired mode
  -f                         do not sort, enable -aU, disable -ls --color
  -F, --classify             append indicator (one of */=>@|) to entries
      --file-type            likewise, except do not append '*'
      --format=WORD          across -x, commas -m, horizontal -x, long -l,
                               single-column -1, verbose -l, vertical -C
      --full-time            like -l --time-style=full-iso
  -g                         like -l, but do not list owner
      --group-directories-first
                             group directories before files;
                               can be augmented with a --sort option, but any
                               use of --sort=none (-U) disables grouping
  -G, --no-group             in a long listing, don't print group names
  -h, --human-readable       with -l and/or -s, print human readable sizes
                               (e.g., 1K 234M 2G)
      --si                   likewise, but use powers of 1000 not 1024
  -H, --dereference-command-line
                             follow symbolic links listed on the command line
      --dereference-command-line-symlink-to-dir
                             follow each command line symbolic link
                               that points to a directory
      --hide=PATTERN         do not list implied entries matching shell PATTERN
                               (overridden by -a or -A)
      --indicator-style=WORD  append indicator with style WORD to entry names:
                               none (default), slash (-p),
                               file-type (--file-type), classify (-F)
  -i, --inode                print the index number of each file
  -I, --ignore=PATTERN       do not list implied entries matching shell PATTERN
  -k, --kibibytes            default to 1024-byte blocks for disk usage
  -l                         use a long listing format
  -L, --dereference          when showing file information for a symbolic
                               link, show information for the file the link
                               references rather than for the link itself
  -m                         fill width with a comma separated list of entries
  -n, --numeric-uid-gid      like -l, but list numeric user and group IDs
  -N, --literal              print raw entry names (don't treat e.g. control
                               characters specially)
  -o                         like -l, but do not list group information
  -p, --indicator-style=slash
                             append / indicator to directories
  -q, --hide-control-chars   print ? instead of nongraphic characters
      --show-control-chars   show nongraphic characters as-is (the default,
                               unless program is 'ls' and output is a terminal)
  -Q, --quote-name           enclose entry names in double quotes
      --quoting-style=WORD   use quoting style WORD for entry names:
                               literal, locale, shell, shell-always,
                               shell-escape, shell-escape-always, c, escape
  -r, --reverse              reverse order while sorting
  -R, --recursive            list subdirectories recursively
  -s, --size                 print the allocated size of each file, in blocks
  -S                         sort by file size, largest first
      --sort=WORD            sort by WORD instead of name: none (-U), size (-S),
                               time (-t), version (-v), extension (-X)
      --time=WORD            with -l, show time as WORD instead of default
                               modification time: atime or access or use (-u);
                               ctime or status (-c); also use specified time
                               as sort key if --sort=time (newest first)
      --time-style=STYLE     with -l, show times using style STYLE:
                               full-iso, long-iso, iso, locale, or +FORMAT;
                               FORMAT is interpreted like in 'date'; if FORMAT
                               is FORMAT1<newline>FORMAT2, then FORMAT1 applies
                               to non-recent files and FORMAT2 to recent files;
                               if STYLE is prefixed with 'posix-', STYLE
                               takes effect only outside the POSIX locale
  -t                         sort by modification time, newest first
  -T, --tabsize=COLS         assume tab stops at each COLS instead of 8
  -u                         with -lt: sort by, and show, access time;
                               with -l: show access time and sort by name;
                               otherwise: sort by access time, newest first
  -U                         do not sort; list entries in directory order
  -v                         natural sort of (version) numbers within text
  -w, --width=COLS           set output width to COLS.  0 means no limit
  -x                         list entries by lines instead of by columns
  -X                         sort alphabetically by entry extension
  -Z, --context              print any security context of each file
  -1                         list one file per line.  Avoid '\n' with -q or -b
      --help     display this help and exit
      --version  output version information and exit

The SIZE argument is an integer and optional unit (example: 10K is 10*1024).
Units are K,M,G,T,P,E,Z,Y (powers of 1024) or KB,MB,... (powers of 1000).

Using color to distinguish file types is disabled both by default and
with --color=never.  With --color=auto, ls emits color codes only when
standard output is connected to a terminal.  The LS_COLORS environment
variable can change the settings.  Use the dircolors command to set it.

Exit status:
 0  if OK,
 1  if minor problems (e.g., cannot access subdirectory),
 2  if serious trouble (e.g., cannot access command-line argument).

GNU coreutils online help: <http://www.gnu.org/software/coreutils/>
Full documentation at: <http://www.gnu.org/software/coreutils/ls>
or available locally via: info '(coreutils) ls invocation'
```

但是```-h```却没有反应

```bash
msun@ubuntu:~$ ls -h
msun@ubuntu:~$ 
```

在这个阶段，还有几个趁手的命令是您需要掌握的，例如 `mv`（用于重命名或移动文件）、 `cp`（拷贝文件）以及 `mkdir`（新建文件夹）。

如果您想要知道关于程序参数、输入输出的信息，亦或是想要了解它们的工作方式，请试试 `man` 这个程序。它会接受一个程序名作为参数，然后将它的文档（用户手册）展现给您。注意，使用 `q` 可以退出该程序。

```bash
msun@ubuntu:~$ man ls

LS(1)                                           User Commands                                           LS(1)

NAME
       ls - list directory contents

SYNOPSIS
       ls [OPTION]... [FILE]...

DESCRIPTION
       List  information  about the FILEs (the current directory by default).  Sort entries alphabetically if
       none of -cftuvSUX nor --sort is specified.

       Mandatory arguments to long options are mandatory for short options too.

       -a, --all
              do not ignore entries starting with .

       -A, --almost-all
              do not list implied . and ..

       --author
              with -l, print the author of each file

       -b, --escape
              print C-style escapes for nongraphic characters

       --block-size=SIZE
              scale sizes by SIZE before printing them; e.g.,  '--block-size=M'  prints  sizes  in  units  of
              1,048,576 bytes; see SIZE format below

       -B, --ignore-backups
              do not list implied entries ending with ~

       -c     with -lt: sort by, and show, ctime (time of last modification of file status information); with
              -l: show ctime and sort by name; otherwise: sort by ctime, newest first

       -C     list entries by columns

       --color[=WHEN]
              colorize the output; WHEN can be 'always' (default if omitted), 'auto', or 'never';  more  info
              below

       -d, --directory
              list directories themselves, not their contents

       -D, --dired
              generate output designed for Emacs' dired mode
```

### 在程序间创建连接

在 shell 中，程序有两个主要的“流”：它们的输入流和输出流。 当程序尝试读取信息时，它们会从输入流中进行读取，当程序打印信息时，它们会将信息输出到输出流中。 通常，一个程序的输入输出流都是您的终端。也就是，您的键盘作为输入，显示器作为输出。 但是，我们也可以重定向这些流！

最简单的重定向是 `< file` 和 `> file`。这两个命令可以将程序的输入输出流分别重定向到文件：

```bash
msun@ubuntu:~$ echo hello > hello.txt
msun@ubuntu:~$ cat hello.txt
hello
msun@ubuntu:~$ cat < hello.txt
hello
msun@ubuntu:~$ cat < hello.txt > hello2.txt
msun@ubuntu:~$ cat hello2.txt
hello
```



您还可以使用 `>>` 来向一个文件追加内容。使用管道（ *pipes* ），我们能够更好的利用文件重定向。 **`|` 操作符允许我们将一个程序的输出和另外一个程序的输入连接起来：**

```
missing:~$ ls -l / | tail -n1
drwxr-xr-x 1 root  root  4096 Jun 20  2019 var
missing:~$ curl --head --silent google.com | grep --ignore-case content-length | cut --delimiter=' ' -f2
219
```

我们会在数据清理一章中更加详细的探讨如何更好的利用管道。

### 一个功能全面又强大的工具

对于大多数的类 Unix 系统，有一类用户是非常特殊的，那就是：根用户（root user）。 您应该已经注意到了，在上面的输出结果中，根用户几乎不受任何限制，他可以创建、读取、更新和删除系统中的任何文件。 通常在我们并不会以根用户的身份直接登录系统，因为这样可能会因为某些错误的操作而破坏系统。 取而代之的是我们会在需要的时候使用 `sudo` 命令。顾名思义，它的作用是让您可以以 su（super user 或 root 的简写）的身份执行一些操作。 当您遇到拒绝访问（permission denied）的错误时，通常是因为此时您必须是根用户才能操作。然而，请再次确认您是真的要执行此操作。

有一件事情是您必须作为根用户才能做的，那就是向 `sysfs` 文件写入内容。系统被挂载在 `/sys` 下，`sysfs` 文件则暴露了一些内核（kernel）参数。 因此，您不需要借助任何专用的工具，就可以轻松地在运行期间配置系统内核。**注意 Windows 和 macOS 没有这个文件**

例如，您笔记本电脑的屏幕亮度写在 `brightness` 文件中，它位于

```
/sys/class/backlight
```

通过将数值写入该文件，我们可以改变屏幕的亮度。现在，蹦到您脑袋里的第一个想法可能是：

```
$ sudo find -L /sys/class/backlight -maxdepth 2 -name '*brightness*'
/sys/class/backlight/thinkpad_screen/brightness
$ cd /sys/class/backlight/thinkpad_screen
$ sudo echo 3 > brightness
An error occurred while redirecting file 'brightness'
open: Permission denied
```

出乎意料的是，我们还是得到了一个错误信息。毕竟，我们已经使用了 `sudo` 命令！关于 shell，有件事我们必须要知道。`|`、`>`、和 `<` 是通过 shell 执行的，而不是被各个程序单独执行。 `echo` 等程序并不知道 `|` 的存在，它们只知道从自己的输入输出流中进行读写。 对于上面这种情况， *shell* (权限为您的当前用户) 在设置 `sudo echo` 前尝试打开 brightness 文件并写入，但是系统拒绝了 shell 的操作因为此时 shell 不是根用户。

明白这一点后，我们可以这样操作：

```
$ echo 3 | sudo tee brightness
```

因为打开 `/sys` 文件的是 `tee` 这个程序，并且该程序以 `root` 权限在运行，因此操作可以进行。 这样您就可以在 `/sys` 中愉快地玩耍了，例如修改系统中各种LED的状态（路径可能会有所不同）：

```
$ echo 1 | sudo tee /sys/class/leds/input6::scrolllock/brightness
```

### HOMEWORK

1.本课程需要使用类Unix shell，例如 Bash 或 ZSH。如果您在 Linux 或者 MacOS 上面完成本课程的练习，则不需要做任何特殊的操作。如果您使用的是 Windows，则您不应该使用 cmd 或是 Powershell；您可以使用[Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/)或者是 Linux 虚拟机。使用`echo $SHELL`命令可以查看您的 shell 是否满足要求。如果打印结果为`/bin/bash`或`/usr/bin/zsh`则是可以的。

2.在 `/tmp` 下新建一个名为 `missing` 的文件夹。

```bash
## 首先是寻找文件夹 我们先拿hello.txt试试
msun@ubuntu:~$ find / -name hello.txt
find: ‘/run/docker’: Permission denied
find: ‘/run/lxcfs’: Permission denied
find: ‘/run/user/1029’: Permission denied
find: ‘/run/user/1020’: Permission denied
find: ‘/run/user/1002’: Permission denied
find: ‘/run/user/1013’: Permission denied
find: ‘/run/user/1022’: Permission denied
find: ‘/run/user/1030’: Permission denied
find: ‘/run/user/1023’: Permission denied
find: ‘/run/user/1032’: Permission denied
find: ‘/run/user/1006’: Permission denied
find: ‘/run/user/1035’: Permission denied
find: ‘/run/user/1033’: Permission denied
find: ‘/run/user/1009’: Permission denied
find: ‘/run/user/1031’: Permission denied
find: ‘/run/user/1010’: Permission denied
find: ‘/run/user/1001’: Permission denied
find: ‘/run/user/1024’: Permission denied
find: ‘/run/user/1014’: Permission denied
find: ‘/run/sudo’: Permission denied
find: ‘/run/screen/S-mzhang’: Permission denied
find: ‘/run/log/journal/eeb4082cb608967c89094bad5b20107e’: Permission denied
find: ‘/run/lvm’: Permission denied
find: ‘/run/systemd/ask-password-block’: Permission denied
find: ‘/run/systemd/inaccessible’: Permission denied
find: ‘/run/lock/lvm’: Permission denied
...
## 在根目录/下find /tmp文件，denied太多，换成.(也就是当前目录/home/msun)即可
msun@ubuntu:~$ find . -name hello.txt
./hello.txt
## 那么下面就是find /tmp

## 然后进入/tmp

## 然后新建文件夹
msun@ubuntu:/tmp$ mkdir missing
```



![在根目录/下find /tmp文件夹deny太多，换成.(也就是当前目录/home/msun)即可 ](https://raw.githubusercontent.com/sunmiao0301/Public-Pic-Bed/main/211216zhp1.png)

3.用 `man` 查看程序 `touch` 的使用手册。

```bash
msun@ubuntu:~$ man touch
TOUCH(1)                                        User Commands                                        TOUCH(1)

NAME
       touch - change file timestamps

SYNOPSIS
       touch [OPTION]... FILE...

DESCRIPTION
       Update the access and modification times of each FILE to the current time.

       A FILE argument that does not exist is created empty, unless -c or -h is supplied.

       A  FILE  argument  string  of  - is handled specially and causes touch to change the times of the file
       associated with standard output.

       Mandatory arguments to long options are mandatory for short options too.

       -a     change only the access time

       -c, --no-create
              do not create any files

       -d, --date=STRING
              parse STRING and use it instead of current time

       -f     (ignored)

```

4.用 `touch` 在 `missing` 文件夹中新建一个叫 `semester` 的文件。

```bash
msun@ubuntu:/tmp$ touch
```

5.将以下内容一行一行地写入semester文件：

```
 #!/bin/sh
 curl --head --silent https://missing.csail.mit.edu
```

第一行可能有点棘手， `#` 在Bash中表示注释，而 `!` 即使被双引号（`"`）包裹也具有特殊的含义。 单引号（`'`）则不一样，此处利用这一点解决输入问题。更多信息请参考 [Bash quoting 手册](https://www.gnu.org/software/bash/manual/html_node/Quoting.html)

```bash
msun@ubuntu:/tmp/missing$ touch semester
msun@ubuntu:/tmp/missing$ ls
semester
## 前面提到 您还可以使用 >> 来向一个文件追加内容 
## 其中，<<EOF 表示当遇到EOF时结束输入。
## echo写入单行
## cat写入多行
## 对于# 我们通过转义字符\解决 未引用的反斜杠 '\' 是 Bash 转义字符。它保留了下一个字符的字面值，除了newline. 如果\newline出现一对，并且反斜杠本身没有被引用，则将\newline 被视为行继续（即，它从输入流中删除并有效地被忽略）。
## 对于! 将字符括在单引号 (''') 保留引号内每个字符的字面值。单引号之间不能出现单引号，即使前面有反斜杠。
## 相比较 双引号的保留字符字面值的能力不如单引号
## 在这里我发现echo写入单行的时候需要用到转义符号 单引号 来完成 
## 但是如果是cat写入多行 那么直接写就行了 对比如下：
## 首先是echo
msun@ubuntu:/tmp/missing$ touch semester_echo
msun@ubuntu:/tmp/missing$ echo \#'!'/bin/sh > semester_echo
## 注意第一行的时候是 >
## 第二行的时候是 >> （追加内容）
msun@ubuntu:/tmp/missing$ echo curl --head --silent https://missing.csail.mit.edu >> semester_echo
msun@ubuntu:/tmp/missing$ vim semester_echo
#!/bin/sh
curl --head --silent https://missing.csail.mit.edu
~                                                      
## 然后是cat
msun@ubuntu:/tmp/missing$ vim semester_cat
msun@ubuntu:/tmp/missing$ cat>semester_cat<<EOF
#!/bin/sh
curl --head --silent https://missing.csail.mit.edu
EOF
msun@ubuntu:/tmp/missing$ vim semester_cat
#!/bin/sh
curl --head --silent https://missing.csail.mit.edu
```

6.尝试执行这个文件。例如，将该脚本的路径（`./semester`）输入到您的shell中并回车。如果程序无法执行，请使用 `ls` 命令来获取信息并理解其不能执行的原因。

```bash
msun@ubuntu:/tmp/missing$ ./semester_cat
-bash: ./semester_cat: Permission denied
## 通过ls -l可以看出是因为权限不够 没有x权限
msun@ubuntu:/tmp/missing$ ls -l
total 20
-rw-rw-r-- 1 msun msun 10 Dec 16 13:06 semester
-rw-rw-r-- 1 msun msun 61 Dec 16 13:12 semester_cat
-rw-rw-r-- 1 msun msun 61 Dec 16 13:10 semester_echo
-rw-rw-r-- 1 msun msun 61 Dec 16 13:04 semeter
-rw-rw-r-- 1 msun msun 61 Dec 16 13:11 semeter_cat
```

7.查看 `chmod` 的手册(例如，使用 `man chmod` 命令)

```bash
msun@ubuntu:/tmp/missing$ man chmod
```

8.使用 `chmod` 命令改变权限，使 `./semester` 能够成功执行，不要使用 `sh semester` 来执行该程序。您的 shell 是如何知晓这个文件需要使用 `sh` 来解析呢？更多信息请参考：[shebang](https://en.wikipedia.org/wiki/Shebang_(Unix))

```bash
## 只有文件所有者和超级用户可以修改文件或目录的权限。可以使用绝对模式（八进制数字模式），符号模式指定文件的权限。
## 语法
## chmod [-cfvR] [--help] [--version] mode file...

## 参数说明
## mode : 权限设定字串，格式如下 :
## [ugoa...][[+-=][rwxX]...][,...]

## 其中：
## u 表示该文件的拥有者，g 表示与该文件的拥有者属于同一个群体(group)者，o 表示其他以外的人，a 表示这三者皆是。
## + 表示增加权限、- 表示取消权限、= 表示唯一设定权限。
## r 表示可读取，w 表示可写入，x 表示可执行，X 表示只有当该文件是个子目录或者该文件已经被设定过为可执行。
## 其他参数说明：

## -c : 若该文件权限确实已经更改，才显示其更改动作
## -f : 若该文件权限无法被更改也不要显示错误讯息
## -v : 显示权限变更的详细资料
## -R : 对目前目录下的所有文件与子目录进行相同的权限变更(即以递归的方式逐个变更)
## --help : 显示辅助说明
## --version : 显示版本

## 于是我们将文件的拥有者的权限修改为可运行
msun@ubuntu:/tmp/missing$ chmod u+x semester_cat
## 此时可以成功运行 但是得不到结果 如下
msun@ubuntu:/tmp/missing$ ./semester_cat
curl: (60) server certificate verification failed. CAfile: /etc/ssl/certs/ca-certificates.crt CRLfile: none
More details here: http://curl.haxx.se/docs/sslcerts.html

curl performs SSL certificate verification by default, using a "bundle"
 of Certificate Authority (CA) public keys (CA certs). If the default
 bundle file isn't adequate, you can specify an alternate file
 using the --cacert option.
If this HTTPS server uses a certificate signed by a CA represented in
 the bundle, the certificate verification probably failed due to a
 problem with the certificate (it might be expired, or the name might
 not match the domain name in the URL).
If you'd like to turn off curl's verification of the certificate, use
 the -k (or --insecure) option.
## 由上可知是证书过期了，可以通过 -k解决 （直接执行或者将-k写到semester里面都行）
msun@ubuntu:/tmp/missing$ curl --head https://missing.csail.mit.edu -k
HTTP/1.1 200 OK
Connection: keep-alive
Content-Length: 7985
Server: GitHub.com
Content-Type: text/html; charset=utf-8
Last-Modified: Thu, 09 Dec 2021 04:11:14 GMT
Access-Control-Allow-Origin: *
ETag: "61b181e2-1f31"
expires: Thu, 16 Dec 2021 03:45:01 GMT
Cache-Control: max-age=600
x-proxy-cache: MISS
X-GitHub-Request-Id: 481C:3298:EE5F4:130A4A:61BAB3E5
Accept-Ranges: bytes
Date: Thu, 16 Dec 2021 06:22:22 GMT
Via: 1.1 varnish
Age: 0
X-Served-By: cache-hkg17930-HKG
X-Cache: MISS
X-Cache-Hits: 0
X-Timer: S1639635742.896278,VS0,VE270
Vary: Accept-Encoding
X-Fastly-Request-ID: 76dc2b0d13ba656aec11afdddf673681030f0211
msun@ubuntu:/tmp/missing$ 
## 在计算中，shebang是由脚本开头的字符数字符号和感叹号( #! )组成的字符序列。它也被称为sha-bang,hashbang,pound-bang或hash-pling
## 当带有 shebang 的文本文件被用作类 Unix操作系统中的可执行文件时，程序加载器机制将文件初始行的其余部分解析为解释器指令。加载器执行指定的解释器程序，将尝试运行脚本时最初使用的路径作为参数传递给它，以便程序可以使用文件作为输入数据。例如，如果一个脚本以路径path/to/script命名，并且它以以下行开头，#!/bin/sh，则指示程序加载器运行程序/bin/sh，传递path/to/script作为第一个论点。在Linux 中，这种行为是内核和用户空间代码的结果
```

9.使用 `|` 和 `>` ，将 `semester` 文件输出的最后更改日期信息，写入主目录下的 `last-modified.txt` 的文件中

```bash
## 根目录：／是树状形式目录的根，只有一个
## 用户目录：主目录是用户的HOME目录，是在添加用户的时候指定的
msun@ubuntu:/tmp/missing$ touch last-modified.txt
msun@ubuntu:/tmp/missing$ curl --head https://missing.csail.mit.edu -k | grep "Last-Modified" > last_modified.txt
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0  7985    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
msun@ubuntu:/tmp/missing$ cat last_modified.txt
Last-Modified: Thu, 09 Dec 2021 04:11:14 GMT

```

10.写一段命令来从 `/sys` 中获取笔记本的电量信息，或者台式机 CPU 的温度。注意：macOS 并没有 sysfs，所以 Mac 用户可以跳过这一题。

```bash
## 虽然我是Linux 但我也没找到电量信息文件。
```

### Supplement from video

```bash
## mv
msun@ceph57 ~ $ touch dotfiles.md
msun@ceph57 ~ $ ls
dotfiles.md
msun@ceph57 ~ $ mv dotfiles.md foo.md
msun@ceph57 ~ $ ls
foo.md

## cp
msun@ceph57 ~ $ cp foo.md dotfiles.md
msun@ceph57 ~ $ ls
dotfiles.md  foo.md

## rm
## 在linux上，删除是default而不是递归（recursive）的，所以不能直接删除一个文件夹。但是可以使用 -r 来完成递归删除
msun@ceph57 ~ $ rm foo.md
rm: remove regular empty file ‘foo.md’? yes
msun@ceph57 ~ $ ls
dotfiles.md

## mkdir
## 注意不要用 mkdir My Photos 因为这样会创建两个文件，一个叫My 一个叫Photos，如果想创建这样的名字的文件，可以用 mkdir "My Photos"

## man = show manual page of sth, Q to exit
msun@ceph57 ~ $ man ls

## tail
## 显示输出的最后几行
## 如下就是显示根目录文件按行输出时的最后1行
msun@ceph57 ~ $ ls -l / | tail -n1
drwxr-xr-x.  19 root root  267 Nov 30 14:10 var

## 管道配合输入输出< >
msun@ceph57 ~ $ ls -l / | tail -n1 > ls.txt
msun@ceph57 ~ $ cat ls.txt
drwxr-xr-x.  19 root root  267 Nov 30 14:10 var

## 奇技淫巧
jon@xpanse intel_backlight ~ $ echo 1060 | sudo tee brightness
```





