### shell脚本 — 进阶

*—— 课后练习3未作*

上一节课，我们已经学习来如何在shell中执行命令，并使用管道将命令组合使用。但是，很多情况下我们需要执行一系列的操作并使用条件或循环这样的控制流。

由此，shell 脚本应运而生，也更加复杂。

大多数shell都有自己的一套脚本语言，包括变量、控制流和自己的语法。shell脚本与其他脚本语言不同之处在于，shell脚本针对shell所从事的相关工作进行来优化。因此，创建命令流程（pipelines）、将结果保存到文件、从标准输入中读取输入，这些都是shell脚本中的原生操作，这让它比通用的脚本语言更易用。本节中，我们会专注于bash脚本，因为它最流行，应用更为广泛。**一言蔽之，shell脚本相较于其他脚本，对于解决shell相关工作中的问题更具有针对性。**

注意：

- 在bash中为变量赋值的语法是`foo=bar`
- 需要注意的是，`foo = bar` （使用空格隔开）是不能正确工作的，因为解释器会调用程序`foo` 并将 `=` 和 `bar`作为参数。但是，在shell脚本中使用空格会起到分割参数的作用，有时候可能会造成混淆，请务必多加检查。（这对于习惯高级语言编程并且有着良好编程习惯的用户来说是不友好的）
- 访问变量中存储的数值，其语法为 `$foo`。 

Bash中的字符串通过`'` 和 `"`分隔符来定义，但是它们的含义并不相同：

- 以`'`定义的字符串为原义字符串，其中的变量不会被转义。

- 以 `"`定义的字符串会将变量值进行替换。

**一言蔽之，单引号使得其写的是什么就打印什么，双引号不行。如下**

```bash
msun@ceph57 ~ $ echo "$foo"
bar
msun@ceph57 ~ $ echo '$foo'
$foo
```

e·g:

```bash
foo=bar
echo "$foo"
# 打印 bar
echo '$foo'
# 打印 $foo
```

和其他大多数的编程语言一样，`bash`也支持`if`, `case`, `while` 和 `for` 这些控制流关键字。同样地， `bash` 也支持函数，它可以接受参数并基于参数进行操作。下面这个函数是一个例子，它会创建一个文件夹并使用`cd`进入该文件夹。

```bash
mcd () {
    mkdir -p "$1"
    cd "$1"
}
```

```bash
## -p 确保目录名存在，不存在则新建一个。
## 一般用法是
mkdir -p runoob2/test
## 在工作目录下的 runoob2 目录中，建立一个名为 test 的子目录。
## 若 runoob2 目录原本不存在，则建立一个。（注：本例若不加 -p 参数，且原本 runoob2 目录不存在，则产生错误。）
```

