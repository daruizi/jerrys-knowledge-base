# Claude Code 命令参考文档

本文档提供 Claude Code 所有命令的完整参考，包含斜杠命令、内置技能、快捷键等。

---

## 斜杠命令 (Slash Commands)

| 命令 | 中文释义 | 示例 |
|------|----------|------|
| `/help` | 获取帮助信息 | `/help` |
| `/clear` | 清除对话历史 | `/clear` |
| `/compact` | 压缩对话以节省上下文空间 | `/compact` |
| `/cost` | 显示当前会话的费用统计 | `/cost` |
| `/doctor` | 运行诊断检查 | `/doctor` |
| `/init` | 初始化项目配置 | `/init` |
| `/logout` | 登出当前账户 | `/logout` |
| `/memory` | 管理持久化记忆 | `/memory` |
| `/model` | 切换 AI 模型 | `/model claude-sonnet-4-6` |
| `/permissions` | 管理权限设置 | `/permissions` |
| `/pr-comments` | 查看 PR 评论 | `/pr-comments` |
| `/review` | 代码审查模式 | `/review` |
| `/status` | 显示当前状态 | `/status` |
| `/terminal-setup` | 设置终端集成 | `/terminal-setup` |
| `/vim` | 切换 Vim 模式 | `/vim` |
| `/config` | 配置管理 | `/config` |
| `/mcp` | MCP 服务器管理 | `/mcp` |
| `/bug` | 报告 Bug | `/bug` |

---

## 内置技能 (Built-in Skills)

| 技能名称 | 中文释义 | 示例 |
|----------|----------|------|
| `commit` | 创建 Git 提交 | `/commit` 或 `/commit -m "message"` |
| `review-pr` | 审查 Pull Request | `/review-pr 123` |
| `pr` | 创建 Pull Request | `/pr` |
| `issue` | 创建或查看 Issue | `/issue 456` |
| `release` | 创建发布 | `/release` |
| `pdf` | 处理 PDF 文件 | `/pdf document.pdf` |
| `loop` | 循环执行命令（定时重复执行提示词或斜杠命令） | `/loop 5m /foo`（每5分钟执行一次，默认10分钟） |
| `simplify` | 简化代码（审查代码的重用性、质量和效率，并修复发现的问题） | `/simplify` 或 "simplify the code" |
| `update-config` | 更新配置（配置 Claude Code 设置、权限、环境变量、钩子等） | `/update-config` 或 "allow npm commands" |
| `claude-api` | Claude API 开发助手（帮助构建使用 Claude API 或 Anthropic SDK 的应用） | 当代码导入 `anthropic` 或 `@anthropic-ai/sdk` 时自动触发 |

---

## 快捷键 (Keyboard Shortcuts)

| 快捷键 | 中文释义 | 说明 |
|--------|----------|------|
| `Ctrl+C` | 中断当前操作 | 取消正在进行的工具调用 |
| `Ctrl+D` | 退出 Claude Code | 结束当前会话 |
| `Ctrl+L` | 清屏 | 清除终端显示内容 |
| `↑` / `↓` | 浏览历史命令 | 在输入历史中导航 |
| `Tab` | 自动补全 | 补全命令或文件路径 |
| `Shift+Tab` | 反向补全 | 反向选择补全选项 |
| `Ctrl+R` | 搜索历史 | 搜索命令历史 |
| `Ctrl+U` | 删除整行 | 删除当前输入行的所有内容 |
| `Ctrl+W` | 删除单词 | 删除光标前的一个单词 |
| `Ctrl+A` | 移到行首 | 将光标移到行首 |
| `Ctrl+E` | 移到行尾 | 将光标移到行尾 |
| `Esc` | 取消当前输入 | 清空当前输入或退出模式 |
| `?` | 显示帮助 | 在某些上下文中显示帮助 |

---

## MCP 相关命令

| 命令 | 中文释义 | 示例 |
|------|----------|------|
| `/mcp` | 管理 MCP 服务器 | `/mcp` |
| `/mcp add` | 添加 MCP 服务器 | `/mcp add server-name` |
| `/mcp remove` | 移除 MCP 服务器 | `/mcp remove server-name` |
| `/mcp list` | 列出所有 MCP 服务器 | `/mcp list` |
| `/mcp restart` | 重启 MCP 服务器 | `/mcp restart server-name` |

---

## 权限管理命令

| 命令 | 中文释义 | 示例 |
|------|----------|------|
| `/permissions` | 打开权限管理 | `/permissions` |
| `/permissions allow` | 允许特定操作 | `/permissions allow Bash(git status)` |
| `/permissions deny` | 拒绝特定操作 | `/permissions deny Write(*.env)` |
| `/permissions reset` | 重置权限设置 | `/permissions reset` |

