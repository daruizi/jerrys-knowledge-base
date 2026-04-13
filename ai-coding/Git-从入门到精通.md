# Git 教程

> 史上最浅显易懂的 Git 教程。没有接触过版本控制概念的读者也可以轻松入门，边学边练，学完即用。

---

## 目录

1. [Git 是什么](#1-git-是什么)
2. [安装 Git](#2-安装-git)
3. [初始配置](#3-初始配置)
4. [创建版本库](#4-创建版本库)
5. [时光机穿梭：版本控制基础](#5-时光机穿梭版本控制基础)
   - 5.1 [版本回退](#51-版本回退)
   - 5.2 [工作区与暂存区](#52-工作区与暂存区)
   - 5.3 [管理修改](#53-管理修改)
   - 5.4 [撤销修改](#54-撤销修改)
   - 5.5 [删除文件](#55-删除文件)
6. [远程仓库](#6-远程仓库)
   - 6.1 [SSH 密钥配置](#61-ssh-密钥配置)
   - 6.2 [添加远程库](#62-添加远程库)
   - 6.3 [从远程库克隆](#63-从远程库克隆)
7. [分支管理](#7-分支管理)
   - 7.1 [创建与合并分支](#71-创建与合并分支)
   - 7.2 [解决冲突](#72-解决冲突)
   - 7.3 [分支管理策略](#73-分支管理策略)
   - 7.4 [Bug 分支（stash）](#74-bug-分支stash)
   - 7.5 [Feature 分支](#75-feature-分支)
   - 7.6 [多人协作](#76-多人协作)
   - 7.7 [Rebase](#77-rebase)
8. [标签管理](#8-标签管理)
   - 8.1 [创建标签](#81-创建标签)
   - 8.2 [操作标签](#82-操作标签)
9. [自定义 Git](#9-自定义-git)
   - 9.1 [忽略特殊文件 .gitignore](#91-忽略特殊文件-gitignore)
   - 9.2 [配置别名](#92-配置别名)
10. [常用命令速查](#10-常用命令速查)

---

## 1. Git 是什么

### 版本控制系统的诞生

软件开发中，代码文件会不断地被修改。如果没有版本控制，你只能手动复制文件来备份，既低效又容易出错。版本控制系统（VCS）解决了这个问题——它记录每一次修改，让你随时可以查看历史版本，甚至"穿越回过去"。

### 集中式 vs 分布式

| 特性 | 集中式（CVS/SVN） | 分布式（Git） |
|------|------------------|--------------|
| 版本库位置 | 只在中央服务器 | 每人本地都有完整版本库 |
| 联网要求 | 必须联网才能提交 | 本地即可提交，联网再同步 |
| 安全性 | 服务器故障则数据丢失 | 任意一台机器都可恢复 |
| 速度 | 受网络速度限制 | 本地操作极快 |

**Git 是目前世界上最先进的分布式版本控制系统**，由 Linux 之父 Linus Torvalds 于 2005 年创造，用两周时间用 C 语言编写完成。

---

## 2. 安装 Git

### macOS

```bash
# 方式一：通过 Homebrew 安装（推荐）
brew install git

# 方式二：安装 Xcode Command Line Tools（自带 Git）
xcode-select --install
```

### Linux（Debian/Ubuntu）

```bash
sudo apt-get install git
```

### Windows

前往 [git-scm.com](https://git-scm.com) 下载安装包，一路默认安装即可。安装完成后在开始菜单找到 **Git Bash** 启动。

### 验证安装

```bash
git --version
# git version 2.x.x
```

---

## 3. 初始配置

安装完成后，第一件事是设置你的用户名和邮箱，这些信息会附在每次提交记录上。

```bash
git config --global user.name "Your Name"
git config --global user.email "youremail@example.com"
```

> `--global` 表示全局配置，对当前用户所有仓库生效。如果某个仓库需要单独配置，去掉 `--global` 即可（在仓库目录内执行）。

查看当前配置：

```bash
git config --list
```

---

## 4. 创建版本库

**版本库（Repository）** 就是一个被 Git 管理的目录，Git 会追踪其中所有文件的变化历史。

### 初始化仓库

```bash
# 1. 创建目录并进入
mkdir my-project
cd my-project

# 2. 初始化为 Git 仓库
git init
# Initialized empty Git repository in /path/to/my-project/.git/
```

此时目录下多出一个 `.git` 隐藏目录，这就是 Git 的版本库核心，**不要手动修改它**。

### 添加文件并提交

```bash
# 1. 创建一个文件
echo "Hello, Git!" > readme.txt

# 2. 将文件添加到暂存区（可添加多个文件）
git add readme.txt

# 3. 将暂存区内容提交到版本库
git commit -m "第一次提交：添加 readme.txt"
```

`-m` 后面是提交说明，**务必写清楚**，方便日后查阅历史。

---

## 5. 时光机穿梭：版本控制基础

### 5.1 版本回退

每次 `git commit` 都会生成一个**快照（版本）**，可以随时回退。

**查看提交历史**

```bash
git log
# 输出每次提交的 commit id（SHA1）、作者、时间、说明
```

加 `--oneline` 参数简化输出：

```bash
git log --oneline
# a1b2c3d 修复登录 Bug
# e4f5g6h 添加用户注册功能
# 7i8j9k0 第一次提交：添加 readme.txt
```

**回退到上一个版本**

```bash
git reset --hard HEAD^
```

- `HEAD` 表示当前版本
- `HEAD^` 表示上一个版本
- `HEAD^^` 表示上上个版本
- `HEAD~100` 表示往上 100 个版本

**回退到指定版本（通过 commit id）**

```bash
git reset --hard a1b2c3d
```

只需写前几位字符，Git 会自动匹配。

**后悔了？找回未来的版本**

回退后反悔，可通过 `git reflog` 找到所有历史操作记录：

```bash
git reflog
# a1b2c3d HEAD@{0}: reset: moving to a1b2c3d
# e4f5g6h HEAD@{1}: commit: 添加用户注册功能
# ...
```

再次 `git reset --hard <commit-id>` 即可恢复。

---

### 5.2 工作区与暂存区

理解 Git 的三个区域是掌握 Git 的关键：

```
工作区 (Working Directory)
   ↓  git add
暂存区 (Stage / Index)
   ↓  git commit
版本库 (Repository / .git)
```

| 区域 | 说明 |
|------|------|
| **工作区** | 你能看到的目录，直接编辑文件的地方 |
| **暂存区** | `git add` 后文件进入的中间区域，在 `.git/index` 中 |
| **版本库** | `git commit` 后永久保存的快照 |

**查看状态**

```bash
git status
```

输出示例：
```
On branch main
Changes to be committed:        ← 已 add，待 commit（暂存区）
  modified: readme.txt

Changes not staged for commit:  ← 已修改，未 add（工作区）
  modified: app.py

Untracked files:                ← 新文件，未被 Git 追踪
  new-file.txt
```

**查看修改内容**

```bash
git diff               # 工作区 vs 暂存区
git diff --cached      # 暂存区 vs 版本库（即将提交的内容）
git diff HEAD          # 工作区 vs 版本库最新版本
```

---

### 5.3 管理修改

Git 跟踪的是**修改**，而非文件本身。每次 `git add` 只是把**当前那一刻的修改**加入暂存区。

**典型问题**：修改文件 → `git add` → **再次修改文件** → `git commit`

此时提交的只有第一次修改，第二次修改没有被提交，因为没有再次 `git add`。

**最佳实践**：每次 `commit` 前，用 `git status` 确认暂存区内容。

---

### 5.4 撤销修改

**情形一：修改了工作区，但还没 `git add`**

```bash
git restore readme.txt        # Git 2.23+ 推荐写法
# 或旧写法：
git checkout -- readme.txt
```

效果：工作区的文件被还原为暂存区（或版本库）中的版本。

**情形二：已经 `git add`，还没 `git commit`**

```bash
# 第一步：将暂存区的修改撤回到工作区
git restore --staged readme.txt
# 或旧写法：
git reset HEAD readme.txt

# 第二步：再撤销工作区的修改（同情形一）
git restore readme.txt
```

**情形三：已经 `git commit`，但没有推送到远程**

```bash
git reset --hard HEAD^   # 回退到上一个版本
```

> 如果已经推送到远程仓库，撤销会变得复杂，要格外谨慎。

---

### 5.5 删除文件

```bash
# 1. 删除工作区文件
rm readme.txt

# 2a. 确认删除，将删除操作提交到暂存区
git rm readme.txt
git commit -m "删除 readme.txt"

# 2b. 误删？从版本库恢复
git restore readme.txt
```

---

## 6. 远程仓库

Git 是分布式系统，通常需要一台"中央服务器"用于团队同步。最常用的是 **GitHub**（国际）或 **Gitee**（国内）。

### 6.1 SSH 密钥配置

本地与 GitHub 的通信通过 SSH 协议加密，需要先配置密钥对。

**第一步：生成 SSH Key**

```bash
ssh-keygen -t rsa -C "youremail@example.com"
```

一路回车，密钥文件生成在 `~/.ssh/` 目录：
- `id_rsa`：私钥（**保密，不可泄露**）
- `id_rsa.pub`：公钥（可以公开）

**第二步：将公钥添加到 GitHub**

1. 登录 GitHub → Settings → SSH and GPG keys → New SSH key
2. 粘贴 `id_rsa.pub` 的内容
3. 点击保存

**验证连接**

```bash
ssh -T git@github.com
# Hi username! You've successfully authenticated...
```

---

### 6.2 添加远程库

在 GitHub 上创建一个新仓库（空仓库），然后将本地仓库与之关联：

```bash
# 关联远程仓库（origin 是约定俗成的远程名）
git remote add origin git@github.com:yourname/my-project.git

# 第一次推送，-u 参数将本地 main 分支与远程 main 关联
git push -u origin main
```

之后每次推送只需：

```bash
git push
```

**查看远程仓库信息**

```bash
git remote -v
```

---

### 6.3 从远程库克隆

从远程仓库下载到本地：

```bash
git clone git@github.com:yourname/my-project.git

# 克隆到指定目录名
git clone git@github.com:yourname/my-project.git my-local-name
```

克隆后，Git 会自动设置好 `origin` 远程地址。

---

## 7. 分支管理

分支是 Git 最强大的特性之一。可以把分支想象成**平行宇宙**——在独立的分支上开发新功能，不会影响主线，开发完成后再合并进来。

Git 的分支操作极快（创建/切换/删除仅需毫秒），因为本质上只是移动指针。

### 7.1 创建与合并分支

```bash
# 查看所有分支（* 标记当前分支）
git branch

# 创建新分支
git branch dev

# 切换到 dev 分支
git switch dev       # Git 2.23+ 推荐
# 或旧写法：
git checkout dev

# 创建并切换（一步到位）
git switch -c dev
# 或旧写法：
git checkout -b dev

# 在 dev 上开发并提交...
git add .
git commit -m "在 dev 上完成新功能"

# 切换回 main，将 dev 合并进来
git switch main
git merge dev

# 删除已合并的分支
git branch -d dev
```

**Fast-forward 合并**：如果 main 没有新提交，Git 直接将 main 指针移到 dev 的最新提交，这就是"快进合并"，速度极快。

---

### 7.2 解决冲突

当两个分支都对同一文件的同一位置做了修改，合并时会发生**冲突**。

```bash
git merge feature-branch
# CONFLICT (content): Merge conflict in readme.txt
# Automatic merge failed; fix conflicts and then commit the result.
```

冲突文件会被标记为：

```
<<<<<<< HEAD
这是 main 分支的内容
=======
这是 feature-branch 的内容
>>>>>>> feature-branch
```

**解决步骤：**

1. 手动编辑冲突文件，保留需要的内容，删除标记符
2. `git add` 标记为已解决
3. `git commit` 完成合并

```bash
# 查看分支合并图
git log --graph --oneline
```

---

### 7.3 分支管理策略

默认的 Fast-forward 合并不会留下分支记录。使用 `--no-ff` 参数可以保留分支历史：

```bash
git merge --no-ff -m "合并 dev 分支" dev
```

**推荐的分支策略：**

```
main      ──●──────────────────────●──  （只保存稳定发布版本）
              \                   /
dev            ●────●────●────●──       （日常开发主分支）
                     \       /
feature               ●────●            （每个新功能一个分支）
```

| 分支 | 用途 |
|------|------|
| `main` / `master` | 稳定版本，可直接发布 |
| `dev` | 开发主线，功能完成后合并到 main |
| `feature/xxx` | 单个新功能开发 |
| `hotfix/xxx` | 紧急 Bug 修复 |

---

### 7.4 Bug 分支（stash）

场景：正在 dev 上开发新功能（一半没完成），突然需要紧急修复 main 上的 Bug。

```bash
# 1. 先将当前未完成的工作"藏起来"
git stash

# 2. 切换到 main，从 main 创建 bug 修复分支
git switch main
git switch -c hotfix/login-bug

# 3. 修复 Bug，提交
git add .
git commit -m "修复登录 Bug"

# 4. 合并回 main，删除临时分支
git switch main
git merge --no-ff -m "合并 hotfix/login-bug" hotfix/login-bug
git branch -d hotfix/login-bug

# 5. 回到 dev，恢复之前的工作现场
git switch dev
git stash pop        # 恢复并删除 stash 记录
# 或
git stash apply      # 恢复但保留 stash 记录
```

**同步 Bug 修复到 dev**（cherry-pick）

```bash
# 将 main 上那次修复提交，"复制"一份到 dev
git cherry-pick <commit-id>
```

---

### 7.5 Feature 分支

每开发一个新功能，建议新建一个独立的 Feature 分支，防止实验性代码污染主线。

```bash
git switch -c feature/new-payment
# 开发...
git commit -m "完成新支付功能"

# 合并到 dev
git switch dev
git merge --no-ff feature/new-payment
git branch -d feature/new-payment
```

如果功能被砍掉，需要强制删除（未合并的分支用 `-d` 会报错）：

```bash
git branch -D feature/new-payment
```

---

### 7.6 多人协作

**推送分支**

```bash
git push origin main    # 推送 main 分支
git push origin dev     # 推送 dev 分支
```

**拉取远程分支**

```bash
# 方式一：fetch + merge（分两步，更安全）
git fetch origin
git merge origin/dev

# 方式二：pull（相当于 fetch + merge，一步完成）
git pull
```

**推送时发生冲突的处理流程：**

```bash
# 1. 推送失败（远程有新提交）
git push
# error: failed to push some refs...

# 2. 先拉取远程最新内容
git pull

# 3. 如果有冲突，解决冲突后再提交
git add .
git commit -m "解决冲突"

# 4. 再次推送
git push
```

**查看远程分支**

```bash
git branch -a           # 查看所有本地+远程分支
git remote show origin  # 查看远程仓库详情
```

---

### 7.7 Rebase

`rebase`（变基）可以将分支上的提交"移植"到另一条线上，让提交历史变成一条直线，更加整洁。

```bash
# 在 dev 分支上，将其变基到 main 的最新提交
git switch dev
git rebase main

# 之后再 fast-forward 合并
git switch main
git merge dev
```

**rebase vs merge 对比：**

| 操作 | 结果 | 适用场景 |
|------|------|---------|
| `merge` | 保留完整历史，有合并节点 | 公共分支合并，保留完整上下文 |
| `rebase` | 历史变成直线，更简洁 | 本地分支整理，推送前清理历史 |

> **黄金法则**：不要对已经推送到远程的公共分支执行 rebase，会导致他人的历史混乱。

---

## 8. 标签管理

标签（Tag）是某个特定提交的别名，通常用于标记版本发布点，比如 `v1.0.0`。比 commit id 更直观易记。

### 8.1 创建标签

```bash
# 给当前最新提交打标签
git tag v1.0.0

# 给指定 commit 打标签
git tag v0.9.0 a1b2c3d

# 创建带说明的标签（附注标签，推荐）
git tag -a v1.0.0 -m "Version 1.0.0 正式发布"

# 查看所有标签（按字母序排列，非时间序）
git tag

# 查看某个标签的详情
git show v1.0.0
```

---

### 8.2 操作标签

```bash
# 推送单个标签到远程
git push origin v1.0.0

# 推送所有未推送的标签
git push origin --tags

# 删除本地标签
git tag -d v1.0.0

# 删除远程标签（需要先删除本地）
git tag -d v1.0.0
git push origin :refs/tags/v1.0.0
```

---

## 9. 自定义 Git

### 9.1 忽略特殊文件 `.gitignore`

项目中有些文件不应该被 Git 追踪，比如：
- 编译产物（`*.class`、`*.pyc`、`dist/`）
- 依赖目录（`node_modules/`、`venv/`）
- 敏感配置（`.env`、含密码的配置文件）
- 系统文件（`.DS_Store`、`Thumbs.db`）

在仓库根目录创建 `.gitignore` 文件：

```gitignore
# Python
__pycache__/
*.pyc
*.pyo
.env
venv/

# Node.js
node_modules/
dist/
.env.local

# 系统文件
.DS_Store
Thumbs.db

# IDE
.vscode/
.idea/
*.swp
```

> GitHub 提供各语言的 `.gitignore` 模板：[github.com/github/gitignore](https://github.com/github/gitignore)

**强制添加被忽略的文件：**

```bash
git add -f example.log
```

**检查某文件为何被忽略：**

```bash
git check-ignore -v example.log
```

---

### 9.2 配置别名

通过别名简化常用命令：

```bash
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit

# 超实用：带颜色的图形化日志
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```

配置后使用：

```bash
git st        # 等价于 git status
git lg        # 彩色图形日志
```

配置文件保存在 `~/.gitconfig`，可直接编辑。

---

## 10. 常用命令速查

### 基础操作

| 命令 | 说明 |
|------|------|
| `git init` | 初始化仓库 |
| `git clone <url>` | 克隆远程仓库 |
| `git status` | 查看工作区状态 |
| `git add <file>` | 添加文件到暂存区 |
| `git add .` | 添加所有修改到暂存区 |
| `git commit -m "msg"` | 提交暂存区到版本库 |
| `git log --oneline` | 查看简洁提交历史 |
| `git diff` | 查看工作区修改 |

### 撤销与回退

| 命令 | 说明 |
|------|------|
| `git restore <file>` | 撤销工作区修改 |
| `git restore --staged <file>` | 撤销 add（从暂存区移回工作区） |
| `git reset --hard HEAD^` | 回退到上一个版本 |
| `git reset --hard <id>` | 回退到指定版本 |
| `git reflog` | 查看所有操作历史（包括回退） |

### 分支操作

| 命令 | 说明 |
|------|------|
| `git branch` | 查看本地分支 |
| `git switch -c <name>` | 创建并切换分支 |
| `git switch <name>` | 切换分支 |
| `git merge <name>` | 合并指定分支到当前分支 |
| `git branch -d <name>` | 删除已合并的分支 |
| `git branch -D <name>` | 强制删除分支 |
| `git stash` | 储藏当前工作现场 |
| `git stash pop` | 恢复并删除储藏 |
| `git cherry-pick <id>` | 复制某次提交到当前分支 |

### 远程操作

| 命令 | 说明 |
|------|------|
| `git remote add origin <url>` | 关联远程仓库 |
| `git push -u origin main` | 首次推送并关联分支 |
| `git push` | 推送到远程 |
| `git pull` | 拉取并合并远程更新 |
| `git fetch` | 拉取远程更新（不合并） |
| `git remote -v` | 查看远程仓库地址 |

### 标签操作

| 命令 | 说明 |
|------|------|
| `git tag v1.0.0` | 创建轻量标签 |
| `git tag -a v1.0.0 -m "msg"` | 创建附注标签 |
| `git tag` | 查看所有标签 |
| `git push origin --tags` | 推送所有标签 |
| `git tag -d v1.0.0` | 删除本地标签 |

---

## 附录：Git 工作流示意

```
远程仓库 (Remote)
      ↑ push          ↓ clone / pull / fetch
本地版本库 (Repository)
      ↑ commit        ↓ reset
暂存区 (Stage/Index)
      ↑ add           ↓ restore --staged
工作区 (Working Directory)
```

---

> **学习建议**：理解 Git 的关键在于搞清楚"工作区 → 暂存区 → 版本库"三层模型，以及每个命令在哪个层面上操作。掌握本教程的内容后，日常开发所需的 Git 操作已全部覆盖。
