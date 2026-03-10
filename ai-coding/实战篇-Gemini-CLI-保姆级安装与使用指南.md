# 实战篇 - Gemini CLI 保姆级安装与使用指南

## 1. 简介
Gemini CLI 是 Google 推出的原生 AI 辅助开发工具，集成了强大的 Gemini 2.0 系列模型。它不仅是一个聊天机器人，更是一个拥有完整终端权限的“虚拟工程师”，能够深度理解项目架构、自动编写代码、运行测试并修复 Bug。

---

## 2. 安装与环境配置

Gemini CLI 依赖 Node.js 环境，支持 Windows、macOS 和 Linux。

### 2.1 安装步骤
打开终端（PowerShell 或 Bash），运行：
```bash
# 全局安装 Gemini CLI
npm install -g @google/gemini-cli
```

### 2.2 身份认证 (Authentication)
安装完成后，需要绑定 Google 账号以获得 API 访问权限：
*   **登录：** `gemini auth login` (将自动打开浏览器进行 OAuth 授权)。
*   **退出：** `gemini auth logout`。
*   **查看状态：** `gemini auth status` (检查 Token 有效期及当前登录账号)。

---

## 3. 交互式核心命令 (Slash Commands)

在 Gemini CLI 的对话框中，以下斜杠命令用于管理会话，不消耗 Token。

| 命令 | 功能描述 | 最佳实践 |
| :--- | :--- | :--- |
| **`/help`** | **显示帮助** | 忘记指令时随时调用。 |
| **`/clear`** | **重置记忆** | **高频：** 开启新任务前必须执行，避免上下文污染。 |
| **`/exit`** | **退出程序** | 安全关闭 CLI 界面。 |
| **`/bug`** | **反馈问题** | 遇到工具运行异常时向 Google 提交报告。 |
| **`/settings`** | **查看配置** | 确认当前使用的模型及工作目录。 |

---

## 4. 账户管理与 Token 使用监控

### 4.1 Token 消耗机制
Gemini CLI 的运行基于 Token 计费。
*   **输入：** 包含历史对话、当前文件内容、`GEMINI.md` 规范。
*   **输出：** AI 生成的代码、分析报告。

### 4.2 实时监控与额度查看
*   **监控方式：** 在每次大型任务结束后，CLI 会输出估算的 Token 消耗。
*   **后台查看：** 登录 [Google AI Studio](https://aistudio.google.com/)，在 "Settings > Plan & Billing" 中查看实时配额。
*   **节省技巧：** 切换话题务必使用 `/clear`；利用 `.geminiignore` 忽略大型依赖库（如 `node_modules`）。

---

## 5. 启动命令行参数 (Startup Flags)

在系统 Shell 中启动 `gemini` 时可携带参数进行初始化：

| 参数 | 描述 | 示例 |
| :--- | :--- | :--- |
| **`--dir <path>`** | 指定 AI 的工作目录（默认为当前目录）。 | `gemini --dir ./my-app` |
| **`--model <id>`** | 强制指定使用的 Gemini 模型版本。 | `gemini --model gemini-2.0-pro` |
| **`--verbose`** | 显示所有后台运行日志（调试用）。 | `gemini --verbose` |
| **`--no-history`** | 启动时不加载上次对话的历史上下文。 | `gemini --no-history` |

---

## 6. 专家子代理 (Expert Sub-agents)

Gemini CLI 内部集成了多个“专家”，系统会自动根据您的需求调用：

1.  **Codebase Investigator (代码库侦探)**
    *   **适用：** 梳理复杂代码逻辑、跨文件追溯变量引用。
    *   **触发词：** “分析登录流程”、“解释这个项目的目录结构”。
2.  **Generalist (全才代理)**
    *   **适用：** 批量处理重复性任务（重构、注释、清理）。
    *   **触发词：** “给全项目加 License 头”、“修复本项目所有的 Lint 错误”。
3.  **CLI Help (官方文档专家)**
    *   **适用：** 咨询工具本身的使用技巧或配置。
    *   **触发词：** “Gemini CLI 有哪些新功能？”、“如何配置代理服务器？”。

---

## 7. 核心进阶：`GEMINI.md` 规范文件

这是控制 AI 行为的“最高宪法”。在项目根目录创建该文件，写下您的编码偏好。

**内容示例：**
```markdown
# 我们的开发规范
- 所有新功能必须使用 TypeScript 编写。
- 强制使用 2 个空格作为缩进，不要使用 Tab。
- 每次修改代码后必须运行 `npm test` 并确认通过。
- 禁止修改 `.config/` 目录下的敏感文件。
```
**AI 每次任务前都会读取此文件，确保产出的代码 100% 符合您的预期。**

---

## 8. 新手实战建议

*   **先问后动：** 刚开始可以先问“如何实现...”，让 AI 提供方案；满意后再要求“请执行刚才的方案”。
*   **安全确认：** 当 AI 提示执行 `rm` 或 `git push` 等敏感命令时，务必审阅弹出的命令详情。
*   **闭环验证：** 下达指令时加上：“修复它，并运行相关测试确认已修复”。

祝您在 AI 辅助开发的旅程中如虎添翼！
