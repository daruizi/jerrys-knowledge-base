# Claude Code 命令与快捷键速查手册

本文档基于官方 [Claude Code 文档](https://docs.anthropic.com/en/docs/claude-code/overview) 整理，提供详尽的命令参考、快捷键指南以及配置说明。

---

## 🏃‍♂️ CLI 命令行参考

可以在终端中直接通过 `claude` 命令及参数启动助手：

| CLI 标志/参数 | 示例格式 | 中文说明 |
|---------------|-----------|----------|
| `claude <query>` | `claude "explain this project"` | 直接执行单次请求并退出 |
| `-p` / `--print` | `claude -p "Check for type errors"` | 执行单次请求，输出后立即退出 |
| `< file \| claude -p` | `cat logs.txt \| claude -p "explain"` | 通过管道传递内容给 Claude |
| `-c` / `--continue` | `claude -c -p "Check errors"` | 加载最后一次会话并追加请求 |
| `-r` / `--resume` | `claude -r "session-name"` | 恢复到指定的历史会话 |
| `-n` / `--name` | `claude -n "my-feature-work"` | 为当前启动的会话命名 |
| `--model` | `claude --model claude-sonnet-4-6` | 覆盖默认模型 (如 opus, sonnet) |
| `--system-prompt` | `claude --system-prompt "Use TS"`| 用指定的系统提示词覆盖默认提示 |
| `--append-system-prompt`| `claude --append-system-prompt "..."`| 在系统提示词末尾追加规则 |
| `--tools` | `claude --tools "Bash,Edit,Read"`| 逗号分隔的允许 Claude 调用的工具列表，为空代表无限制 |
| `--permission-mode` | `claude --permission-mode plan`  | 指定权限模型（默认启用或仅计划） |
| `--dangerously-skip-permissions` | `claude --dangerously-skip-permissions` | 跳过所有权限确认（自动执行高风险操作） |
| `-w` / `--worktree` | `claude -w feature-auth` | 在指定的 Git 工作树下运行 |
| `--mcp-config` | `claude --mcp-config ./mcp.json` | 强制使用指定路径的 MCP 配置文件 |
| `--remote` | `claude --remote "fix bug"` | 从终端发起一个 Web 会话执行请求 |
| `--remote-control` / `--rc`| `claude --rc "Project"` | 通过远程控制模式启动 Claude Code |
| `claude update` | `claude update` | 更新 Claude Code 本身 |
| `claude auth ...` | `claude auth login`/`logout`/`status`| 管理 Anthropic 账号认证状态 |
| `claude mcp` | `claude mcp` | 管理与配置本地 MCP 服务器 |

---

## ⚡ 内置斜杠命令 (Slash Commands)

进入 Claude Code 交互模式后，可以通过以 `/` 开头的命令控制会话态势。

### 核心管理与导航
| 命令 | 别名 | 中文说明 |
|------|------|----------|
| `/help` 或 `/?` | - | 显示帮助信息 |
| `/clear` | `/reset`, `/new` | 清除上下文对话历史，开始全新对话 |
| `/compact` | - | 将历史指令和上下文压缩，以节省 Tokens/上下文空间 |
| `/history`或`↑`| - | 查看/浏览历史命令记录 |
| `/exit` | `/quit`, `Ctrl+D`| 结束当前对话与应用进程 |
| `/status` | - | 显示当前工作状态、启用的模型及上下文使用情况 |
| `/cost` | - | 显示当前会话的开销统计 |
| `/usage` | - | 展现系统资源与使用量 |

### 文件与项目协同
| 命令 | 中文说明 |
|------|----------|
| `/add-dir <path>` | 向上下文中显式添加某个文件夹 |
| `/review` | 请求 Claude 对当前改动或指定代码进行审查诊断 |
| `/pr-comments [PR]` | 快速拉取并分析 GitHub 的 PR 评论信息 |
| `/branch [name]` | 检出或创建指定的 Git 分支 |
| `/init` | 在当前目录下初始化 `CLAUDE.md` 项目指令文件 |

### 执行与任务
| 命令 | 中文说明 |
|------|----------|
| `/simplify` | **核心技能**：快速简化和重构代码（关注内存与代码复用效率） |
| `/batch <instruction>`| 批量执行操作，配合工作树 (`/worktree`) 处理海量文件重构 |
| `/debug [问题描述]` | **核心技能**：快速进入调试模式寻找 Bug |
| `/tasks` | 整理、创建和规划当前会话的任务结构 |
| `/loop [时间] <提示词>`| 定时执行指令，如 `/loop 5m check status` (默认间隔 10 分钟) |
| `/claude-api` | 帮助构建依赖 Claude API 或 SDK 的应用代码 |
| `/mcp__<server>__<prompt>`| 触发特定 MCP 工具，例如 `/mcp__sqlite__schema` |
| `/skill-name` | 执行定义在 `.claude/skills/` 下的自定义拓展技能 |

### 模型与配置控制
| 命令 | 中文说明 |
|------|----------|
| `/model [model]` | 切换当前交流的底层模型（如 `claude-opus-4-6` 或 `claude-sonnet-4-6`） |
| `/effort [级别]` | 动态设定模型努力级别：`low` / `medium` / `high` / `max` / `auto` |
| `/fast [on\|off]` | 切换至极速但相对简略推理的极速模式 |
| `/permissions` | 调整命令交互模式和权限（允许/拒绝特定的工具执行） |
| `/config` / `/settings`| 直接调出配置文件设定窗口 |
| `/theme` / `/color [色值]`| 设置交互界面的主题与颜色（如 `red`, `blue`, `default` 等） |
| `/terminal-setup`| 设定操作系统的快捷键支持体验 (例如将 Option 键设为 Meta 键) |
| `/desktop`/`/app`| 将目前的终端环境迁移传递给 Desktop App 会话 |
| `/privacy-settings`| 隐私保护与数据收集详细管理 |

---

## ⌨️ 快捷键 (Interactive Mode Shortcuts)

Claude Code 提供了一套极其强悍的类似 Emacs/Vim 的快捷键模式（部分按键需前置终端配置支持，具体可通过 `/terminal-setup` 指引并借助终端设置，将 `Alt/Option` 作为 `Meta` 键）。

### 通用控制
| 快捷键 | 功能 | 快捷键 | 功能 |
|--------|------|--------|------|
| `Ctrl+C` | 中断当前 AI 响应 / 终止正在运行的命令 / 清空输入行 | `Ctrl+L` | 清屏，保留上下文历史 |
| `Ctrl+D` | 退出 Claude Code（空行时） | `Ctrl+R` | 在输入历史中进行反向增量搜索 (Reverse Search) |
| `Tab` | 自动补全代码路径或命令 / 在侯选项中顺向切换 | `Shift+Tab`| 在侯选列表项中逆向切换 |
| `↑` / `↓` | 在历史命令记录中上下追回游标 | `Enter` | 提交请求 |
| `Esc` | 重置/取消候选项补缀状态或中止特殊会话窗（也可以用退格回滚） | `Ctrl+T` | 打开任务列表状态浮层查看 |

### 行内编辑快捷键
| 快捷键 | 功能 | 快捷键 | 功能 |
|--------|------|--------|------|
| `Ctrl+A` | 移动光标到行首 | `Ctrl+E` | 移动光标到行尾 |
| `Ctrl+F` | 光标前移(前进)一个字符 (Forward) | `Ctrl+B` | 光标后移(后退)一个字符 (Backward) |
| `Alt+F` | 光标前移一个完整词 (Forward Word) | `Alt+B` | 光标后移一个完整词 (Backward Word) |
| `Ctrl+U` | **删除光标前整行的文本内容** | `Ctrl+K` | **删除光标所在位置直至行尾的内容** |
| `Ctrl+W` | 剪切/删除光标前的一个词 | `Ctrl+Y` | 粘贴最近一次被 `Ctrl+K/U/W` 剪切的文本 |
| `Alt+D` | 剪切向后的一个大片段 | `Alt+Y` | 粘贴被 `Alt` 被组合键剪切的长文本区块 |

### 针对长文本的多行输入 (Multiline Input)
长段落无需直接回车触发模型响应，可通过以下方式换行：
- 发起输入时在末尾加上 `\` 再跟 `Enter` 以换行不提交
- 直接用 `Shift+Enter` (或 `Option+Enter`, `Ctrl+J`) 做软回车换行。

### 插入特殊标志
- 输入 `!` 可直接启动一个内置命令块请求系统运行 bash (如 `!git status`)
- 输入 `@` 可自动唤起工作区文件补全快速引用
- 对话中间可穿插 `/btw <question>` 抛出旁支任务与旁支会话，不会污染主干历史脉络。

### 语音识别支持 (Voice Input)
若当前桌面终端平台支持麦克风捕获，可绑定**按住拾音快捷键**（默认某些绑定为空格键）。该配置可在 `/voice` 定义。

---

## 🛠️ 工具权限列表 (Tools Available to Claude)

在会话期间，Claude 实际上会调度下方的自动化工具替您完成对应任务；你可以在 `--allowedTools` 参数里设定限定它准许调用的范围集合：

1. **核心编辑工具**：
   - `Read`: 查阅及读取特定文件/源代码区间内容。
   - `Write`: 安全生成并覆盖文件，或创建特定文件。
   - `Edit`: 高度精确的代码单点与块状增量替换修改。
2. **搜索体系工具**：
   - `Glob`: 遍历项目下符合特殊语法的树状文件目录。
   - `Grep`: 全局执行代码文本与高阶正则信息挖掘。
   - `ToolSearch`: 通过 MCP 扩展在大规模生态内寻求外部服务和资源组件接口。
3. **安全运行时与控制工具**：
   - `Bash`: 在项目环境中执行合规 shell 命令（需要手动授权确认）。
   - `TaskCreate` / `TaskList` / `TaskUpdate`: 动态规划任务、查看所有未读与记录并打勾确认，方便 Agent 回滚或多步工作进程串联。
   - `AskUserQuestion`: 遇到不确定设计细节主动中止请求向人类提问的功能节点。
   - `EnterPlanMode` / `ExitPlanMode`: 进入方案推理设计和出沙盒请求批准的指令转换工具。
4. **外部支持和扩展**：
   - `WebFetch` / `WebSearch`: 读取并抽取互联网资料信息，直接整合到请求回答流当中。
   - `ReadMcpResourceTool` / `ListMcpResourcesTool`: 与基于 Model Context Protocol 创建本地后端和私域系统的微服务互动资源交换通道通道。
   - `Agent`: 孵化（Fork）衍生“子代理 (Subagent)” 专门负责特定封闭或耗时子任务。
   - `NotebookEdit`: 可独立修改操控 Jupyter Notebook `.ipynb` 可视化块资源。

---

## ⚙️ 设置、上下文指引与进阶配置位置

### 项目级永久记忆 (`CLAUDE.md`)
使用 `/init` 生成项目级上下文守则文件 `CLAUDE.md`。这将被 Claude 视作 **不可妥协的铁律**，任何属于本代码库独有的构建规范（例如："必须使用 TS 不用 JS"，"使用 pnpm 替代 npm"，"样式首选 Tailwind"）都会在它运行初期首当拉取读取，实现永久记忆的统一和持久化上下文绑定机制。它会通过 `Auto Memory`（自动记忆累加机制）实现知识更新，可通过 `/memory` 即时调整内容。

### 核心配置文件与环境变量
1. **全局基本设置层**：配置文件存放在你的用户主目录下的 `~/.claude/settings.json`（设定高维操作，例如默认模型等参数）。
2. **项目级设置层**：
   - 项目级别默认在工作区建立 `.claude/settings.json` 用于协作。
   - 个人偏好建立在 `.claude/settings.local.json`，不会通过 Git 污染队友。
3. **环境变量**：可以通过加载 `.env` 为工具提供注入密钥令牌和 MCP 所需的环境配置，从而不泄露到外网。
4. **管理与策略控制文件 (Managed settings)**：适用于企业，在 `macOS: /Library/Application Support/ClaudeCode/` 或 `Windows: C:\Program Files\ClaudeCode\`。

*有关进阶基于 `SKILL.md` (Skills) 和协议构建的内容，请另行参考本仓高阶开发文档。*