---

## 模型相关命令

| 命令 | 中文释义 | 示例 |
|------|----------|------|
| `/model` | 查看当前模型 | `/model` |
| `/model <name>` | 切换模型 | `/model claude-opus-4-6` |
| `/fast` | 切换快速模式 | `/fast` |

### 可用模型

| 模型 ID | 中文名称 | 说明 |
|---------|----------|------|
| `claude-opus-4-6` | Claude Opus 4.6 | 最强大的模型，适合复杂任务 |
| `claude-sonnet-4-6` | Claude Sonnet 4.6 | 平衡性能与速度 |
| `claude-haiku-4-5-20251001` | Claude Haiku 4.5 | 快速响应，适合简单任务 |

---

## 工具调用说明

Claude Code 可以调用以下工具：

| 工具 | 中文释义 | 用途 |
|------|----------|------|
| `Read` | 读取文件 | 读取文件内容 |
| `Write` | 写入文件 | 创建或覆盖文件 |
| `Edit` | 编辑文件 | 对文件进行精确编辑 |
| `Glob` | 文件搜索 | 按模式搜索文件 |
| `Grep` | 内容搜索 | 在文件中搜索内容 |
| `Bash` | 执行命令 | 运行 shell 命令 |
| `WebFetch` | 获取网页 | 获取网页内容 |
| `WebSearch` | 网页搜索 | 搜索互联网信息 |
| `Agent` | 子代理 | 启动专门代理处理任务 |
| `TaskCreate` | 创建任务 | 创建结构化任务列表 |
| `TaskList` | 列出任务 | 查看所有任务 |
| `TaskUpdate` | 更新任务 | 更新任务状态 |
| `AskUserQuestion` | 提问用户 | 向用户提问获取输入 |
| `EnterPlanMode` | 进入计划模式 | 规划实现方案 |
| `ExitPlanMode` | 退出计划模式 | 完成规划并请求批准 |
| `EnterWorktree` | 进入工作树 | 创建隔离的 Git 工作树 |
| `ExitWorktree` | 退出工作树 | 离开工作树环境 |
| `NotebookEdit` | 编辑笔记本 | 编辑 Jupyter Notebook |
| `CronCreate` | 创建定时任务 | 调度定时执行的任务 |
| `CronDelete` | 删除定时任务 | 取消已调度的任务 |
| `CronList` | 列出定时任务 | 查看所有定时任务 |
| `Skill` | 执行技能 | 调用预定义的技能 |

---

## Git 相关操作

| 操作 | 中文释义 | 命令示例 |
|------|----------|----------|
| 创建提交 | 创建新的 Git 提交 | `/commit` 或让 Claude 执行 `git commit` |
| 创建 PR | 创建 Pull Request | `/pr` 或 `/review-pr <number>` |
| 查看状态 | 查看 Git 仓库状态 | 让 Claude 执行 `git status` |
| 查看日志 | 查看提交历史 | 让 Claude 执行 `git log` |
| 切换分支 | 切换到其他分支 | 让 Claude 执行 `git checkout <branch>` |
| 合并分支 | 合并分支 | 让 Claude 执行 `git merge <branch>` |

---

## 特殊命令格式

### 文件引用
```
@filename    # 引用特定文件
@folder/     # 引用文件夹
```

### 多行输入
```
"""多行文本
可以在这里输入
多行内容
"""
```

### JSON 块
```json
{"key": "value"}
```

---

## 配置文件位置

| 文件 | 位置 | 用途 |
|------|------|------|
| `CLAUDE.md` | 项目根目录 | 项目级指令 |
| `settings.json` | `~/.claude/` | 全局设置 |
| `memory/` | `~/.claude/projects/<project>/memory/` | 持久化记忆 |

---

## 常见使用场景

### 代码审查
```
/review
或
"请审查我的代码变更"
```

### 创建提交
```
/commit
或
"请帮我创建一个提交"
```

### 调试问题
```
"帮我调试这个错误: [错误信息]"
```

### 解释代码
```
"请解释这个函数的作用"
```

### 重构代码
```
"请重构这个模块，使其更易读"
```

---

## 提示

1. **使用 `/help`** - 随时获取帮助
2. **使用 Tab 补全** - 快速输入命令
3. **使用 `↑` 键** - 重复之前的命令
4. **使用 `@` 引用** - 快速引用文件
5. **使用 `/compact`** - 当上下文过长时压缩对话