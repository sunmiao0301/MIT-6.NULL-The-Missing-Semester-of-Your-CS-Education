### 版本控制(Git)

版本控制系统 (VCSs) 是一类用于追踪源代码（或其他文件、文件夹）改动的工具。顾名思义，这些工具可以帮助我们管理代码的修改历史；不仅如此，它还可以让协作编码变得更方便。VCS通过一系列的快照将某个文件夹及其内容保存了起来，每个快照都包含了文件或文件夹的完整状态。同时它还维护了快照创建者的信息以及每个快照的相关信息等等。

为什么说版本控制系统非常有用？即使您只是一个人进行编程工作，它也可以帮您创建项目的快照，记录每个改动的目的、基于多分支并行开发等等。和别人协作开发时，它更是一个无价之宝，您可以看到别人对代码进行的修改，同时解决由于并行开发引起的冲突。

现代的版本控制系统可以帮助您轻松地（甚至自动地）回答以下问题：

- 当前模块是谁编写的？
- 这个文件的这一行是什么时候被编辑的？是谁作出的修改？修改原因是什么呢？
- 最近的1000个版本中，何时/为什么导致了单元测试失败？

尽管版本控制系统有很多， 其事实上的标准则是 **Git** 。而这篇 [XKCD 漫画](https://xkcd.com/1597/) 则反映出了人们对 Git 的评价：

![1227git.png](https://raw.githubusercontent.com/sunmiao0301/Public-Pic-Bed/main/1227git.png)

因为 Git 接口的抽象泄漏（leaky abstraction）问题，通过自顶向下的方式（从命令行接口开始）学习 Git 可能会让人感到非常困惑。很多时候您只能死记硬背一些命令行，然后像使用魔法一样使用它们，一旦出现问题，就只能像上面那幅漫画里说的那样去处理了。—— *一旦犯错，把你的工作保存在其他地方，然后删除项目，最后下载一个新的Copy*

尽管 Git 的接口有些丑陋，但是它的底层设计和思想却是非常优雅的。丑陋的接口只能靠死记硬背，而优雅的底层设计则非常容易被人理解。因此，我们将通过一种自底向上的方式向您介绍 Git。我们会从数据模型开始，最后再学习它的接口。一旦您搞懂了 Git 的数据模型，再学习其接口并理解这些接口是如何操作数据模型的就非常容易了。

### Git的数据模型

进行版本控制的方法很多。Git 拥有一个经过精心设计的模型，这使其能够支持版本控制所需的所有特性，例如维护历史记录、支持分支和促进协作。

#### 快照

Git 将顶级目录中的文件和文件夹作为集合，并通过一系列快照来管理其历史记录。在Git的术语里，文件被称作Blob对象（数据对象），也就是一组数据。目录则被称之为“树”，它将名字与 Blob 对象或树对象进行映射（使得目录中可以包含其他目录）。快照则是被追踪的最顶层的树。例如，一个树看起来可能是这样的：

```
<root> (tree)
|
+- foo (tree)
|  |
|  + bar.txt (blob, contents = "hello world")
|
+- baz.txt (blob, contents = "git is wonderful")
```

这个顶层的树包含了两个元素，一个名为 “foo” 的树（它本身包含了一个blob对象 “bar.txt”），以及一个 blob 对象 “baz.txt”。

#### 关联快照

版本控制系统和快照有什么关系呢？线性历史记录是一种最简单的模型，它包含了一组按照时间顺序线性排列的快照。不过处于种种原因，Git 并没有采用这样的模型。

在 Git 中，历史记录是一个由快照组成的有向无环图。有向无环图，听上去似乎是什么高大上的数学名词。不过不要怕，您只需要知道这代表 Git 中的每个快照都有一系列的“父辈”，也就是其之前的一系列快照。注意，快照具有多个“父辈”而非一个，因为某个快照可能由多个父辈而来。例如，经过合并后的两条分支。

在 Git 中，这些快照被称为“提交”。通过可视化的方式来表示这些历史提交记录时，看起来差不多是这样的：

```
o <-- o <-- o <-- o
            ^  
             \
              --- o <-- o
```

上面是一个 ASCII 码构成的简图，其中的 `o` 表示一次提交（快照）。

箭头指向了当前提交的父辈（这是一种“在。。。之前”，而不是“在。。。之后”的关系）。在第三次提交之后，历史记录分岔成了两条独立的分支。这可能因为此时需要同时开发两个不同的特性，它们之间是相互独立的。开发完成后，这些分支可能会被合并并创建一个新的提交，这个新的提交会同时包含这些特性。新的提交会创建一个新的历史记录，看上去像这样（最新的合并提交用粗体标记，但是markdown的代码块好像不支持）：

```
o <-- o <-- o <-- o <---- o
            ^            /
             \          v
              --- o <-- o
```

Git 中的提交是不可改变的。但这并不代表错误不能被修改，只不过这种“修改”实际上是创建了一个全新的提交记录。而引用（参见下文）则被更新为指向这些新的提交。

#### 数据模型及其伪代码表示

以伪代码的形式来学习 Git 的数据模型，可能更加清晰：

```c
// 文件就是一组数据
type blob = array<byte>

// 一个包含文件和目录的目录
type tree = map<string, tree | blob>

// 每个提交都包含一个父辈，元数据和顶层树
type commit = struct {
    parent: array<commit>
    author: string
    message: string
    snapshot: tree
}
```

这是一种简洁的历史模型。

#### 对象和内存寻址

Git 中的对象可以是 blob、树或提交：

```
type object = blob | tree | commit
```

Git 在储存数据时，所有的对象都会基于它们的 [SHA-1 哈希](https://en.wikipedia.org/wiki/SHA-1) 进行寻址。

```c
objects = map<string, object>

def store(object):
    id = sha1(object)
    objects[id] = object

def load(id):
    return objects[id]
```

Blobs、树和提交都一样，它们都是对象。当它们引用其他对象时，它们并没有真正的在硬盘上保存这些对象，而是仅仅保存了它们的哈希值作为引用。

例如，[上面](https://missing-semester-cn.github.io/2020/version-control/#snapshots)例子中的树（可以通过 `git cat-file -p 698281bc680d1995c5f4caaf3359721a5a58d48d` 来进行可视化），看上去是这样的：

```
100644 blob 4448adbf7ecd394f42ae135bbeed9676e894af85    baz.txt
040000 tree c68d233a33c5c06e0340e4c224f0afca87c8ce87    foo
```

树本身会包含一些指向其他内容的指针，例如 `baz.txt` (blob) 和 `foo` (树)。如果我们用 `git cat-file -p 4448adbf7ecd394f42ae135bbeed9676e894af85`，即通过哈希值查看 baz.txt 的内容，会得到以下信息：

```
git is wonderful
```

#### 引用

现在，所有的快照都可以通过它们的 SHA-1 哈希值来标记了。但这也太不方便了，谁也记不住一串 40 位的十六进制字符。

针对这一问题，Git 的解决方法是给这些哈希值赋予人类可读的名字，也就是引用（references）。引用是指向提交的指针。与对象不同的是，它是可变的（引用可以被更新，指向新的提交）。例如，`master` 引用通常会指向主分支的最新一次提交。

```
references = map<string, string>

def update_reference(name, id):
    references[name] = id

def read_reference(name):
    return references[name]

def load_reference(name_or_id):
    if name_or_id in references:
        return load(references[name_or_id])
    else:
        return load(name_or_id)
```

这样，Git 就可以使用诸如 “master” 这样人类可读的名称来表示历史记录中某个特定的提交，而不需要在使用一长串十六进制字符了。

有一个细节需要我们注意， 通常情况下，我们会想要知道“我们当前所在位置”，并将其标记下来。这样当我们创建新的快照的时候，我们就可以知道它的相对位置（如何设置它的“父辈”）。在 Git 中，我们当前的位置有一个特殊的索引，它就是 “HEAD”。

#### 仓库

最后，我们可以粗略地给出 Git 仓库的定义了：`对象` 和 `引用`。

在硬盘上，Git 仅存储对象和引用：因为其数据模型仅包含这些东西。所有的 `git` 命令都对应着对提交树的操作，例如增加对象，增加或删除引用。

当您输入某个指令时，请思考一下这条命令是如何对底层的图数据结构进行操作的。另一方面，如果您希望修改提交树，例如“丢弃未提交的修改和将 ‘master’ 引用指向提交 `5d83f9e` 时，有什么命令可以完成该操作（针对这个具体问题，您可以使用 `git checkout master; git reset --hard 5d83f9e`）

### 暂存区

Git 中还包括一个和数据模型完全不相关的概念，但它确是创建提交的接口的一部分。

就上面介绍的快照系统来说，您也许会期望它的实现里包括一个 “创建快照” 的命令，该命令能够基于当前工作目录的当前状态创建一个全新的快照。有些版本控制系统确实是这样工作的，但 Git 不是。**我们希望简洁的快照，而且每次从当前状态创建快照可能效果并不理想。**例如，考虑如下场景，您开发了两个独立的特性，然后您希望创建两个独立的提交，其中第一个提交仅包含第一个特性，而第二个提交仅包含第二个特性。或者，假设您在调试代码时添加了很多打印语句，然后您仅仅希望提交和修复 bug 相关的代码而丢弃所有的打印语句。

Git 处理这些场景的方法是使用一种叫做 “暂存区（staging area）”的机制，**它允许您指定下次快照中要包括那些改动。**

### Git的命令行接口

为了避免重复信息，我们将不会详细解释以下命令行。强烈推荐您阅读 [Pro Git 中文版](https://git-scm.com/book/zh/v2)或可以观看本讲座的视频来学习。

#### 基础

- `git help <command>`: 获取 git 命令的帮助信息
- `git init`: 创建一个新的 git 仓库，其数据会存放在一个名为 `.git` 的目录下
- `git status`: 显示当前的仓库状态
- `git add <filename>`: 添加文件到暂存区
- ```git commit```: 创建一个新的提交
  - 如何编写 [良好的提交信息](https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)!
  - 为何要 [编写良好的提交信息](https://chris.beams.io/posts/git-commit/)
- `git log`: 显示历史日志
- `git log --all --graph --decorate`: 可视化历史记录（有向无环图）
- `git diff <filename>`: 显示与暂存区文件的差异
- `git diff <revision> <filename>`: 显示某个文件两个版本之间的差异
- `git checkout <revision>`: 更新 HEAD 和目前的分支

#### 分支和合并

- `git branch`: 显示分支

- `git branch <name>`: 创建分支

- ```plaintext
  git checkout -b <name>
  ```

  : 创建分支并切换到该分支

  - 相当于 `git branch <name>; git checkout <name>`

- `git merge <revision>`: 合并到当前分支

- `git mergetool`: 使用工具来处理合并冲突

- `git rebase`: 将一系列补丁变基（rebase）为新的基线

#### 远端操作

- `git remote`: 列出远端
- `git remote add <name> <url>`: 添加一个远端
- `git push <remote> <local branch>:<remote branch>`: 将对象传送至远端并更新远端引用
- `git branch --set-upstream-to=<remote>/<remote branch>`: 创建本地和远端分支的关联关系
- `git fetch`: 从远端获取对象/索引
- `git pull`: 相当于 `git fetch; git merge`
- `git clone`: 从远端下载仓库

#### 撤销

- `git commit --amend`: 编辑提交的内容或信息
- `git reset HEAD <file>`: 恢复暂存的文件
- `git checkout -- <file>`: 丢弃修改

### Git 高级操作

- `git config`: Git 是一个 [高度可定制的](https://git-scm.com/docs/git-config) 工具
- `git clone --depth=1`: 浅克隆（shallow clone），不包括完整的版本历史信息
- `git add -p`: 交互式暂存
- `git rebase -i`: 交互式变基
- `git blame`: 查看最后修改某行的人
- `git stash`: 暂时移除工作目录下的修改内容
- `git bisect`: 通过二分查找搜索历史记录
- `.gitignore`: [指定](https://git-scm.com/docs/gitignore) 故意不追踪的文件

### 杂项

- **图形用户界面**: Git 的 [图形用户界面客户端](https://git-scm.com/downloads/guis) 有很多，但是我们自己并不使用这些图形用户界面的客户端，我们选择使用命令行接口
- **Shell 集成**: 将 Git 状态集成到您的 shell 中会非常方便。([zsh](https://github.com/olivierverdier/zsh-git-prompt), [bash](https://github.com/magicmonty/bash-git-prompt))。[Oh My Zsh](https://github.com/ohmyzsh/ohmyzsh)这样的框架中一般以及集成了这一功能
- **编辑器集成**: 和上面一条类似，将 Git 集成到编辑器中好处多多。[fugitive.vim](https://github.com/tpope/vim-fugitive) 是 Vim 中集成 GIt 的常用插件
- **工作流**: 我们已经讲解了数据模型与一些基础命令，但还没讨论到进行大型项目时的一些惯例 ( 有[很多](https://nvie.com/posts/a-successful-git-branching-model/) [不同的](https://www.endoflineblog.com/gitflow-considered-harmful) [处理方法](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow))
- **GitHub**: Git 并不等同于 GitHub。 在 GitHub 中您需要使用一个被称作[拉取请求（pull request）](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-requests)的方法来向其他项目贡献代码
- **其他 Git 提供商**: GitHub 并不是唯一的。还有像 [GitLab](https://about.gitlab.com/) 和 [BitBucket](https://bitbucket.org/) 这样的平台。

### 资源

- [Pro Git](https://git-scm.com/book/en/v2) ，**强烈推荐**！学习前五章的内容可以教会您流畅使用 Git 的绝大多数技巧，因为您已经理解了 Git 的数据模型。后面的章节提供了很多有趣的高级主题。（[Pro Git 中文版](https://git-scm.com/book/zh/v2)）；
- [Oh Shit, Git!?!](https://ohshitgit.com/) ，简短的介绍了如何从 Git 错误中恢复；
- [Git for Computer Scientists](https://eagain.net/articles/git-for-computer-scientists/) ，简短的介绍了 Git 的数据模型，与本文相比包含较少量的伪代码以及大量的精美图片；
- [Git from the Bottom Up](https://jwiegley.github.io/git-from-the-bottom-up/)详细的介绍了 Git 的实现细节，而不仅仅局限于数据模型。好奇的同学可以看看；
- [How to explain git in simple words](https://smusamashah.github.io/blog/2017/10/14/explain-git-in-simple-words)；
- [Learn Git Branching](https://learngitbranching.js.org/) 通过基于浏览器的游戏来学习 Git ；

# 课后练习

[习题解答](https://missing-semester-cn.github.io/missing-notes-and-solutions/2020/solutions//version-control-solution)

1. 如果您之前从来没有用过 Git，推荐您阅读 [Pro Git](https://git-scm.com/book/en/v2) 的前几章，或者完成像 [Learn Git Branching](https://learngitbranching.js.org/)这样的教程。重点关注 Git 命令和数据模型相关内容；

2. Fork本课程网站的仓库

   1. 将版本历史可视化并进行探索
   2. 是谁最后修改了 `README.md`文件？（提示：使用 `git log` 命令并添加合适的参数）
   3. 最后一次修改`_config.yml` 文件中 `collections:` 行时的提交信息是什么？（提示：使用 `git blame` 和 `git show`）

3. 使用 Git 时的一个常见错误是提交本不应该由 Git 管理的大文件，或是将含有敏感信息的文件提交给 Git 。尝试向仓库中添加一个文件并添加提交信息，然后将其从历史中删除 ( [这篇文章也许会有帮助](https://help.github.com/articles/removing-sensitive-data-from-a-repository/))；

   ```bash
   # ssh到本地
   hfut_@DESKTOP-HA2C4BG MINGW64 /d/repo (master)
   $ git clone git@github.com:sunmiao0301/Public-Pic-Bed.git
   Cloning into 'Public-Pic-Bed'...
   remote: Enumerating objects: 28, done.
   remote: Counting objects: 100% (19/19), done.
   remote: Compressing objects: 100% (18/18), done.
   remote: Total 28 (delta 5), reused 0 (delta 0), pack-reused 9
   Receiving objects: 100% (28/28), 438.95 KiB | 487.00 KiB/s, done.
   Resolving deltas: 100% (5/5), done.
   
   # 进入repository
   hfut_@DESKTOP-HA2C4BG MINGW64 /d/repo (master)
   $ cd Public-Pic-Bed
   
   # 看文件
   hfut_@DESKTOP-HA2C4BG MINGW64 /d/repo/Public-Pic-Bed (main)
   $ ls
   1216Ceph_stack.png  1224SwitchOmega.png  1227git.png
   1224Dynamic.png     1224local.png        211216zhp1.png
   1224LaTex.png       1224tanshe1.png      README.md
   
   # 新建一个txt
   hfut_@DESKTOP-HA2C4BG MINGW64 /d/repo/Public-Pic-Bed (main)
   $ echo "pw is 123" > pw.txt
   
   # done
   hfut_@DESKTOP-HA2C4BG MINGW64 /d/repo/Public-Pic-Bed (main)
   $ ls
   1216Ceph_stack.png  1224SwitchOmega.png  1227git.png     pw.txt
   1224Dynamic.png     1224local.png        211216zhp1.png
   1224LaTex.png       1224tanshe1.png      README.md
   
   # add
   hfut_@DESKTOP-HA2C4BG MINGW64 /d/repo/Public-Pic-Bed (main)
   $ git add .
   warning: LF will be replaced by CRLF in pw.txt.
   The file will have its original line endings in your working directory
   
   # done
   hfut_@DESKTOP-HA2C4BG MINGW64 /d/repo/Public-Pic-Bed (main)
   $ git status
   On branch main
   Your branch is up to date with 'origin/main'.
   
   Changes to be committed:
     (use "git restore --staged <file>..." to unstage)
           new file:   pw.txt
   
   # commit
   hfut_@DESKTOP-HA2C4BG MINGW64 /d/repo/Public-Pic-Bed (main)
   $  git commit -m "my password"
   [main 156a097] my password
    1 file changed, 1 insertion(+)
    create mode 100644 pw.txt
   
   # done
   hfut_@DESKTOP-HA2C4BG MINGW64 /d/repo/Public-Pic-Bed (main)
   $ git status
   On branch main
   Your branch is ahead of 'origin/main' by 1 commit.
     (use "git push" to publish your local commits)
   
   nothing to commit, working tree clean
   
   # 看日志
   hfut_@DESKTOP-HA2C4BG MINGW64 /d/repo/Public-Pic-Bed (main)
   $ git log HEAD
   commit 156a09768a474b4ad74ae6a6488716a48df03fe3 (HEAD -> main)
   Author: sunmiao0301 <hfut_sm@163.com>
   Date:   Mon Dec 27 11:10:43 2021 +0800
   
       my password
   
   # 回退
   hfut_@DESKTOP-HA2C4BG MINGW64 /d/repo/Public-Pic-Bed (main)
   $ git reset --soft HEAD^
   
   # done
   hfut_@DESKTOP-HA2C4BG MINGW64 /d/repo/Public-Pic-Bed (main)
   $ git status
   On branch main
   Your branch is up to date with 'origin/main'.
   
   Changes to be committed:
     (use "git restore --staged <file>..." to unstage)
           new file:   pw.txt
           
   # 这样就成功的撤销了你的commit，注意，仅仅是撤回commit操作，您写的代码仍然保留。
   
   # HEAD^的意思是上一个版本，也可以写成HEAD~1
   # 如果你进行了2次commit，想都撤回，可以使用HEAD~2
   # 至于这几个参数：
   # --mixed 
   # 意思是：不删除工作空间改动代码，撤销commit，并且撤销git add . 操作这个为默认参数,git reset --mixed HEAD^ 和 git reset HEAD^ 效果是一样的。
   # --soft 
   # 不删除工作空间改动代码，撤销commit，不撤销git add . 
   # --hard
   # 删除工作空间改动代码，撤销commit，撤销git add .
   # 完成这个操作后，就恢复到了上一次的commit状态。
   
   # 顺便说一下，如果commit注释写错了，只是想改一下注释，只需要：git commit --amend，此时会进入默认vim编辑器，修改注释完毕后保存就ok了。
   ```

   [原文链接](https://blog.csdn.net/w958796636/article/details/53611133)

4. 从 GitHub 上克隆某个仓库，修改一些文件。当您使用 `git stash` 会发生什么？当您执行 `git log --all --oneline` 时会显示什么？通过 `git stash pop` 命令来撤销 `git stash` 操作，什么时候会用到这一技巧？

5. 与其他的命令行工具一样，Git 也提供了一个名为 `~/.gitconfig` 配置文件 (或 dotfile)。请在 `~/.gitconfig` 中创建一个别名，使您在运行 `git graph` 时，您可以得到 `git log --all --graph --decorate --oneline` 的输出结果；

6. 您可以通过执行 `git config --global core.excludesfile ~/.gitignore_global` 在 `~/.gitignore_global` 中创建全局忽略规则。配置您的全局 gitignore 文件来自动忽略系统或编辑器的临时文件，例如 `.DS_Store`；

7. 克隆 [本课程网站的仓库](https://github.com/missing-semester/missing-semester)，找找有没有错别字或其他可以改进的地方，在 GitHub 上发起拉取请求（Pull Request）；

   ```bash
   首先 fork 本网站仓库，然后克隆 fork 后的仓库
   git clone https://github.com/hanxiaomax/missing-semester.git
   在本地进行修改后，提交到 fork 后的仓库，然后发起 PR 即可
   ```

   ## Supplement from video

   ```bash
   ~
   $ cd /d
   
   /d (master)
   $ mkdir demo
   
   /d (master)
   $ cd demo
   
   /d/demo (master)
   $ ls
   
   /d/demo (master)
   $ git init
   Initialized empty Git repository in D:/demo/.git/
   
   /d/demo (master)
   $ ls
   
   # git init 使得生成了隐藏文件
   /d/demo (master)
   $ ls -a
   ./  ../  .git/
   
   /d/demo (master)
   $ ls .git
   HEAD  config  description  hooks/  info/  objects/  refs/
   
   # 如下命令会打开一个网页 关于git init的讲解
   /d/demo (master)
   $ git help init
   
   /d/demo (master)
   $ echo "hello world" > hello.txt
   
   /d/demo (master)
   $ ls
   hello.txt
   
   /d/demo (master)
   $ git status
   On branch master
   
   No commits yet
   # it mains that there is a new created file in currently file but not included in the next snapshot
   Untracked files:
     (use "git add <file>..." to include in what will be committed)
           hello.txt
   
   nothing added to commit but untracked files present (use "git add" to track)
   
   /d/demo (master)
   $ git add hello.txt
   warning: LF will be replaced by CRLF in hello.txt.
   The file will have its original line endings in your working directory
   
   /d/demo (master)
   $ git status
   On branch master
   
   No commits yet
   
   Changes to be committed:
     (use "git rm --cached <file>..." to unstage)
           new file:   hello.txt
   
   # run git commit
   
   Add hello.txt
   # Please enter the commit message for your changes. Lines starting
   # with '#' will be ignored, and an empty message aborts the commit.
   #
   # On branch master
   #
   # Initial commit
   #
   # Changes to be committed:
   #       new file:   hello.txt
   #
   ~
   ~
   ~
   ~
   ~
   ~
   ~
   ~
   ~
   ~
   ~
   D:/demo/.git/COMMIT_EDITMSG [unix] (14:27 27/12/2021)                  1,0-1 All
   "D:/demo/.git/COMMIT_EDITMSG" [unix] 11L, 231B
   
   # Esc -> :x
   
   /d/demo (master)
   $ git commit
   [master (root-commit) c87b88b] Add hello.txt
    1 file changed, 1 insertion(+)
    create mode 100644 hello.txt
   
   /d/demo (master)
   $ git log
   commit c87b88b4bef7456bc621f711b25464c0fb19fb6b (HEAD -> master)
   Author: sunmiao0301 <hfut_sm@163.com>
   Date:   Mon Dec 27 14:27:13 2021 +0800
   
       Add hello.txt
   
   /d/demo (master)
   $ echo "another line" >> hello.txt
   
   /d/demo (master)
   $ cat hello.txt
   hello world
   another line
   
   # 因为只是做了修改 而没有将其add到目前的暂存区中 所以无法commit
   /d/demo (master)
   $ git commit
   On branch master
   Changes not staged for commit:
     (use "git add <file>..." to update what will be committed)
     (use "git restore <file>..." to discard changes in working directory)
           modified:   hello.txt
   
   no changes added to commit (use "git add" and/or "git commit -a")
   
   /d/demo (master)
   $ git add hello.txt
   warning: LF will be replaced by CRLF in hello.txt.
   The file will have its original line endings in your working directory
   
   /d/demo (master)
   $ git status
   On branch master
   # this file is ready to be commit
   Changes to be committed:
     (use "git restore --staged <file>..." to unstage)
           modified:   hello.txt
   
   /d/demo (master)
   $ git commit
   [master 7d7b3d9] X
    1 file changed, 1 insertion(+)
   
   # the new change has made
   
   # github is a host for github repositories
   
   /d/demo (master)
   $ git log --all --graph  --decorate
   #### 可以看到 目前的head -> master
   * commit 7d7b3d91e0121a5e141b3768db7653767bb46b1d (HEAD -> master)
   | Author: sunmiao0301 <hfut_sm@163.com>
   | Date:   Mon Dec 27 14:38:41 2021 +0800
   |
   |     X
   |
   * commit c87b88b4bef7456bc621f711b25464c0fb19fb6b
     Author: sunmiao0301 <hfut_sm@163.com>
     Date:   Mon Dec 27 14:27:13 2021 +0800
   
         Add hello.txt
   
   /d/demo (master)
   $ git checkout c87b88b4
   Note: switching to 'c87b88b4'.
   
   You are in 'detached HEAD' state. You can look around, make experimental
   changes and commit them, and you can discard any commits you make in this
   state without impacting any branches by switching back to a branch.
   
   If you want to create a new branch to retain commits you create, you may
   do so (now or later) by using -c with the switch command. Example:
   
     git switch -c <new-branch-name>
   
   Or undo this operation with:
   
     git switch -
   
   Turn off this advice by setting config variable advice.detachedHead to false
   
   HEAD is now at c87b88b Add hello.txt
   
   /d/demo ((c87b88b...))
   $ git log --all --graph  --decorate
   * commit 7d7b3d91e0121a5e141b3768db7653767bb46b1d (master)
   | Author: sunmiao0301 <hfut_sm@163.com>
   | Date:   Mon Dec 27 14:38:41 2021 +0800
   |
   |     X
   |
   * commit c87b88b4bef7456bc621f711b25464c0fb19fb6b (HEAD)
   #### 可以看到head来到了这儿
     Author: sunmiao0301 <hfut_sm@163.com>
     Date:   Mon Dec 27 14:27:13 2021 +0800
   
         Add hello.txt
   
   /d/demo ((c87b88b...))
   
   /d/demo ((c87b88b...))
   $ git checkout master
   Previous HEAD position was c87b88b Add hello.txt
   Switched to branch 'master'
   
   /d/demo (master)
   $ cat hello.txt
   hello world
   another line
   
   # do some new change
   /d/demo (master)
   $ vim hello.txt
   
   # notice! git check will destory some of your modification if you are not care about it
   
   /d/demo (master)
   $ git diff c87b88b4 hello.txt
   diff --git a/hello.txt b/hello.txt
   index 3b18e51..8b5ce7a 100644
   --- a/hello.txt
   +++ b/hello.txt
   @@ -1 +1,3 @@
    hello world
   +another line
   +yet another line
   
   /d/demo (master)
   $ git branch
   * master
   
   /d/demo (master)
   $ git branch -vv
   * master 7d7b3d9 X
   
   /d/demo (master)
   $ git branch cat
   
   /d/demo (master)
   $ git log --all --graph  --decorate
   * commit 7d7b3d91e0121a5e141b3768db7653767bb46b1d (HEAD -> master, cat)
   | Author: sunmiao0301 <hfut_sm@163.com>
   | Date:   Mon Dec 27 14:38:41 2021 +0800
   |
   |     X
   |
   * commit c87b88b4bef7456bc621f711b25464c0fb19fb6b
     Author: sunmiao0301 <hfut_sm@163.com>
     Date:   Mon Dec 27 14:27:13 2021 +0800
   
         Add hello.txt
   
   /d/demo (master)
   
   /d/demo (master)
   $ git checkout cat
   Switched to branch 'cat'
   M       hello.txt
   
   /d/demo (cat)
   $ git log --all --graph  --decorate
   * commit 7d7b3d91e0121a5e141b3768db7653767bb46b1d (HEAD -> cat, master)
   | Author: sunmiao0301 <hfut_sm@163.com>
   | Date:   Mon Dec 27 14:38:41 2021 +0800
   |
   |     X
   |
   * commit c87b88b4bef7456bc621f711b25464c0fb19fb6b
     Author: sunmiao0301 <hfut_sm@163.com>
     Date:   Mon Dec 27 14:27:13 2021 +0800
   
         Add hello.txt
   
   /d/demo (cat)
   $ git diff
   diff --git a/hello.txt b/hello.txt
   index fdff486..8b5ce7a 100644
   --- a/hello.txt
   +++ b/hello.txt
   @@ -1,2 +1,3 @@
    hello world
    another line
   +yet another line
   
   /d/demo (cat)
   $ git log --all --graph  --decorate --oneline
   * 7d7b3d9 (HEAD -> cat, master) X
   * c87b88b Add hello.txt
   
   ... 53:29
   ```

   



