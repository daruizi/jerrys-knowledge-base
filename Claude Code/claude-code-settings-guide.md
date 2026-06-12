# Claude Code 配置文件完全指南

从零开始，系统掌握 Claude Code 的配置体系。本文涵盖配置文件的层级结构、优先级规则、合并行为、安全限制，以及多模型切换的实战方案。即使你从未接触过 Claude Code 的配置系统，也能轻松读懂。

> **适用对象：** 本文面向 Claude Code 新手用户，以及希望系统化理解配置机制的进阶用户。如果你只想快速切换模型，可以直接跳到 [多模型切换方案](#多模型切换方案)。

---

## 为什么需要配置

Claude Code 的配置文件决定了以下关键行为：

- **使用哪个 AI 模型** — 如 Claude Sonnet、Qwen、GPT 等
- **连接哪个 API 端点** — Base URL 决定了请求发送到哪里
- **认证方式** — API Key 决定了你能否成功调用服务
- **权限范围** — Claude Code 可以执行哪些操作（如运行命令、读写文件）
- **扩展能力** — 通过 MCP 服务器接入外部工具

合理配置能让你在不同项目间无缝切换模型和 API，而不需要每次都手动修改。

---

## 四级配置体系

Claude Code 采用**四级配置体系**，从「全局」到「命令行」，优先级逐级递增。理解这个层级关系，是掌握整个配置系统的关键。

### 配置优先级（从高到低）

```
┌─────────────────────────────────────────────────────────┐
│ 最高优先级  │ 命令行参数 --model --apiBaseUrl           │
└────────────────────────┬────────────────────────────────┘
                         ↓ 覆盖
┌────────────────────────┴────────────────────────────────┐
│ 第 3 级    │ 项目本地设置 .claude/settings.local.json    │
└────────────────────────┬────────────────────────────────┘
                         ↓ 覆盖
┌────────────────────────┴────────────────────────────────┐
│ 第 2 级    │ 项目设置 .claude/settings.json              │
└────────────────────────┬────────────────────────────────┘
                         ↓ 覆盖
┌────────────────────────┴────────────────────────────────┐
│ 最低优先级  │ 全局设置 ~/.claude/settings.json           │
└─────────────────────────────────────────────────────────┘
```

### 1. 全局设置

| 属性 | 说明 |
|------|------|
| **文件路径** | `~/.claude/settings.json`<br>Windows: `%USERPROFILE%\.claude\settings.json` |
| **作用范围** | 对**所有项目**生效，是你的默认配置基线 |

```json
// ~/.claude/settings.json — 全局配置示例
{
  "model": "claude-sonnet-4-20250514",
  "permissions": {
    "allow": ["Read", "Glob"]
  }
}
```

> **提示：** 全局配置适合设置你希望在所有项目中通用的选项，比如默认模型、常用权限等。

### 2. 项目设置

| 属性 | 说明 |
|------|------|
| **文件路径** | `<项目根目录>/.claude/settings.json` |
| **可提交到 Git** | 这个文件**可以**提交到版本控制，让整个团队共享同一套配置 |

```json
// .claude/settings.json — 项目级配置示例
{
  "model": "claude-sonnet-4-20250514",
  "permissions": {
    "allow": ["Bash(npm:*)", "Bash(git:*)"]
  }
}
```

### 3. 项目本地设置

| 属性 | 说明 |
|------|------|
| **文件路径** | `<项目根目录>/.claude/settings.local.json` |
| **不要提交到 Git** | 包含个人敏感信息（如 API Key），应加入 `.gitignore` |

```json
// .claude/settings.local.json — 本地个人配置
{
  "model": "qwen-max",
  "apiBaseUrl": "https://dashscope.aliyuncs.com/compatible-mode/v1",
  "env": {
    "ANTHROPIC_API_KEY": "sk-your-key-here"
  }
}
```

> **注意：** `settings.local.json` 的优先级高于 `settings.json`。即使项目 `settings.json` 提交到了 Git，你本地的个人配置也会覆盖它，不会被推送。

### 4. 命令行参数

通过启动 Claude Code 时传入的参数，拥有**最高优先级**，可以临时覆盖任何 JSON 配置：

```bash
# 临时使用指定模型启动
claude --model claude-sonnet-4-20250514

# 临时指定 API 端点
claude --apiBaseUrl https://api.openai.com/v1
```

> **提示：** 命令行参数适合偶尔需要临时切换模型的场景，不需要修改任何配置文件。

---

## 优先级规则

当多个配置文件对同一个字段设置了不同的值时，**高优先级配置覆盖低优先级**。

### 场景举例

假设三个文件分别配置了不同的模型：

| 配置文件 | 模型 (model) | API 端点 (apiBaseUrl) |
|----------|--------------|----------------------|
| `~/.claude/settings.json` (全局) | `gpt-4` | `https://api.openai.com` |
| `.claude/settings.json` (项目) | `claude-sonnet-4-20250514` | `https://api.anthropic.com` |
| `.claude/settings.local.json` (本地) | `qwen-max` | `https://dashscope.aliyuncs.com` |

> **最终结果：** 使用的是 `settings.local.json` 中的配置 — **阿里云 DashScope + qwen-max**。因为 `model` 是标量值，高优先级直接覆盖，不会合并。

---

## 合并行为

并非所有配置都是「覆盖」。Claude Code 根据字段类型采用不同的合并策略：

| 合并类型 | 说明 |
|----------|------|
| **数组字段 — 追加** | 如 `permissions.allow`，各层级的数组元素会**合并到一起**，而非替换 |
| **对象字段 — 深度合并** | 如 `env`、`hooks`，子属性按 key 递归合并，相同 key 高优先级覆盖 |
| **标量字段 — 覆盖** | 如 `model`、`apiBaseUrl`，高优先级**直接替换**低优先级的值 |

### 数组追加示例

```json
// 全局 settings.json
"permissions": { "allow": ["Bash(ls:*)"] }

// 项目 settings.json
"permissions": { "allow": ["Bash(git:*)"] }

// 最终生效的结果：
"permissions": { "allow": ["Bash(ls:*)", "Bash(git:*)"] }
//                   ↑ 两个权限被合并在一起
```

### 对象深度合并示例

```json
// 全局
"env": { "API_KEY": "key1", "DEBUG": "true" }

// 项目本地
"env": { "API_KEY": "key2", "LOG_LEVEL": "info" }

// 最终结果：
"env": {
  "API_KEY": "key2",     // ← 被 local 覆盖
  "DEBUG": "true",       // ← 从全局保留
  "LOG_LEVEL": "info"    // ← 从本地新增
}
```

### 标量覆盖示例

```json
// 全局
"model": "gpt-4"

// 项目本地
"model": "qwen-max"

// 最终结果：model = "qwen-max"（本地直接覆盖全局）
```

---

## 安全限制

出于安全考虑，部分配置项**不允许**在项目级 `settings.json` 中设置：

| 配置项 | 全局 | 项目 | 本地 | 说明 |
|--------|------|------|------|------|
| `env` | ✓ | ✗ | ✓ | 防止恶意项目通过 `settings.json` 窃取环境变量 |
| `model` | ✓ | ✓ | ✓ | 所有层级均可设置 |
| `permissions` | ✓ | ✓ | ✓ | 所有层级均可设置 |
| `mcpServers` | ✓ | ✓ | ✓ | 所有层级均可设置 |
| `hooks` | ✓ | ✓ | ✓ | 所有层级均可设置 |

> **安全须知：** `env` 字段（包含 API Key 等敏感信息）只能在**全局** `settings.json` 或项目级 `settings.local.json` 中配置，不能写在可提交到 Git 的 `settings.json` 中。

---

## 配置字段详解

### 模型配置

模型配置决定了 Claude Code 使用哪个 AI 模型和哪个 API 端点。

| 字段 | 类型 | 说明 | 示例 |
|------|------|------|------|
| `model` | string | 要使用的模型名称 | `"claude-sonnet-4-20250514"` |
| `apiBaseUrl` | string | API 端点地址 | `"https://dashscope.aliyuncs.com/compatible-mode/v1"` |

```json
{
  "model": "qwen-max",
  "apiBaseUrl": "https://dashscope.aliyuncs.com/compatible-mode/v1"
}
```

### 权限配置

控制 Claude Code 可以执行哪些操作，防止误操作带来的风险。

| 字段 | 类型 | 说明 |
|------|------|------|
| `permissions.allow` | string[] | 允许的操作列表（白名单） |
| `permissions.deny` | string[] | 禁止的操作列表（黑名单） |

```json
{
  "permissions": {
    "allow": [
      "Read",
      "Glob",
      "Bash(git:*)",
      "Bash(npm:*)"
    ],
    "deny": [
      "Bash(rm -rf:*)"
    ]
  }
}
```

### 环境变量

设置 Claude Code 运行时可用的环境变量，常用于配置 API Key。

```json
{
  "env": {
    "ANTHROPIC_API_KEY": "sk-ant-xxx",
    "OPENAI_API_KEY": "sk-xxx"
  }
}
```

> **再次提醒：** `env` 不能写在项目级 `settings.json` 中（安全限制），只能写在全局 `settings.json` 或 `settings.local.json` 中。

### MCP 服务器

MCP (Model Context Protocol) 让 Claude Code 接入外部工具和数据源。

```json
{
  "mcpServers": {
    "my-database": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres", "postgresql://localhost/mydb"]
    },
    "my-filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/dir"]
    }
  }
}
```

### Hooks

Hooks 允许你在特定事件发生时自动执行脚本，比如提交代码前自动格式化。

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit",
        "command": "echo 'About to edit a file...'"
      }
    ]
  }
}
```

---

## 多模型切换方案

如果你需要在不同模型之间切换（比如阿里云 Qwen、OpenAI GPT、Anthropic Claude），以下是三种实用方案。

### 方案一：按项目目录隔离

最直观的方法 — 不同项目目录放不同的配置，天然隔离。

| 目录 | 配置内容 | 模型 |
|------|----------|------|
| `C:\Projects\dashscope\.claude\settings.local.json` | 阿里云 DashScope | qwen-max |
| `C:\Projects\openai\.claude\settings.local.json` | OpenAI | gpt-4o |
| `C:\Projects\anthropic\.claude\settings.local.json` | Anthropic | claude-sonnet-4-20250514 |

```json
// C:\Projects\dashscope\.claude\settings.local.json
{
  "model": "qwen-max",
  "apiBaseUrl": "https://dashscope.aliyuncs.com/compatible-mode/v1",
  "env": {
    "ANTHROPIC_API_KEY": "sk-dashscope-xxx"
  }
}
```

> **优点：** 零切换成本。进入哪个目录，就自动使用哪个配置。
> **缺点：** 需要为每个模型维护一个项目目录，不够灵活。

### 方案二：快捷启动脚本

在用户目录下创建多个启动脚本，每个脚本配置不同的模型和 API Key。

#### claude-qwen.bat

```batch
@echo off
set ANTHROPIC_API_KEY=sk-dashscope-xxx
claude --model qwen-max --apiBaseUrl https://dashscope.aliyuncs.com/compatible-mode/v1
```

#### claude-gpt.bat

```batch
@echo off
set ANTHROPIC_API_KEY=sk-openai-xxx
claude --model gpt-4o --apiBaseUrl https://api.openai.com/v1
```

#### claude-sonnet.bat

```batch
@echo off
set ANTHROPIC_API_KEY=sk-ant-xxx
claude --model claude-sonnet-4-20250514
```

> **优点：** 双击即可启动，适合固定使用几个模型的场景。
> **缺点：** 新增模型需要创建新脚本文件。

### 方案三：参数化统一脚本（推荐）

一个脚本管所有模型，通过参数选择。新增模型只需加一个分支。

#### claude-switch.bat

```batch
@echo off
set profile=%1