与其他脚本语言不同的是，bash使用了很多特殊的变量来表示参数、错误代码和相关变量。下面是列举来其中一些变量，更完整的列表可以参考 [这里](https://www.tldp.org/LDP/abs/html/special-chars.html)。

- `$0` - 脚本名
- `$1` 到 `$9` - 脚本的参数。 `$1` 是第一个参数，依此类推。
- `$@` - 所有参数
- `$#` - 参数个数
- `$?` - 前一个命令的返回值

```bash
msun@ceph57 ~ $ echo "hello"
hello
# 因为上一条命令没有任何问题 零退出代表一切正常 没有错误
msun@ceph57 ~ $ echo $?
0
# 比如
msun@ceph57 ~ $ true
msun@ceph57 ~ $ echo $?
0
msun@ceph57 ~ $ false
msun@ceph57 ~ $ echo $?
1
```

- `$$` - 当前脚本的进程识别码
- `!!` - 完整的上一条命令，包括参数。常见应用：当你因为权限不足执行命令失败时，可以使用 `sudo !!`再尝试一次。

```bash
msun@ceph57 ~ $ tree .
# 会提示没有权限 then u can
msun@ceph57 ~ $ sudo !!
# !! 会被替换为上一条命令 
```

- `$_` - 上一条命令的最后一个参数。如果你正在使用的是交互式shell，你可以通过按下 `Esc` 之后键入 . 来获取这个值。

```bash
msun@ceph57 ~ $ mkdir test
msun@ceph57 ~ $ cd $_
msun@ceph57 ~/test $ 
```

命令通常使用 `STDOUT`来返回输出值，使用`STDERR` 来返回错误及错误码，便于脚本以更加友好的方式报告错误。 返回码或退出状态是脚本/命令之间交流执行状态的方式。返回值0表示正常执行，其他所有非0的返回值都表示有错误发生。

退出码可以搭配`&&` (与操作符) 和 `||` (或操作符)使用，用来进行条件判断，决定是否执行其他程序。 同一行的多个命令可以用` ; `分隔。程序 `true` 的返回码永远是`0`，`false` 的返回码永远是`1`。让我们看几个例子

```bash
## 下面几个例子展示了 || && 作为短路运算符的表现

false || echo "Oops, fail"
## 输出为 Oops, fail
## 因为仅一个false无法提前得到 || 结果 故没有发生短路 echo被执行并输出

true || echo "Will not be printed"
## 没有输出
## 因为 true 短路了这个 || 后面的echo没有能够执行

true && echo "Things went well"
## Things went well
## 同理 仅一个 true 无法得到 && 运算的结果 echo得以执行

false && echo "Will not be printed"
## false 短路了 && 运算 echo得不到执行

false ; echo "This will always run"
## This will always run
## 同一行的多个命令可以用 ; 分隔
```

**另一个常见的模式是以变量的形式获取一个命令的输出，这可以通过 *命令替换* (*command substitution*)实现。**当您通过 `$( CMD )` 这样的方式来执行`CMD` 这个命令时，它的输出结果会替换掉 `$( CMD )` 。例如，如果执行 `for file in $(ls)` ，shell首先将调用`ls` ，然后遍历得到的这些返回值。

```bash
msun@ceph57 ~ $ foo=$(pwd)
msun@ceph57 ~ $ echo $foo
/home/msun
msun@ceph57 ~ $ echo "We are in $(pwd)"
We are in /home/msun
```

还有一个冷门的类似特性是 *进程替换*（*process substitution*）， `<( CMD )` 会执行 `CMD` 并将结果输出到一个临时文件中，并将 `<( CMD )` 替换成临时文件名。这在我们希望返回值通过文件而不是STDIN传递时很有用。例如， `diff <(ls foo) <(ls bar)` 会显示文件夹 `foo` 和 `bar` 中文件的区别。

下面开始实例脚本：

遍历我们提供的参数，使用`grep` 搜索字符串 `foobar`，如果没有找到，则将其作为注释追加到文件中。

```bash
#!/bin/bash

echo "Starting program at $(date)" # date会被替换成日期和时间

echo "Running program $0 with $# arguments with pid $$"

for file in "$@"; do
    grep foobar "$file" > /dev/null 2> /dev/null
    # 如果模式没有找到，则grep退出状态为 1
    # 我们将标准输出流和标准错误流重定向到Null，因为我们并不关心这些信息
    if [[ $? -ne 0 ]]; then
        echo "File $file does not have any foobar, adding one"
        echo "# foobar" >> "$file"
    fi
done
```

在条件语句中，我们比较 `$?` 是否等于0。 Bash实现了许多类似的比较操作，您可以查看 [`test 手册`](https://man7.org/linux/man-pages/man1/test.1.html)。 在bash中进行比较时，尽量使用双方括号 `[[ ]]` 而不是单方括号 `[ ]`，这样会降低犯错的几率，尽管这样并不能兼容 `sh`。 更详细的说明参见[这里](http://mywiki.wooledge.org/BashFAQ/031)。

当执行脚本时，我们经常需要提供形式类似的参数。bash使我们可以轻松的实现这一操作，它可以基于文件扩展名展开表达式。这一技术被称为shell的 *通配*（ *globbing*）

- 通配符 - 当你想要利用通配符进行匹配时，你可以分别使用 `?` 和 `*` 来匹配一个或任意个字符。例如，对于文件`foo`, `foo1`, `foo2`, `foo10` 和 `bar`,

  - `rm foo?`这条命令会删除`foo1` 和 `foo2` 

  - 而`rm foo*` 则会删除除了`bar`之外的所有文件。

```bash
# 比如我们有如下文件夹
msun@ceph57 ~/pro $ ls
pro1  pro12  pro2
# 然后我们用ls查找相关文件 ?可以通配一个字符 但是*不可以
msun@ceph57 ~/pro $ ls pro*
pro1  pro12  pro2
msun@ceph57 ~/pro $ ls pro?
pro1  pro2
```

- 花括号`{}` - 当你有一系列的指令，其中包含一段公共子串时，可以用花括号来自动展开这些命令。这在批量移动或转换文件时非常方便。

```bash
convert image.{png,jpg}
# 会展开为
convert image.png image.jpg

cp /path/to/project/{foo,bar,baz}.sh /newpath
# 会展开为
cp /path/to/project/foo.sh /path/to/project/bar.sh /path/to/project/baz.sh /newpath

# 也可以结合通配使用
mv *{.py,.sh} folder
# 会移动所有 *.py 和 *.sh 文件

# 下面命令会创建foo/a, foo/b, ... foo/h, bar/a, bar/b, ... bar/h这些文件
touch {foo,bar}/{a..h} # 参见笛卡尔积

# 比较文件夹 foo 和 bar 中包含文件的不同
touch foo/x bar/y
diff <(ls foo) <(ls bar)
# 输出
# < x
# ---
# > y
```

编写 `bash` 脚本有时候会很别扭和反直觉。例如 [shellcheck](https://github.com/koalaman/shellcheck)这样的工具可以帮助你定位sh/bash脚本中的错误。

注意，脚本并不一定只有用bash写才能在终端里调用。比如说，这是一段Python脚本，作用是将输入的参数倒序输出：

```bash
#!/usr/local/bin/python
import sys
for arg in reversed(sys.argv[1:]):
    print(arg)
```

内核知道去用python解释器而不是shell命令来运行这段脚本，是因为脚本的开头第一行的[shebang](https://en.wikipedia.org/wiki/Shebang_(Unix))。

在 `shebang` 行中使用 [`env`](https://man7.org/linux/man-pages/man1/env.1.html) 命令是一种好的实践，它会利用环境变量中的程序来解析该脚本，这样就提高来您的脚本的可移植性。`env` 会利用我们第一节讲座中介绍过的`PATH` 环境变量来进行定位。 例如，使用了`env`的shebang看上去时这样的`#!/usr/bin/env python`。

shell函数和脚本有如下一些不同点：

- 函数只能用与shell使用相同的语言，脚本可以使用任意语言。因此在脚本中包含 `shebang` 是很重要的。
- 函数仅在定义时被加载，脚本会在每次被执行时加载。这让函数的加载比脚本略快一些，但每次修改函数定义，都要重新加载一次。
- 函数会在当前的shell环境中执行，脚本会在单独的进程中执行。因此，函数可以对环境变量进行更改，比如改变当前工作目录，脚本则不行。脚本需要使用 [`export`](httsp://man7.org/linux/man-pages/man1/export.1p.html) 将环境变量导出，并将值传递给环境变量。
- 与其他程序语言一样，函数可以提高代码模块性、代码复用性并创建清晰性的结构。shell脚本中往往也会包含它们自己的函数定义。

### Shell工具

如何找到命令的flag？不需要StackOverflow:

- --help 
- -h 
- man

以上都能够解决问题，比如```man rm```，但是得到的手册往往过于详细，所以我们可以使用一款不错的man替代品[TLDR pages](https://tldr.sh/)，致力于常用的命令，帮助我们更快速的找到正确的选项。

### 查看文件

程序员面临的最常见的重复任务就是查找文件或目录。find是shell上用于查找文件的绝佳工具。`find`命令会递归地搜索符合条件的文件，例如：

```bash
# 目前路径下一个文件夹 d_for_find 两个文件 dotfiles.md  ls.txt
msun@ceph57 ~ $ ls
d_for_find  dotfiles.md  ls.txt

# 查找所有名称为 d_for_find 的文件夹 因为 d - directory
msun@ceph57 ~ $ find . -name d_for_find -type d
./d_for_find

# 查找所有名称为 ls.txt 的文件 因为 f - file
msun@ceph57 ~ $ find -name ls.txt -type f
./ls.txt

# 查找在指定目录/home下所有文件夹路径中包含home的txt文件
## > file将标准输出重定向到文件
## 1> file将标准输出重定向到文件
## 2> file 将标准错误重定向到文件

## &> file将 stdout 和 stderr
## > file 2>&1重定向到文件将 stdout 和 stderr 重定向到文件
## 请注意，这> file 2>&1是一个仍然有效的旧语法，&> file更简洁，但不适用于旧系统。

## /dev/null是空设备，它接受您想要的任何输入并将其丢弃。它可用于抑制任何输出。不然会有太多denied
msun@ceph57 ~ $ cd ..
msun@ceph57 /home $ find .. -path '*/home/*.txt' -type f 2>/dev/null
../home/msun/ls.txt
```

奇技淫巧

```bash
# 查找前一天修改的所有文件
find . -mtime -1

# 查找所有大小在500k至10M的tar.gz文件
find . -size +500k -size -10M -name '*.tar.gz'
```

除了列出所寻找的文件之外，find还能对所有查找到的文件进行操作。这能极大地简化一些单调的任务。

```bash
# 删除全部扩展名为.tmp 的文件
find . -name '*.tmp' -exec rm {} \;
# 查找全部的 PNG 文件并将其转换为 JPG
# 注意convert命令的应用范围：convert  between  image formats as well as resize an image,
find . -name '*.png' -exec convert {} {}.jpg \;
```

尽管 `find` 用途广泛，它的语法却比较难以记忆。

例如，为了查找满足模式 `PATTERN` 的文件，您需要执行 `find -name '*PATTERN*'` (如果您希望模式匹配时是不区分大小写，可以使用`-iname`选项），我们可以寻找更好的替代品。

 [`fd`](https://github.com/sharkdp/fd) 就是一个更简单、更快速、更友好的程序，它可以用来作为`find`的替代品。它有很多不错的默认设置，例如输出着色、默认支持正则匹配、支持unicode并且我认为它的语法更符合直觉。以模式`PATTERN` 搜索的语法是 `fd PATTERN`。

大多数人都认为 `find` 和 `fd` 已经很好用了，但是有的人可能想知道，我们是不是可以有更高效的方法，例如不要每次都搜索文件而是通过编译索引或建立数据库的方式来实现更加快速地搜索。

这就要靠 [`locate`](https://man7.org/linux/man-pages/man1/locate.1.html) 了。 `locate` 使用一个由 [`updatedb`](https://man7.org/linux/man-pages/man1/updatedb.1.html)负责更新的数据库，在大多数系统中 `updatedb` 都会通过 [`cron`](https://man7.org/linux/man-pages/man8/cron.8.html)每日更新。这便需要我们在速度和时效性之间作出权衡。而且，`find` 和类似的工具可以通过别的属性比如文件大小、修改时间或是权限来查找文件，`locate`则只能通过文件名。 [here on stackoverflow](https://unix.stackexchange.com/questions/60205/locate-vs-find-usage-pros-and-cons-of-each-other)有一个更详细的对比。

**以上，一言蔽之，locate会出现错误（关乎数据库更新问题），而find适合linux老手。**

### 查找代码

通过文件名查找文件是很有用的技能，但是很多时候您的目标其实是查看文件的内容。一个最常见的场景是您希望查找具有某种模式的全部文件，并找它们的位置。

为了实现这一点，很多类UNIX的系统都提供了[`grep`](https://man7.org/linux/man-pages/man1/grep.1.html)命令，它是用于对输入文本进行匹配的通用工具。它是一个非常重要的shell工具，我们会在后续的数据清理课程中深入的探讨它。

```bash
msun@ceph57 ~ $ cat ls.txt
drwxr-xr-x.  19 root root  267 Nov 30 14:10 var
msun@ceph57 ~ $ grep Nov ls.txt
drwxr-xr-x.  19 root root  267 Nov 30 14:10 var
```

`grep` 有很多选项，这也使它成为一个非常全能的工具。经常使用的有 

`-C` ：获取查找结果的上下文（Context）；

`-v` 将对结果进行反选（Invert），也就是输出不匹配的结果。

举例来说， `grep -C 5` 会输出匹配结果前后五行。当需要搜索大量文件的时候，使用 `-R` 会递归地进入子目录并搜索所有的文本文件。

但是，我们有很多办法可以对 `grep -R` 进行改进，例如使其忽略`.git` 文件夹，使用多CPU等等。

### 查找自己使用过的命令

您可能想要找到之前输入过的某条命令。首先，按向上的方向键会显示你使用过的上一条命令，继续按上键则会遍历整个历史记录。

`history` 命令允许您以程序员的方式来访问shell中输入的历史命令。这个命令会在标准输出中打印shell中的里面命令。如果我们要搜索历史记录，则可以利用管道将输出结果传递给 `grep` 进行模式搜索。 `history | grep find` 会打印包含find子串的命令。

```bash
msun@ceph57 ~ $ history | grep find
  120  2021-12-17 15:40:41 man find
```

你可以修改 shell history 的行为，例如，如果在命令的开头加上一个空格，它就不会被加进shell记录中。当你输入包含密码或是其他敏感信息的命令时会用到这一特性。 为此你需要在`.bashrc`中添加`HISTCONTROL=ignorespace`或者向`.zshrc` 添加 `setopt HIST_IGNORE_SPACE`。 （不过我这边使用centos好像已经默认了）如果你不小心忘了在前面加空格，可以通过编辑。`bash_history`或 `.zhistory` 来手动地从历史记录中移除那一项。

此外，与历史相关的还有

- Ctrl + R
- ↑ ↓
- Ctrl + R 配合 [fzf](https://github.com/junegunn/fzf/wiki/Configuring-shell-key-bindings#ctrl-r) 使用，`fzf` 是一个通用对模糊查找工具，它可以和很多命令一起使用。

**另外一个和历史命令相关的技巧称之为基于历史的自动补全。 这一特性最初是由 [fish](https://fishshell.com/) shell 创建的，它可以根据您最近使用过的开头相同的命令，动态地对当前对shell命令进行补全。这一功能在 [zsh](https://github.com/zsh-users/zsh-autosuggestions) 中也可以使用，它可以极大的提高用户体验。**

### 文件夹导航

之前对所有操作我们都默认一个前提，即您已经位于想要执行命令的目录下，但是如何才能高效地在目录间随意切换呢？

有很多简便的方法可以做到，比如设置alias，使用 [ln -s](https://man7.org/linux/man-pages/man1/ln.1.html)创建符号连接等。而开发者们已经想到了很多更为精妙的解决方案。

由于本课程的目的是尽可能对你的日常习惯进行优化。因此，我们可以使用[`fasd`](https://github.com/clvv/fasd)和[autojump](https://github.com/wting/autojump)这两个工具来查找最常用或最近使用的文件和目录。

Fasd 基于 [*frecency*](https://developer.mozilla.org/en-US/docs/Mozilla/Tech/Places/Frecency_algorithm)对文件和文件排序，也就是说它会同时针对频率（*frequency* ）和时效（ *recency*）进行排序。默认情况下，`fasd`使用命令 `z` 帮助我们快速切换到最常访问的目录。例如， 如果您经常访问`/home/user/files/cool_project` 目录，那么可以直接使用 `z cool` 跳转到该目录。对于 autojump，则使用`j cool`代替即可。

还有一些更复杂的工具可以用来概览目录结构，例如 [`tree`](https://linux.die.net/man/1/tree), [`broot`](https://github.com/Canop/broot) 或更加完整的文件管理器，例如 [`nnn`](https://github.com/jarun/nnn) 或 [`ranger`](https://github.com/ranger/ranger)。

# 课后练习

1. 阅读 [`man ls`](https://man7.org/linux/man-pages/man1/ls.1.html) ，然后使用`ls` 命令进行如下操作：

   - 所有文件（包括隐藏文件）/
   - 文件打印以人类可以理解的格式输出 (例如，使用454M 而不是 454279954)
   - 文件以最近访问顺序排序
   - 以彩色文本显示输出结果

   典型输出如下：

   ```bash
    -rw-r--r--   1 user group 1.1M Jan 14 09:53 baz
    drwxr-xr-x   5 user group  160 Jan 14 09:53 .
    -rw-r--r--   1 user group  514 Jan 14 06:42 bar
    -rw-r--r--   1 user group 106M Jan 13 12:12 foo
    drwx------+ 47 user group 1.5K Jan 12 18:08 ..
   ```

   ```bash
   # man ls
   # /all
   ## -a --all
   #		do not ignore entries starting with .
   ## -A --almost-all
   #		do not list implied . and ..
   msun@ceph57 ~ $ ls -A
   .bash_history  .bash_profile  d_for_find   .emacs    grep      ls.txt
   .bash_logout   .bashrc        dotfiles.md  .emacs.d  .lesshst
   
   # man ls
   # /human
   # -h --human-readable
   # 		with -l, print sizes in human readable format (e.g., 1K 234M 2G)
   msun@ceph57 ~ $ ls -l
   total 4
   drwxrwxr-x 2 msun msun  6 Dec 17 15:45 d_for_find
   -rw-rw-r-- 1 msun msun  0 Dec 16 15:46 dotfiles.md
   -rw-rw-r-- 1 msun msun  0 Dec 17 17:08 grep
   -rw-rw-r-- 1 msun msun 48 Dec 16 16:02 ls.txt
   msun@ceph57 ~ $ ls -l -h
   total 4.0K
   # 但是这里不知道为什么就是单个文件米有显示KB MB等 待解决
   drwxrwxr-x 2 msun msun  6 Dec 17 15:45 d_for_find
   -rw-rw-r-- 1 msun msun  0 Dec 16 15:46 dotfiles.md
   -rw-rw-r-- 1 msun msun  0 Dec 17 17:08 grep
   -rw-rw-r-- 1 msun msun 48 Dec 16 16:02 ls.txt
   
   ## man ls
   # /time
   # -u     with -lt: sort by, and show, access time; with -l: show access time and sort by name; other‐
   #              wise: sort by access time
   msun@ceph57 ~ $ ls -lt
   total 4
   -rw-rw-r-- 1 msun msun  0 Dec 17 17:08 grep
   drwxrwxr-x 2 msun msun  6 Dec 17 15:45 d_for_find
   -rw-rw-r-- 1 msun msun 48 Dec 16 16:02 ls.txt
   -rw-rw-r-- 1 msun msun  0 Dec 16 15:46 dotfiles.md
   
   ## man ls
   # /color
   #       --color[=WHEN]
   #              colorize the output; WHEN can be 'never', 'auto', or 'always' (the default); more info below
   # 其实默认的就是输出彩色 所以就算不加flag也能得到彩色输出结果
   msun@ceph57 ~ $ ls --color=auto
   d_for_find  dotfiles.md  grep  ls.txt
   ```

2. 编写两个bash函数 `marco` 和 `polo` 执行下面的操作。 每当你执行 `marco` 时，当前的工作目录应当以某种形式保存，当执行 `polo` 时，无论现在处在什么目录下，都应当 `cd` 回到当时执行 `marco` 的目录。 为了方便debug，你可以把代码写在单独的文件 `marco.sh` 中，并通过 `source marco.sh`命令，（重新）加载函数。

   ```bash
   # 首先测试如何得到pwd的值 如下
   root@ceph57 ~/summiao # echo "$(pwd)"
   /root/summiao
   
   # 所以脚本可以是
     7 #!/bin/bash
     6 marco(){
     5         echo "$(pwd)" > /root/marco.log
     4         echo "save the $(pwd)"
     3 }
     2 
     1 # invoke it
   8   marco
   
   # 然后给一下权限
   root@ceph57 ~/summiao # chmod u+x marco.sh
   
   # 就可以执行了 执行结果没问题
   root@ceph57 ~/summiao # ./marco.sh
   save the /root/summiao
   
   # 还可以确认一下日志文件
   root@ceph57 ~/summiao # cat /root/marco.log
   /root/summiao
   
   # 同理进行polo的编写
     5 #!/bin/bash
     4 polo(){
     3         cd "$(cat "/root/marco.log")"
     2 }
     1 
   6   polo
   
   # 然后执行polo
   root@ceph57 ~ # source ~/summiao/polo.sh
   root@ceph57 ~/summiao # 
   ```

   或者如下所示

   ```bash
    #!/bin/bash
    marco() {
        export MARCO=$(pwd)
    }
    polo() {
        cd "$MARCO"
    }
   ```

3. 假设您有一个命令，它很少出错。因此为了在出错时能够对其进行调试，需要花费大量的时间重现错误并捕获输出。 编写一段bash脚本，运行如下的脚本直到它出错，将它的标准输出和标准错误流记录到文件，并在最后输出所有内容。 加分项：报告脚本在失败前共运行了多少次。

   ```bash
    #!/usr/bin/env bash
   
    n=$(( RANDOM % 100 ))
   
    if [[ n -eq 42 ]]; then
       echo "Something went wrong"
       >&2 echo "The error was using magic numbers"
       exit 1
    fi
   
    echo "Everything went according to plan"
   ```

   使用 while 循环完成

   ```bash
    count=1
   
    while true
    do
        ./buggy.sh 2> out.log
        if [[ $? -ne 0 ]]; then
            echo "failed after $count times"
            cat out.log
            break
        fi
        ((count++))
   
    done
   ```

   使用 for 循环完成

   ```bash
    for ((count=1;;count++))
    do
        ./buggy.sh 2> out.log
        if [[ $? -ne 0 ]]; then
            echo "failed after $count times"
            cat out.log
            break
   
        echo "$count try"
        fi
    done
   ```

   使用 until 完成

   ```bash
    #!/usr/bin/env bash
    count=0
    until [[ "$?" -ne 0 ]];
    do
        count=$((count+1))
        ./random.sh 2> out.txt
    done
   
    echo "found error after $count runs"
    cat out.txt
   ```

4. 本节课我们讲解的 `find` 命令中的 `-exec` 参数非常强大，它可以对我们查找的文件进行操作。但是，如果我们要对所有文件进行操作呢？例如创建一个zip压缩文件？我们已经知道，命令行可以从参数或标准输入接受输入。在用管道连接命令时，我们将标准输出和标准输入连接起来，**但是有些命令，例如`tar` 则需要从参数接受输入。这里我们可以使用[`xargs`](https://man7.org/linux/man-pages/man1/xargs.1.html) 命令，它可以使用标准输入中的内容作为参数。** 例如 `ls | xargs rm` 会删除当前目录中的所有文件。

   **xargs全称是transform arguments，意为转换参数，它将标准输入转换为命令行参数。**

   您的任务是编写一个命令，它可以**递归地**查找文件夹中所有的HTML文件，并将它们压缩成zip文件。**注意，即使文件名中包含空格，您的命令也应该能够正确执行**（提示：查看 `xargs`的参数`-d`，译注：MacOS 上的 `xargs`没有`-d`，[查看这个issue](https://github.com/missing-semester/missing-semester/issues/93)）

   在Linux操作系统中，find命令非常强大，在文件与目录的查找结合xargs命令使用，更是强大无比。

   ```bash
   # 我们首先看一下man find
   # /-print0
   # True; print the full file name on the standard output, followed by a null character (instead of the newline character that -print uses).  This allows file names that contain newlines or other types of white space to be correctly interpreted by programs  that  process  the  find  output. This option corresponds to the -0 option of xargs.
   # 真的;在标准输出上打印完整的文件名，后跟一个空字符
   #（而不是 -print 使用的换行符）意即默认的话 find找到的文件名结果之间是以换行符分开的
   # 这允许处理查找输出的程序正确解释包含换行符或其他类型空格的文件名。此选项对应于 xargs 的 -0 选项。
   
   # msun@ceph57 ~ $ man xargs
   # /-d
   # --delimiter=delim, -d delim
   # Input items are terminated by the specified character.  The specified  delimiter  may  be  a single  character,  a C-style character escape such as \n, or an octal or hexadecimal escape code.  Octal and hexadecimal escape codes are understood as for the printf command.   Multibyte  characters are not supported.  When processing the input, quotes and backslash are not special; every character in the input is taken literally.  The -d option disables  any  end-of-file  string, which is treated like any other argument.  You can use this option when the input consists of simply newline-separated items, although it is almost always better to design your program to use --null where this is possible.
   # 输入项以指定的字符结束。当输入由简单的换行符分隔项组成时，您可以使用此选项，尽管将程序设计为在可能的情况下使用 --null 几乎总是更好。
   ## 还有 EXAMPLE
   find /tmp -name core -type f -print0 | xargs -0 /bin/rm -f
   #       Find files named core in or below the directory /tmp and delete them, processing filenames in  such
   #       a way that file or directory names containing spaces or newlines are correctly handled.
   
   # 创建一个带空格的名称的html文件
   msun@ceph57 ~ $ touch spa\ ce.html
   
   msun@ceph57 ~ $ ls
   d_for_find  dotfiles.md  grep  ls  ls.txt  spa ce.html  xargs.html
   
   # 建立子目录 （为了测试是否递归 —— 实际上find默认是递归的）
   msun@ceph57 ~/d_for_find $ ls
   recursive.html
   # 此时文件树为：
   msun@ceph57 ~ $ sudo tree .
   .
   ├── d_for_find
   │   └── recursive.html
   ├── spa\ ce.html
   └── xargs.html
   
   # 注意 find 本身就是递归查找的
   # 但是直接查找找不到带空格的html文件 如下
   msun@ceph57 ~ $ find . -name '*.html' -type f
   ./d_for_find/recursive.html
   ./xargs.html
   
   # 正确的处理方法有很多 比如
   # 这种方法就是将find操作得到的结果中的换行符用空字符替换
   # 然后对应的xargs用空字符找文件之间的分界线
   find . -name "*.html" -print0 | xargs -0 zip html.zip
   
   # 又或者是这种方法
   # 对find的结果不作替换 仍旧是以换行符\n作为分界
   # 在xargs的时候 用-d 指定换行符为多个文件之间的分界线
   find . -name "*.html" | xargs -d "\n" zip html.zip
   
   # 检查一下
   msun@ceph57 ~ $ unzip html.zip
   Archive:  html.zip
   replace d_for_find/recursive.html? [y]es, [n]o, [A]ll, [N]one, [r]ename: y
    extracting: d_for_find/recursive.html  
   replace spa ce.html? [y]es, [n]o, [A]ll, [N]one, [r]ename: y
    extracting: spa ce.html             
   replace xargs.html? [y]es, [n]o, [A]ll, [N]one, [r]ename: y
    extracting: xargs.html     
   ```

   如果您使用的是 MacOS，请注意默认的 BSD `find` 与[GNU coreutils](https://en.wikipedia.org/wiki/List_of_GNU_Core_Utilities_commands) 中的是不一样的。你可以为`find`添加`-print0`选项，并为`xargs`添加`-0`选项。作为 Mac 用户，您需要注意 mac 系统自带的命令行工具和 GNU 中对应的工具是有区别的；如果你想使用 GNU 版本的工具，也可以使用 [brew 来安装](https://formulae.brew.sh/formula/coreutils)。

5. (进阶) 编写一个命令或脚本递归的查找文件夹中最近使用的文件。更通用的做法，你可以按照最近的使用时间列出文件吗？

   ```bash
   # . 可以替换成任何文件夹
   # -t 表示以时间为序
   # -4 可以替换成任何数字 表示从上往下的显示的行数
   root@ceph57 ~/summiao # find . -type f print0 | xargs -0 ls -lt | head -4
   find: paths must precede expression: print0
   Usage: find [-H] [-L] [-P] [-Olevel] [-D help|tree|search|stat|rates|opt|exec] [path...] [expression]
   total 24
   -rwxr--r--  1 root root   59 Dec 28 14:44 polo.sh
   -rwxr--r--  1 root root  101 Dec 28 14:34 marco.sh
   -rw-r--r--  1 root root    6 Dec 28 09:38 dhello.txt
   
   # p.s.
   # 当文件数量较多时，上面的解答会得出错误结果，解决办法是增加 -mmin 条件，先将最近修改的文件进行初步筛选再交给ls进行排序显示 
   find . -type f -mmin -60 -print0 | xargs -0 ls -lt | head -10
   ```
