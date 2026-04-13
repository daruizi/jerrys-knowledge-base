# Git 从入门到精通

> 本教程面向零基础小白，边学边练，学完即用。参考廖雪峰 Git 教程（2025 版）整理。

---

## 目录

1. [Git 是什么，为什么要用它？](#1-git-是什么为什么要用它)
2. [安装 Git](#2-安装-git)
3. [第一次使用：初始配置](#3-第一次使用初始配置)
4. [创建版本库](#4-创建版本库)
5. [时光机穿梭：版本控制的核心操作](#5-时光机穿梭版本控制的核心操作)
   - [查看状态与修改](#51-查看状态与修改)
   - [版本回退](#52-版本回退)
   - [工作区与暂存区（重要概念）](#53-工作区与暂存区重要概念)
   - [管理修改：Git 到底追踪什么](#54-管理修改git-到底追踪什么)
   - [撤销修改：后悔药怎么吃](#55-撤销修改后悔药怎么吃)
   - [删除文件](#56-删除文件)
6. [远程仓库：代码上云](#6-远程仓库代码上云)
   - [SSH 密钥配置](#61-ssh-密钥配置)
   - [关联并推送远程库](#62-关联并推送远程库)
   - [从远程库克隆](#63-从远程库克隆)
7. [分支管理：Git 最强大的功能](#7-分支管理git-最强大的功能)
   - [创建与合并分支](#71-创建与合并分支)
   - [解决冲突](#72-解决冲突)
   - [分支管理策略](#73-分支管理策略)
   - [Bug 分支：用 stash 保存现场](#74-bug-分支用-stash-保存现场)
   - [Feature 分支](#75-feature-分支)
   - [多人协作](#76-多人协作)
   - [Rebase：让历史变成一条直线](#77-rebase让历史变成一条直线)
8. [标签管理](#8-标签管理)
9. [使用 GitHub 参与开源](#9-使用-github-参与开源)
10. [国内替代：使用 Gitee](#10-国内替代使用-gitee)
11. [自定义 Git](#11-自定义-git)
    - [忽略特殊文件 .gitignore](#111-忽略特殊文件-gitignore)
    - [配置别名](#112-配置别名)
    - [搭建私有 Git 服务器](#113-搭建私有-git-服务器)
12. [图形界面工具 SourceTree](#12-图形界面工具-sourcetree)
13. [常用命令速查表](#13-常用命令速查表)

---

## 1. Git 是什么，为什么要用它？

### 没有版本控制会怎样？

想象你在写一篇重要的报告。改着改着，你想删掉一段内容，但又怕删了以后找不回来，于是你把文件"另存为"一个新名字：

```
报告_v1.docx
报告_v2.docx
报告_v2_修改版.docx
报告_v2_修改版_最终版.docx
报告_v2_修改版_最终版_真的最终版.docx
```

一周后，你完全不记得哪个是最新的，也不知道每个版本之间改了什么。这就是**没有版本控制的痛苦**。

如果是多人协作，问题更严重——你改了文件，同事也改了文件，最后怎么合并？谁改了哪里？这些问题都没有答案。

**版本控制系统**就是为了解决这些问题而生的。它能自动记录每次修改：

| 版本 | 文件名 | 修改人 | 说明 | 日期 |
|------|--------|--------|------|------|
| 1 | readme.txt | 张三 | 新增介绍段落 | 3/7 10:38 |
| 2 | readme.txt | 李四 | 修正错别字 | 3/7 14:20 |
| 3 | readme.txt | 张三 | 补充参考资料 | 3/8 09:51 |

### 集中式 vs 分布式

版本控制系统分两大流派：

**集中式（CVS、SVN）**：版本库只放在中央服务器上。你工作时，必须先联网从服务器取最新版本，改完后再推回去。就像图书馆借书——没有书不行，还书也要去图书馆。**最大的缺点：必须联网才能工作**，网速慢就等慢，服务器宕机大家都傻眼。

**分布式（Git）**：每个人的电脑上都有一份**完整的版本库**。没有网络照样可以提交、查看历史、创建分支，等有网络了再同步。**即使服务器挂了，任何一台电脑都可以恢复整个项目历史**。

```
集中式（SVN）：                 分布式（Git）：

    [中央服务器]               你的电脑（完整版本库）
    ↑ push  ↓ pull                    ↑↓
[你的电脑]  [同事的电脑]       [远程服务器（GitHub）]
 （无完整历史）                        ↑↓
                               同事的电脑（完整版本库）
```

### Git 的诞生

Git 由 **Linux 之父 Linus Torvalds** 在 2005 年用 **C 语言** 仅花 **两周时间** 写出来的。

起因：Linux 社区一直免费使用一个商业版本控制系统 BitKeeper，但因为社区成员试图破解其协议，BitKeeper 公司取消了免费授权。Linus 一怒之下自己写了一个，这就是 Git。一个月后，Linux 内核源码全部切换到 Git 管理。

2008 年，GitHub 上线，为开源项目免费提供 Git 托管，Git 迅速成为全球最流行的版本控制系统。

---

## 2. 安装 Git

### macOS

```bash
# 推荐方式：通过 Homebrew 安装
brew install git

# 验证安装
git --version
# git version 2.x.x
```

也可以安装 Xcode Command Line Tools（会自带 Git）：

```bash
xcode-select --install
```

### Linux（Ubuntu / Debian）

```bash
sudo apt install git
```

其他发行版：

```bash
# CentOS / RHEL
sudo yum install git

# Arch Linux
sudo pacman -S git
```

### Windows

**方式一（推荐）**：到 [git-scm.com](https://git-scm.com) 下载安装包，按默认选项安装。安装完后，在开始菜单找到 **Git Bash**，这是专为 Git 准备的命令行工具。

**方式二**：使用包管理器 Scoop：

```powershell
# 先安装 Scoop（在 PowerShell 中执行）
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
Invoke-RestMethod -Uri https://get.scoop.sh | Invoke-Expression

# 再安装 Git
scoop install git

# 验证
git -v
```

> **Windows 用户注意**：
> 1. 目录路径中不要包含中文
> 2. **不要用 Windows 自带的记事本编辑文件**，它会在文件开头添加特殊字符导致奇怪问题。推荐用 VS Code

---

## 3. 第一次使用：初始配置

安装完 Git 后，第一件事是告诉 Git 你是谁。这些信息会附在每次提交记录上。

```bash
git config --global user.name "Your Name"
git config --global user.email "youremail@example.com"
```

`--global` 表示对当前用户的所有仓库生效。如果某个项目需要用不同的邮箱（比如公司项目用公司邮箱），可以进入该项目目录后不加 `--global` 单独配置：

```bash
cd /path/to/company-project
git config user.email "work@company.com"
```

**开启彩色输出**（让命令结果更好看）：

```bash
git config --global color.ui true
```

**查看当前所有配置**：

```bash
git config --list
```

配置信息保存在 `~/.gitconfig` 文件中，可以直接查看：

```bash
cat ~/.gitconfig
# [user]
#     name = Your Name
#     email = youremail@example.com
# [color]
#     ui = true
```

---

## 4. 创建版本库

**版本库（Repository）**，也叫仓库，就是一个被 Git 管理的目录。这个目录里所有文件的每次修改和删除，Git 都能追踪，随时可以还原到任意历史版本。

### 初始化仓库

```bash
# 第一步：创建一个目录
mkdir learngit
cd learngit
pwd
# /Users/xxx/learngit

# 第二步：把这个目录初始化为 Git 仓库
git init
# Initialized empty Git repository in /Users/xxx/learngit/.git/
```

执行后，目录下会多出一个 `.git` 隐藏目录，这是 Git 的"大脑"，存储所有版本历史。**千万不要手动修改 `.git` 里的内容**，否则仓库就坏了。

看不到 `.git`？用这个命令显示隐藏文件：

```bash
ls -ah
# .  ..  .git
```

### 把文件加入版本库

> **重要说明**：Git **只能追踪文本文件**（.txt、.py、.js、.md 等）的具体修改内容。图片、视频等二进制文件也可以托管，但 Git 只知道"文件大小从 100KB 变成了 120KB"，不知道具体改了什么。Word 文件（.docx）也是二进制格式，无法追踪具体修改。
>
> 强烈建议使用 **UTF-8 编码**保存文件，兼容性最好。

创建一个文件并提交：

```bash
# 新建 readme.txt，写入如下内容：
# Git is a version control system.
# Git is free software.

# 第一步：将文件添加到暂存区
git add readme.txt
# 执行后没有任何输出，正常！Unix 的哲学是"没有消息就是好消息"

# 第二步：将暂存区内容提交到版本库
git commit -m "wrote a readme file"
# [master (root-commit) eaadf4e] wrote a readme file
#  1 file changed, 2 insertions(+)
#  create mode 100644 readme.txt
```

`-m` 后面是**提交说明**，一定要写清楚改了什么，方便日后查阅历史。

可以一次 `add` 多个文件，然后一起 `commit`：

```bash
git add file1.txt
git add file2.txt file3.txt
git commit -m "add 3 files"
```

**常见报错处理**：

| 报错信息 | 原因 | 解决方法 |
|----------|------|---------|
| `fatal: not a git repository` | 不在 Git 仓库目录中 | `cd` 进入仓库目录再执行 |
| `pathspec 'xxx' did not match any files` | 文件不存在或路径写错 | 用 `ls` 确认文件存在 |

---

## 5. 时光机穿梭：版本控制的核心操作

### 5.1 查看状态与修改

假设你修改了 `readme.txt`，把第一行改成：

```
Git is a distributed version control system.
```

**查看仓库当前状态**：

```bash
git status
# On branch master
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   modified:   readme.txt
# no changes added to commit
```

Git 告诉你：`readme.txt` 被修改了，但还没有提交。

**查看具体改了什么内容**：

```bash
git diff readme.txt
# diff --git a/readme.txt b/readme.txt
# --- a/readme.txt
# +++ b/readme.txt
# @@ -1,2 +1,2 @@
# -Git is a version control system.
# +Git is a distributed version control system.
#  Git is free software.
```

`-` 开头（红色）是删除的内容，`+` 开头（绿色）是新增的内容。

确认修改没问题后，提交：

```bash
git add readme.txt
git commit -m "add distributed"
# [master 1094adb] add distributed
#  1 file changed, 1 insertion(+), 1 deletion(-)
```

**好习惯**：每次提交前先用 `git status` 确认状态，再用 `git diff` 查看具体改动，确认无误再提交。

---

### 5.2 版本回退

每次 `commit` 都会生成一个版本快照，可以随时回退。

**查看所有提交历史**：

```bash
git log
# commit a6c5819e... (HEAD -> master)
# Author: xxx <xxx@example.com>
# Date:   Mon Mar 10 09:21:06 2025
#     append GPL
#
# commit 1094adb7...
#     add distributed
#
# commit eaadf4e...
#     wrote a readme file
```

信息太多？用 `--oneline` 简化：

```bash
git log --oneline
# a6c5819 (HEAD -> master) append GPL
# 1094adb add distributed
# eaadf4e wrote a readme file
```

**HEAD 是什么？**

`HEAD` 是一个指针，始终指向**当前所在的版本**。

```
HEAD → master → 最新 commit（a6c5819）
                    ↑
              上一个 commit（1094adb）
                    ↑
              更早的 commit（eaadf4e）
```

**回退到上一个版本**：

```bash
git reset --hard HEAD^
# HEAD is now at 1094adb add distributed
```

- `HEAD^` = 上一个版本
- `HEAD^^` = 上上个版本
- `HEAD~100` = 往上 100 个版本

**回退到指定版本**（通过 commit id）：

```bash
git reset --hard 1094adb
# 只需写前几位字符，Git 会自动补全
```

**回退后反悔了，怎么回到"未来"？**

用 `git log` 已经看不到"未来"的版本了。但 Git 提供了 `git reflog`，它记录了你**每一次操作**（包括回退操作）：

```bash
git reflog
# 1094adb (HEAD -> master) HEAD@{0}: reset: moving to HEAD^
# a6c5819 HEAD@{1}: commit: append GPL
# 1094adb HEAD@{2}: commit: add distributed
# eaadf4e HEAD@{3}: commit: wrote a readme file
```

找到目标版本的 commit id（这里是 `a6c5819`），再次 `git reset --hard` 即可回来：

```bash
git reset --hard a6c5819
```

> **原理**：Git 的版本回退之所以极快，是因为内部只是移动了 `HEAD` 指针的位置，根本没有删除任何数据。

---

### 5.3 工作区与暂存区（重要概念）

这是理解 Git 的**最关键概念**，搞懂了这个，Git 的大部分操作就都明白了。

```
┌──────────────────────────────────────────────────────────┐
│                        .git 版本库                        │
│  ┌──────────────────┐         ┌──────────────────────┐  │
│  │   暂存区 (Stage)  │─commit─▶│  版本库 (Repository)  │  │
│  │   .git/index     │         │  各版本的 commit 快照   │  │
│  └────────▲─────────┘         └──────────────────────┘  │
│           │ git add                                       │
└───────────┼──────────────────────────────────────────────┘
            │
┌───────────┴──────────────────────────────────────────────┐
│              工作区 (Working Directory)                    │
│           你直接看到、可以自由编辑文件的地方                  │
└──────────────────────────────────────────────────────────┘
```

| 区域 | 位置 | 说明 |
|------|------|------|
| **工作区** | 你的项目目录 | 直接编辑文件的地方 |
| **暂存区** | `.git/index` | `git add` 后文件进入的中间站 |
| **版本库** | `.git/` 目录内 | `git commit` 后永久保存的快照 |

**实验验证**——修改 `readme.txt`，同时新建一个 `LICENSE` 文件，看看状态：

```bash
git status
# Changes not staged for commit:   ← 已修改，在工作区，未 add
#   modified:   readme.txt
#
# Untracked files:                  ← 新文件，从未被 Git 追踪
#   LICENSE
```

执行 `git add readme.txt LICENSE` 后再看状态：

```bash
git status
# Changes to be committed:          ← 已 add，在暂存区，待 commit
#   new file:   LICENSE
#   modified:   readme.txt
```

`git commit` 后，暂存区清空，工作区变干净：

```bash
git commit -m "understand how stage works"
git status
# nothing to commit, working tree clean
```

---

### 5.4 管理修改：Git 到底追踪什么

**Git 追踪的是修改，而不是文件本身。**

这句话很关键，来看一个容易踩的坑：

```bash
# 第一次修改 readme.txt
git add readme.txt          # ← 第一次修改进入暂存区

# 再次修改 readme.txt（第二次修改）
git commit -m "git tracks changes"  # ← 只提交了暂存区里的第一次修改！
```

操作流程：`第一次修改 → git add → 第二次修改 → git commit`

此时提交的**只有第一次修改**，第二次修改没有 `add` 进暂存区，所以不会被提交。

验证：

```bash
git diff HEAD -- readme.txt
# 会显示第二次修改依然存在于工作区，尚未提交
```

**正确做法**：每次修改后都 `git add`，然后再 `git commit`：

```bash
第一次修改 → git add → 第二次修改 → git add → git commit
```

或者统一用 `git add .` 把所有修改一次加入暂存区：

```bash
git add .
git commit -m "提交所有修改"
```

---

### 5.5 撤销修改：后悔药怎么吃

根据"犯错程度"，Git 提供了不同级别的后悔药：

#### 情形一：只修改了工作区，还没 `git add`

```bash
# 比如刚在文件里写了不该写的内容，想撤销
git restore readme.txt         # Git 2.23+ 推荐写法
# 旧写法：git checkout -- readme.txt
```

效果：工作区的文件被还原到最近一次 `git add` 或 `git commit` 的状态，就像从未修改过。

> 注意：`git checkout -- file` 中的 `--` 非常重要，没有 `--` 就变成"切换到某个分支"的命令了！

#### 情形二：已经 `git add`，还没 `git commit`

```bash
# 第一步：把暂存区的修改撤回到工作区
git restore --staged readme.txt
# 旧写法：git reset HEAD readme.txt

# 第二步：再撤销工作区的修改（同情形一）
git restore readme.txt
```

#### 情形三：已经 `git commit`，但没有 push 到远程

```bash
git reset --hard HEAD^   # 回退到上一个版本
```

#### 三种情形小结

| 情形 | 解决方案 |
|------|---------|
| 改了工作区，未 add | `git restore <file>` |
| 已 add，未 commit | `git restore --staged <file>` → `git restore <file>` |
| 已 commit，未 push | `git reset --hard HEAD^` |
| 已 push | 复杂，需要强制推送，谨慎操作 |

---

### 5.6 删除文件

在 Git 中，"删除"也是一种修改操作。

```bash
# 先提交一个测试文件
git add test.txt
git commit -m "add test.txt"

# 从文件系统中删除
rm test.txt

# git status 会显示：deleted: test.txt
git status
# Changes not staged for commit:
#   deleted:    test.txt
```

**情况 A：确实要永久删除这个文件**

```bash
git rm test.txt           # 将删除操作登记到暂存区
git commit -m "remove test.txt"
```

> 提示：先手动 `rm test.txt`，再 `git rm test.txt`，和直接 `git rm test.txt`（一步删除工作区文件并登记）效果相同。

**情况 B：删错了，想从版本库恢复**

```bash
git restore test.txt      # 从版本库恢复到工作区
# 旧写法：git checkout -- test.txt
```

> **注意**：只有被 `commit` 过的文件才能恢复。从未提交过的文件一旦删除，无法找回。恢复后只能还原到最近一次提交的版本，最后一次提交之后的修改会丢失。

---

## 6. 远程仓库：代码上云

到目前为止，所有操作都在本地进行。Git 真正的杀手级功能之一是**远程仓库**——让代码可以备份到云端，并与他人协作。

**GitHub** 是全球最大的 Git 代码托管平台，免费账户可以创建公开和私有仓库。国内访问 GitHub 较慢时，可以使用 **Gitee**（第 10 章介绍）。

### 6.1 SSH 密钥配置

本地和 GitHub 之间的数据传输通过 SSH 协议加密，需要先配置密钥对。

**第一步：生成 SSH Key**

```bash
ssh-keygen -t rsa -C "youremail@example.com"
# 一路回车，不需要设置密码
```

完成后，在 `~/.ssh/` 目录下生成两个文件：

- `id_rsa`：**私钥**（绝对不能泄露！相当于你的身份证原件）
- `id_rsa.pub`：**公钥**（可以公开，需要上传到 GitHub）

查看公钥内容：

```bash
cat ~/.ssh/id_rsa.pub
# ssh-rsa AAAAB3NzaC1yc2EAAA... youremail@example.com
```

**第二步：将公钥添加到 GitHub**

1. 登录 GitHub
2. 右上角头像 → **Settings** → **SSH and GPG keys**
3. 点击 **New SSH key**
4. Title 随便填（比如 "My MacBook"），将 `id_rsa.pub` 的完整内容粘贴到 Key 里
5. 点击 **Add SSH key**

**验证连接**：

```bash
ssh -T git@github.com
# Hi username! You've successfully authenticated, but GitHub does not provide shell access.
```

看到 "successfully authenticated" 就说明配置成功了。

> **为什么需要 SSH Key？** GitHub 用公钥来识别推送者身份——只有持有对应私钥的人才能推送。一台电脑对应一对 Key，你可以把多台电脑的公钥都添加到 GitHub，从任何一台电脑都能推送。

---

### 6.2 关联并推送远程库

**场景**：你已经在本地创建了仓库，现在想推送到 GitHub。

**第一步**：在 GitHub 上创建一个新的空仓库（**不要勾选 "Initialize this repository with a README"**，否则会冲突）。

**第二步**：在本地仓库目录中关联远程库：

```bash
git remote add origin git@github.com:你的GitHub用户名/learngit.git
```

这里 `origin` 是远程仓库的默认名称，约定俗成，也可以改成其他名字，但不建议。

**第三步**：第一次推送，加 `-u` 参数：

```bash
git push -u origin master
# Counting objects: 20, done.
# ...
# Branch 'master' set up to track remote branch 'master' from 'origin'.
```

`-u` 参数的作用：不仅推送代码，还将本地 `master` 分支与远程 `master` 分支**关联**起来，以后推送和拉取就可以直接用简化命令。

**之后每次推送**：

```bash
git push
# 或明确指定
git push origin master
```

**第一次连接 GitHub 时的 SSH 警告**：

```
The authenticity of host 'github.com' can't be established.
RSA key fingerprint is xx:xx:xx...
Are you sure you want to continue connecting (yes/no)?
```

输入 `yes` 回车即可。Git 会把 GitHub 的公钥指纹加入本机信任列表，以后不会再出现这个警告。

**解除远程关联**（比如地址填错了）：

```bash
git remote -v                # 先查看当前关联的远程库
git remote rm origin         # 解除关联（只是断开连接，不会删除 GitHub 上的远程仓库）
```

---

### 6.3 从远程库克隆

**场景**：从零开始一个新项目，或者接手别人的项目。

**推荐做法**：先在 GitHub 上创建仓库（勾选 "Initialize with README"），然后克隆到本地：

```bash
# 克隆远程仓库（自动创建同名目录）
git clone git@github.com:michaelliao/gitskills.git

# 克隆到指定目录名
git clone git@github.com:michaelliao/gitskills.git my-project

# 进入目录，已经可以直接工作
cd gitskills
ls
# README.md
```

克隆完成后，Git 会自动配置好 `origin` 远程地址，直接就可以 push/pull。

> **SSH vs HTTPS 地址**：
> - SSH 格式：`git@github.com:username/repo.git`（推荐，配置一次不再需要每次输密码）
> - HTTPS 格式：`https://github.com/username/repo.git`（不需要 SSH Key，但每次 push 可能要输 Token）

---

## 7. 分支管理：Git 最强大的功能

分支是 Git 的核心特性，也是 Git 相比 SVN 最大的优势。

**什么是分支？** 想象平行宇宙——你在 `dev` 分支上大刀阔斧地开发新功能，完全不影响 `master` 上已经稳定运行的代码。开发完成后，把两个"宇宙"合并，大功告成。

**Git 分支为什么这么快？** 因为分支的本质只是一个**指针**。创建、切换、删除分支，都只是移动指针，不涉及文件复制，速度极快（毫秒级）。

### 7.1 创建与合并分支

```bash
# 查看所有分支（* 标记当前所在分支）
git branch
# * master

# 创建并切换到 dev 分支（一步到位，推荐）
git switch -c dev
# Switched to a new branch 'dev'
# 旧写法：git checkout -b dev

# 在 dev 上开发、提交
echo "new feature" >> readme.txt
git add readme.txt
git commit -m "branch test"

# 切换回 master
git switch master
# 旧写法：git checkout master

# 查看 readme.txt，你会发现刚才的修改"不见了"
# 因为那次提交在 dev 分支上，master 上没有

# 将 dev 分支合并到 master（Fast-forward 合并）
git merge dev
# Updating eaadf4e..1094adb
# Fast-forward
#  readme.txt | 1 +

# 删除已合并的 dev 分支
git branch -d dev
# Deleted branch dev (was 1094adb).

# 查看提交历史（已经变成一条直线）
git log --oneline
```

**Fast-forward（快进合并）**原理：

当 `master` 自从分叉后没有新提交，合并时 Git 直接把 `master` 指针"快进"到 `dev` 的最新提交——速度极快，但合并后历史是一条直线，看不出曾经有过分支。

```
合并前：
master: A → B
                 dev: A → B → C → D

Fast-forward 后：
master: A → B → C → D  （master 指针移到 D）
```

---

### 7.2 解决冲突

当两个分支都对**同一文件的同一位置**做了不同修改，合并时就会产生冲突，Git 无法自动决定保留谁的版本。

**模拟冲突**：

```bash
# 在 feature 分支上修改 readme.txt 最后一行
git switch -c feature
# 将最后一行改为：Creating a new branch is quick AND simple.
git add readme.txt && git commit -m "AND simple"

# 切回 master，也修改同一行
git switch master
# 将最后一行改为：Creating a new branch is quick & simple.
git add readme.txt && git commit -m "& simple"

# 合并——冲突！
git merge feature
# CONFLICT (content): Merge conflict in readme.txt
# Automatic merge failed; fix conflicts and then commit the result.
```

打开 `readme.txt`，会看到冲突标记：

```
<<<<<<< HEAD
Creating a new branch is quick & simple.
=======
Creating a new branch is quick AND simple.
>>>>>>> feature
```

- `<<<<<<< HEAD` 到 `=======`：当前分支（master）的内容
- `=======` 到 `>>>>>>> feature`：对方分支（feature）的内容

**解决步骤**：

1. 手动编辑文件，保留你想要的内容，**删掉所有冲突标记**（`<<<<<<<`、`=======`、`>>>>>>>`）
2. 保存文件后，`git add` 标记为已解决
3. `git commit` 完成合并

```bash
# 编辑后文件内容：Creating a new branch is quick and simple.
git add readme.txt
git commit -m "conflict fixed"
```

**查看带分支图的提交历史**：

```bash
git log --graph --oneline
# *   cf810e4 (HEAD -> master) conflict fixed
# |\
# | * 14096d0 AND simple
# * | 5dc6824 & simple
# |/
# * b17d20e branch test
```

---

### 7.3 分支管理策略

#### `--no-ff`：禁用快进合并，保留分支历史

Fast-forward 合并后，历史看起来是一条直线，无法看出曾经有过分支。使用 `--no-ff` 可以保留完整的分支历史：

```bash
git merge --no-ff -m "merge with no-ff" dev
```

对比效果：

```
Fast-forward（无分支历史）：    --no-ff（保留分支历史）：

master: A─B─C─D                master: A─B─────E
                                              ↗   ↖
                                        dev: C──D
```

#### 推荐的分支模型（Git Flow 简化版）

在团队项目中，一套清晰的分支规范可以避免很多混乱：

```
main/master  ──●──────────────────────────────●──  （仅接受稳定发布，打 tag）
               │                              │
               └─── dev ──●──●──●──●──●──●───┘     （日常开发主线）
                               │           │
                           feature/A   feature/B    （独立功能分支）
                                 hotfix/bug-101      （紧急修复分支）
```

| 分支类型 | 命名示例 | 说明 |
|----------|----------|------|
| 主分支 | `main` / `master` | 始终稳定，只接受来自 dev 的合并 |
| 开发分支 | `dev` | 日常开发，功能完成合并到 main |
| 功能分支 | `feature/user-login` | 单个功能，完成合并到 dev |
| 修复分支 | `hotfix/bug-101` | 紧急 Bug，从 main 拉出，修完合回 main 和 dev |

---

### 7.4 Bug 分支：用 stash 保存现场

**经典场景**：正在 `dev` 上开发新功能，代码写了一半，突然接到紧急 Bug 修复任务——Bug 在 `master` 上，两小时内必须修复！

**问题**：开发到一半的代码无法提交（不完整），怎么临时"冻结"现场？

**解决方案：`git stash`**

```bash
# 第一步：储藏当前工作现场（工作区+暂存区全部冻结）
git stash
# Saved working directory and index state WIP on dev: f52c633 add merge

# 工作区现在变干净了，可以安心切换分支
git status
# nothing to commit, working tree clean

# 第二步：切到 master，从 master 创建 bug 修复分支
git switch master
git switch -c hotfix/issue-101

# 第三步：修复 Bug，提交
# 把 "Git is free software ..." 改为 "Git is a free software ..."
git add readme.txt
git commit -m "fix bug 101"

# 第四步：合并回 master，删除临时分支
git switch master
git merge --no-ff -m "merged bugfix 101" hotfix/issue-101
git branch -d hotfix/issue-101

# 第五步：回到 dev，恢复工作现场
git switch dev
git stash pop      # 恢复最近的 stash 并删除记录
```

**stash 常用命令**：

```bash
git stash list                # 查看所有储藏记录
# stash@{0}: WIP on dev: f52c633 add merge

git stash pop                 # 恢复最近的 stash 并删除记录
git stash apply               # 只恢复，不删除记录（可多次 apply）
git stash drop                # 手动删除最近的 stash 记录
git stash apply stash@{1}     # 恢复指定的 stash
```

#### 用 cherry-pick 同步 Bug 修复

`master` 上修复了 Bug，`dev` 分支是早期从 `master` 分出来的，同样的 Bug 也存在于 `dev`。怎么把这个修复同步过来，而不是整个 `master` 合并到 `dev`？

```bash
# 把 master 上那次修复提交（比如 commit id 是 4c805e2）"摘取"到 dev 分支
git switch dev
git cherry-pick 4c805e2
# [dev 1d4b803] fix bug 101
```

`cherry-pick` 会在当前分支创建一个内容相同但 commit id 不同的新提交。这样就不需要重复修复同一个 Bug。

---

### 7.5 Feature 分支

每开发一个新功能，建议新建一个独立的 `feature` 分支，防止实验性代码污染主线。

```bash
# 新建 feature 分支并开始开发
git switch -c feature/vulcan

# 开发完成，提交
git add vulcan.py
git commit -m "add feature vulcan"

# 切回 dev，合并
git switch dev
git merge --no-ff -m "merge feature vulcan" feature/vulcan

# 删除 feature 分支
git branch -d feature/vulcan
```

**功能突然被取消，怎么丢弃没有合并过的分支？**

用 `-d` 会报错（防止误删），需要用大写 `-D` 强制删除：

```bash
git branch -d feature/vulcan
# error: The branch 'feature/vulcan' is not fully merged.
# If you are sure you want to delete it, run 'git branch -D feature/vulcan'.

git branch -D feature/vulcan
# Deleted branch feature/vulcan (was 287773e).
```

---

### 7.6 多人协作

当你克隆远程仓库时，Git 自动把本地 `master` 和远程 `master` 对应起来，远程仓库的默认名字是 `origin`。

**查看远程库信息**：

```bash
git remote
# origin

git remote -v
# origin  git@github.com:xxx/learngit.git (fetch)
# origin  git@github.com:xxx/learngit.git (push)
```

**推送分支到远程**：

```bash
git push origin master    # 推送 master（必须始终与远程同步）
git push origin dev       # 推送 dev（团队共享开发分支）
```

哪些分支需要推送？

- `master`：主分支，必须始终与远程同步
- `dev`：团队共享开发分支，需要推送
- `hotfix` / `bug` 分支：本地临时用，不需要推送
- `feature` 分支：取决于是否和他人协作开发

**抓取远程分支并在本地开发**：

同事克隆仓库后，默认只能看到本地的 `master`。如果要在 `dev` 上开发：

```bash
git checkout -b dev origin/dev    # 创建本地 dev 并与远程 origin/dev 关联
```

**多人协作冲突处理全流程**：

```bash
# 1. 尝试推送自己的修改
git push origin dev

# 2. 推送失败（远程有别人新推送的提交）
# error: failed to push some refs to ...
# hint: Updates were rejected because the tip of your current branch is behind

# 3. 先拉取远程最新内容
git pull
# 如果提示 "no tracking information"，先建立关联：
git branch --set-upstream-to=origin/dev dev
git pull

# 4. 如果有冲突，手动解决后提交
# 编辑冲突文件 → 删掉冲突标记 → 保存
git add .
git commit -m "fix merge conflict"

# 5. 再次推送，成功！
git push origin dev
```

**多人协作工作模式总结**：

1. `git push origin <branch>` 推送自己的修改
2. 推送失败 → `git pull` 同步远程最新内容
3. 有冲突 → 本地解决冲突 → 提交
4. 再次 `git push` 成功

---

### 7.7 Rebase：让历史变成一条直线

多人协作时，提交历史经常分叉：

```bash
git log --graph --oneline
# *   e0ea545 (HEAD -> master) Merge branch 'master' of github.com:xxx/learngit
# |\
# | * f005ed4 (origin/master) set exit=1
# * | 582d922 add author
# * | 8875536 add comment
# |/
# * d1be385 init hello
```

这种分叉历史对强迫症不友好。`git rebase`（变基）可以把分叉整理成一条直线：

```bash
git rebase
# First, rewinding head to replay your work on top of it...
# Applying: add comment
# Applying: add author

git log --graph --oneline
# * 7e61ed4 (HEAD -> master) add author
# * 3611cfe add comment
# * f005ed4 (origin/master) set exit=1
# * d1be385 init hello
```

直线了！现在推送：

```bash
git push origin master
```

**rebase 原理**：Git 把你本地相比远程"多出来"的那些提交，"移植"到远程最新提交之后，就像它们原本就是基于最新版本写的。操作前后最终代码内容不变，但 commit id 会改变。

**rebase vs merge 对比**：

| 操作 | 历史形状 | 何时用 |
|------|----------|--------|
| `merge` | 有合并节点，保留完整历史 | 合并公共分支，保留完整上下文 |
| `rebase` | 一条直线，更简洁 | 整理本地未推送的提交，推送前清理历史 |

> **黄金法则**：**不要对已经 push 到远程的公共分支执行 rebase**。因为 rebase 会改写 commit id，其他人本地的历史会与远程不一致，引发严重混乱。Rebase 只用于整理自己本地未推送的提交。

---

## 8. 标签管理

每次发布软件新版本，都对应某一个 commit。但 commit id 是一串难记的哈希值，比如 `1094adb7b9b3807...`，没人能记住。

**标签（Tag）** 就是给某个 commit 起一个有意义的别名，比如 `v1.0.0`，让版本管理更直观：

> "请把上周一的那个版本打包发布，commit 号是 6a5819e..."  "一串乱码不好找！"
>
> 改成 → "请把 v1.2 版本打包发布" "好的，按 tag v1.2 找就行！"

**标签 vs 分支**：分支可以移动（指向最新提交），标签创建后**永远固定**在那个 commit 上，不能移动。

### 创建标签

```bash
# 切换到需要打标签的分支
git switch master

# 给当前最新提交打一个轻量标签
git tag v1.0

# 查看所有标签（按字母序，不是时间序）
git tag
# v1.0

# 给历史上某次提交打标签（先查看 commit id）
git log --oneline
# 12a631b (HEAD -> master) merged bug fix 101
# 4c805e2 fix bug 101
# f52c633 add merge
# ...

git tag v0.9 f52c633    # 给历史提交 f52c633 打标签

# 查看某个标签的信息
git show v0.9
```

**带说明的附注标签（推荐正式发布时使用）**：

```bash
git tag -a v1.0 -m "version 1.0 released" 12a631b
```

附注标签包含打标签人的姓名、邮箱、时间和说明，信息更完整，适合正式版本发布。

### 操作标签

```bash
# 删除本地标签（创建后不会自动推送，打错了可以安全删除）
git tag -d v0.1
# Deleted tag 'v0.1' (was f15b0dd)

# 推送单个标签到远程
git push origin v1.0

# 一次性推送所有未推送的标签
git push origin --tags

# 删除远程标签（需要先删除本地，再删除远程）
git tag -d v0.9
git push origin :refs/tags/v0.9
# 登录 GitHub 可以验证是否删除成功
```

---

## 9. 使用 GitHub 参与开源

GitHub 不只是代码托管，它还是一个**全球最大的开源协作社区**。利用 Git 强大的分支和克隆功能，任何人都可以参与开源项目。

### Fork + Pull Request 工作流

假设你想给开源项目 `bootstrap` 修复一个 Bug 或贡献新功能：

**关系图**：

```
GitHub 上：
  twbs/bootstrap（官方仓库，你没有写权限）
        │  Fork（在 GitHub 页面点击 Fork 按钮）
        ↓
  your-name/bootstrap（你的 Fork，你有完全控制权）
        │  git clone
        ↓
  本地 bootstrap（你的本地开发环境）
```

**操作流程**：

1. 在 GitHub 上点击 **Fork** 按钮，在自己账号下复制一份
2. **克隆自己 Fork 的仓库**（注意：不是克隆官方仓库）：

   ```bash
   git clone git@github.com:your-name/bootstrap.git
   cd bootstrap
   ```

3. 在本地新建分支，修复 Bug 或开发功能：

   ```bash
   git switch -c fix/typo-in-readme
   # 修改代码...
   git add .
   git commit -m "fix typo in README"
   git push origin fix/typo-in-readme
   ```

4. 在 GitHub 上，你的仓库页面会提示可以发起 **Pull Request**
5. 填写 PR 说明，提交给官方仓库作者审核
6. 作者接受后，你的代码就合并进了开源项目！

> **小结**：
> - 可以 Fork 任何开源仓库
> - Fork 后你对自己账号下的副本有完全写权限
> - 通过 Pull Request 向原仓库贡献代码

---

## 10. 国内替代：使用 Gitee

GitHub 在国内访问有时较慢。**Gitee（码云）** 是国内的 Git 托管平台，使用体验类似 GitHub，速度更快。

### 配置 Gitee

1. 在 [gitee.com](https://gitee.com) 注册账号
2. 添加 SSH 公钥：头像 → 设置 → SSH 公钥 → 粘贴 `id_rsa.pub` 内容（和 GitHub 一样的公钥就行）

### 关联 Gitee 远程库

```bash
# 如果本地已关联 GitHub，先查看
git remote -v
# origin  git@github.com:xxx/learngit.git (fetch)
# origin  git@github.com:xxx/learngit.git (push)

# 删除原来的 GitHub 关联
git remote rm origin

# 关联 Gitee
git remote add origin git@gitee.com:your-name/learngit.git
git push -u origin master
```

### 同时关联 GitHub 和 Gitee（双备份）

```bash
# 删除原有 origin
git remote rm origin

# 分别起不同名字关联两个平台
git remote add github git@github.com:your-name/learngit.git
git remote add gitee  git@gitee.com:your-name/learngit.git

# 查看
git remote -v
# gitee   git@gitee.com:your-name/learngit.git (fetch)
# gitee   git@gitee.com:your-name/learngit.git (push)
# github  git@github.com:your-name/learngit.git (fetch)
# github  git@github.com:your-name/learngit.git (push)

# 分别推送到两个平台
git push github master
git push gitee  master
```

这样本地一份代码，同时备份到两个平台。

---

## 11. 自定义 Git

### 11.1 忽略特殊文件 `.gitignore`

有些文件不应该被 Git 追踪：

- 编译产物（`*.pyc`、`*.class`、`dist/`）
- 依赖目录（`node_modules/`、`venv/`）
- 含敏感信息的配置文件（`.env`、`db.ini`）
- 系统自动生成的文件（`.DS_Store`、`Thumbs.db`）

在仓库根目录新建 `.gitignore` 文件，写入忽略规则：

```gitignore
# 操作系统文件
.DS_Store
Thumbs.db
ehthumbs.db
Desktop.ini

# Python
__pycache__/
*.py[cod]
*.so
*.egg
*.egg-info/
dist/
build/
.env
venv/

# Node.js
node_modules/
dist/
.env.local

# IDE
.vscode/
.idea/
*.swp

# 敏感配置
db.ini
deploy_key_rsa
```

**规则语法速查**：

```gitignore
*.log          # 忽略所有 .log 文件（* 是通配符）
build/         # 忽略 build 目录（/ 结尾表示目录）
doc/*.txt      # 忽略 doc/ 下的 .txt 文件（不含子目录）
doc/**/*.txt   # 忽略 doc/ 下所有层级的 .txt 文件
!important.log # 例外：即使匹配了上面的规则，也不忽略这个文件
```

**强制添加被忽略的文件**（-f 强制）：

```bash
git add -f App.class
```

**检查某文件为何被忽略**：

```bash
git check-ignore -v App.class
# .gitignore:3:*.class    App.class
# 第 3 行的 *.class 规则导致的
```

> `.gitignore` 文件本身必须提交到 Git 仓库，这样团队所有人共享同一套忽略规则。
>
> 各语言的 `.gitignore` 模板：[github.com/github/gitignore](https://github.com/github/gitignore)
> 在线生成工具：[gitignore.io](https://gitignore.io)

---

### 11.2 配置别名

常用命令打起来太长？配置别名简化：

```bash
git config --global alias.st   status
git config --global alias.co   checkout
git config --global alias.ci   commit
git config --global alias.br   branch
git config --global alias.unstage 'reset HEAD'
git config --global alias.last    'log -1'
```

配置后使用：

```bash
git st              # git status
git co dev          # git checkout dev
git ci -m "xxx"     # git commit -m "xxx"
git unstage file    # git reset HEAD file
git last            # 查看最近一次提交信息
```

**超实用的彩色图形日志别名**：

```bash
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```

执行 `git lg`，提交历史变得彩色、直观、好看。

**查看和修改配置文件**：

```bash
# 当前仓库的配置（保存在 .git/config）
cat .git/config
# [remote "origin"]
#     url = git@github.com:...
# [alias]
#     last = log -1

# 全局配置（保存在 ~/.gitconfig）
cat ~/.gitconfig
# [alias]
#     st = status
#     co = checkout
```

直接编辑这两个文件也可以修改配置，删除别名只需删掉对应行。

---

### 11.3 搭建私有 Git 服务器

如果不想用 GitHub/Gitee，可以在自己的 Linux 服务器上搭建私有 Git 仓库，适合企业内部代码托管。

**环境要求**：Ubuntu/Debian 系统，有 `sudo` 权限。

```bash
# 第一步：安装 Git
sudo apt install git

# 第二步：创建 git 用户（专门运行 git 服务）
sudo adduser git

# 第三步：配置证书登录
# 收集所有团队成员的 id_rsa.pub 内容，每行一个，写入：
sudo vim /home/git/.ssh/authorized_keys

# 第四步：初始化裸仓库（裸仓库没有工作区，纯粹用于共享）
sudo git init --bare /srv/sample.git
sudo chown -R git:git /srv/sample.git

# 第五步：禁止 git 用户登录 Shell（安全考虑）
sudo vim /etc/passwd
# 找到这行：git:x:1001:1001:,,,:/home/git:/bin/bash
# 改为：    git:x:1001:1001:,,,:/home/git:/usr/bin/git-shell
# 这样 git 用户可以通过 SSH 使用 Git，但无法登录 Shell

# 第六步：团队成员克隆使用
git clone git@your-server-ip:/srv/sample.git
```

> 团队较大时，推荐部署 **Gitea**（开源的自托管 GitHub 替代品，有 Web 界面，功能完整，安装简单）。需要精细权限控制时可以使用 **Gitolite**。

---

## 12. 图形界面工具 SourceTree

熟练掌握命令行后，可以用图形界面工具进一步提升效率。**SourceTree**（Atlassian 出品，免费）是最受欢迎的 Git GUI 工具之一。

### 基本操作对照

**添加仓库**：
- 把本地仓库文件夹拖入 SourceTree
- 或选择 "New" → "Clone from URL" 从远程克隆

**提交**：
- "File Status" 面板显示所有已修改文件（Unstaged）
- 勾选文件 → 移入 Staged（等价于 `git add`）
- 填写 Commit 说明 → 点击 "Commit"（等价于 `git commit`）

**切换分支**：
- 左侧 "BRANCHES" 列出所有本地分支
- 右键 → "Check out xxx"（等价于 `git switch xxx`）

**合并分支**：
- 右键目标分支 → "Merge xxx into current branch"（等价于 `git merge xxx`）

**推送/拉取**：
- 工具栏 "Pull" / "Push" 按钮（等价于 `git pull` / `git push`）

> **重要**：SourceTree 本质上是命令行的图形封装，出错时会显示实际执行的 Git 命令和错误信息。**建议先熟练掌握命令行，再用 SourceTree 提效**——只会 GUI 而不懂命令，遇到复杂问题会束手无策。

---

## 13. 常用命令速查表

### 仓库初始化

| 命令 | 说明 |
|------|------|
| `git init` | 初始化本地仓库 |
| `git clone <url>` | 克隆远程仓库 |
| `git config --global user.name "xxx"` | 设置全局用户名 |
| `git config --global user.email "xxx"` | 设置全局邮箱 |
| `git config --list` | 查看所有配置 |

### 查看状态与历史

| 命令 | 说明 |
|------|------|
| `git status` | 查看工作区/暂存区状态 |
| `git diff` | 工作区 vs 暂存区（未 add 的改动） |
| `git diff --cached` | 暂存区 vs 版本库（已 add 未 commit） |
| `git diff HEAD` | 工作区 vs 最新版本 |
| `git diff HEAD -- <file>` | 指定文件：工作区 vs 最新版本 |
| `git log` | 查看提交历史 |
| `git log --oneline` | 简洁历史 |
| `git log --graph --oneline` | 图形化历史 |
| `git reflog` | 查看所有操作记录（包括回退） |

### 提交文件

| 命令 | 说明 |
|------|------|
| `git add <file>` | 添加指定文件到暂存区 |
| `git add .` | 添加当前目录所有修改到暂存区 |
| `git commit -m "说明"` | 提交暂存区内容到版本库 |

### 撤销与回退

| 命令 | 说明 |
|------|------|
| `git restore <file>` | 撤销工作区修改 |
| `git restore --staged <file>` | 将文件从暂存区撤回工作区 |
| `git reset --hard HEAD^` | 回退到上一个版本 |
| `git reset --hard <id>` | 回退到指定 commit |
| `git checkout -- <file>` | （旧写法）撤销工作区修改 |
| `git reset HEAD <file>` | （旧写法）撤销暂存区 |

### 删除文件

| 命令 | 说明 |
|------|------|
| `git rm <file>` | 删除文件并登记到暂存区 |
| `git rm --cached <file>` | 仅从暂存区删除，保留工作区文件 |

### 远程仓库

| 命令 | 说明 |
|------|------|
| `git remote add origin <url>` | 关联远程仓库 |
| `git remote -v` | 查看远程仓库信息 |
| `git remote rm origin` | 解除远程关联 |
| `git push -u origin master` | 首次推送（同时关联分支） |
| `git push` | 推送到已关联的远程分支 |
| `git push origin <branch>` | 推送指定分支 |
| `git pull` | 拉取并合并远程更新 |
| `git fetch` | 只拉取，不合并 |

### 分支管理

| 命令 | 说明 |
|------|------|
| `git branch` | 查看本地分支 |
| `git branch -a` | 查看所有分支（含远程） |
| `git branch <name>` | 创建分支 |
| `git switch <name>` | 切换分支 |
| `git switch -c <name>` | 创建并切换分支 |
| `git merge <name>` | 合并指定分支到当前分支 |
| `git merge --no-ff <name>` | 禁用快进合并，保留分支历史 |
| `git branch -d <name>` | 删除已合并的分支 |
| `git branch -D <name>` | 强制删除分支 |
| `git cherry-pick <id>` | 复制某次提交到当前分支 |
| `git rebase` | 变基，整理提交历史 |
| `git branch --set-upstream-to=origin/<branch> <branch>` | 建立本地与远程分支的追踪关系 |

### stash 暂存工作现场

| 命令 | 说明 |
|------|------|
| `git stash` | 储藏当前工作现场 |
| `git stash list` | 查看所有储藏 |
| `git stash pop` | 恢复最近储藏并删除记录 |
| `git stash apply` | 恢复最近储藏但保留记录 |
| `git stash apply stash@{n}` | 恢复指定储藏 |
| `git stash drop` | 删除最近储藏记录 |

### 标签管理

| 命令 | 说明 |
|------|------|
| `git tag <name>` | 创建轻量标签 |
| `git tag -a <name> -m "说明"` | 创建附注标签 |
| `git tag <name> <commit-id>` | 给历史提交创建标签 |
| `git tag` | 查看所有标签 |
| `git show <tagname>` | 查看标签详情 |
| `git tag -d <name>` | 删除本地标签 |
| `git push origin <tagname>` | 推送标签到远程 |
| `git push origin --tags` | 推送所有未推送的标签 |
| `git push origin :refs/tags/<name>` | 删除远程标签 |

---

## 附录：Git 工作流一图流

```
                          远程仓库 (GitHub / Gitee)
                          ┌──────────────────────┐
                          │   origin/master        │
                          └──────────┬─────────────┘
                   git push          │          git pull / fetch
                          │          ↓          │
┌─────────────────────────────────────────────────────────────┐
│                         本地环境                              │
│                                                             │
│  工作区               暂存区 (Stage)      版本库 (Repository) │
│  ┌──────────┐         ┌───────────┐      ┌──────────────┐  │
│  │ 修改文件  │──add──▶│ .git/index │─commit─▶│.git/objects  │ │
│  └──────────┘         └───────────┘      └──────────────┘  │
│       ↑                    ↑                    ↑           │
│  restore <file>    restore --staged      reset --hard       │
└─────────────────────────────────────────────────────────────┘
```

**学习路线建议**：

| 阶段 | 掌握内容 |
|------|---------|
| 入门 | init、add、commit、status、diff、log |
| 基础 | reset、restore、rm、remote、push、pull、clone |
| 进阶 | branch、switch、merge、conflict 解决、stash、cherry-pick |
| 高级 | rebase、tag、.gitignore、别名配置、多人协作工作流 |

> **最后的建议**：理解"工作区 → 暂存区 → 版本库"三层模型是掌握 Git 的钥匙。分支要敢于多用，提交说明要认真写，三个月后你会感谢现在认真的自己。

---

> 参考资料：廖雪峰 Git 教程（2025-06-16）[https://liaoxuefeng.com/books/git/](https://liaoxuefeng.com/books/git/)