if "%profile%"=="qwen" (
    set ANTHROPIC_API_KEY=sk-dashscope-xxx
    claude --model qwen-max --apiBaseUrl https://dashscope.aliyuncs.com/compatible-mode/v1
) else if "%profile%"=="gpt" (
    set ANTHROPIC_API_KEY=sk-openai-xxx
    claude --model gpt-4o --apiBaseUrl https://api.openai.com/v1
) else if "%profile%"=="sonnet" (
    set ANTHROPIC_API_KEY=sk-ant-xxx
    claude --model claude-sonnet-4-20250514
) else (
    echo Usage: claude-switch [qwen^|gpt^|sonnet]
)
```

使用方式：

```bash
claude-switch qwen      # 使用阿里云 qwen-max
claude-switch gpt       # 使用 OpenAI gpt-4o
claude-switch sonnet    # 使用 Anthropic Claude Sonnet
```

> **推荐理由：** 一个入口管理所有模型配置，新增模型只需加一个 `else if` 分支。不动全局配置文件，灵活且易维护。

### 方案对比总结

| 方案 | 适合场景 | 切换方式 | 新增模型 | 维护成本 |
|------|----------|----------|----------|----------|
| **目录隔离** | 项目与模型绑定 | `cd` 到目录 | 新建目录+配置 | 低 |
| **多个脚本** | 固定几个模型 | 运行不同脚本 | 新建脚本 | 中 |
| **参数化脚本** ★ | 灵活切换 | `claude-switch <name>` | 加一个分支 | 低 |

---

## 常见问题

### Q: 我在项目目录下修改了 `settings.json`，但配置没有生效？

检查是否存在 `settings.local.json`，它的优先级更高，会覆盖 `settings.json` 的同名字段。

### Q: 全局配置的 API Key 和项目配置的不同，哪个会用？

`env` 字段按 key 深度合并。如果全局和本地都设置了 `ANTHROPIC_API_KEY`，本地（`settings.local.json`）的值会覆盖全局的。

### Q: `settings.local.json` 需要手动创建吗？

是的，如果不存在，手动创建即可。Claude Code 不会自动生成这个文件。

### Q: 如何确认当前生效的配置？

在 Claude Code 中使用 `/config` 命令可以查看当前配置状态。

### Q: 多个项目的 `permissions.allow` 会互相影响吗？

不会。项目级权限只在对应项目目录下生效。全局权限则会叠加到所有项目中。

### Q: 使用第三方 API（如阿里云 DashScope）时，需要注意什么？

1. 确保 `apiBaseUrl` 指向正确的兼容模式端点
2. 将对应的 API Key 通过 `env` 字段或环境变量设置
3. 确保 `model` 名称与该平台支持的模型名一致

---

## 速查表

### 配置文件层级

| 文件路径 | 作用范围 | 可提交 Git | 优先级 |
|----------|----------|------------|--------|
| `~/.claude/settings.json` | 所有项目 | N/A（个人目录） | 最低 |
| `<项目>/.claude/settings.json` | 当前项目 | ✓ 可以 | 中 |
| `<项目>/.claude/settings.local.json` | 当前项目（个人） | ✗ 不可以 | 高 |
| CLI 参数 `--model` 等 | 当次启动 | N/A | 最高 |

### 字段合并行为

| 字段类型 | 合并行为 | 示例 |
|----------|----------|------|
| 数组 `string[]` | **追加合并** | `permissions.allow` |
| 对象 `object` | **深度合并（同 key 覆盖）** | `env`、`hooks` |
| 标量 `string` | **高优先级直接覆盖** | `model`、`apiBaseUrl` |

---

*Claude Code 配置文件完全指南 · 2026.06 · 内容基于 Claude Code 官方文档整理*
