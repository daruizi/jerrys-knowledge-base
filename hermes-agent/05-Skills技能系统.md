# 第 05 章：Skills 技能系统

## 什么是 Skills？

Skills 是 Hermes Agent 的**可安装能力包**。它们是便携、可共享、社区贡献的工作流程定义。

类比理解：
- **记忆** = Agent 的"知识"（知道什么）
- **技能** = Agent 的"手艺"（会做什么）

---

## 技能来源

| 来源 | 位置 | 说明 |
|------|------|------|
| 内置技能 | `skills/` | 始终可用，随项目发布 |
| 可选技能 | `optional-skills/` | 官方提供，需手动安装 |
| 社区技能 | [agentskills.io](https://agentskills.io) | 社区贡献，在线安装 |
| 自创技能 | 对话中自动生成 | Agent 从经验中提炼 |

---

## 学习闭环：技能如何从经验中诞生

这是 Hermes Agent 最独特的能力——**自我学习**：

```
完成复杂任务 → Agent 回顾执行过程
                    ↓
提炼关键步骤 → 编写技能文档
                    ↓
保存到技能目录 → 下次遇到类似任务时自动加载
                    ↓
使用过程中改进 → 发现更好的方法 → 更新技能文档
```

### 技能自我改进的触发条件

1. **技能描述不清晰** → Agent 改进了描述使其更精确
2. **步骤有遗漏** → 补充了缺失的步骤
3. **发现更高效方法** → 更新为更优方案
4. **用户反馈有效/无效** → 调整技能内容

---

## 技能文件格式

```markdown
---
name: code-review
description: 代码审查工作流
version: 1.0.0
---

# 代码审查

## 触发条件
当用户请求代码审查时

## 前置要求
- 有文件读取工具
- 有终端执行工具

## 步骤
1. 读取目标文件内容
2. 检查代码风格和命名规范
3. 识别潜在 bug 和边界情况
4. 评估性能和复杂度
5. 给出具体改进建议

## 输出格式
按严重程度排列：🔴 严重 / 🟡 警告 / 💡 建议
```

---

## 管理技能

### 安装技能

```bash
# 从 Skills Hub
hermes skills install openai/skills/k8s

# 从本地文件
hermes skills install ./my-skill/

# 在对话中
/skills install openai/skills/k8s
```

### 搜索技能

```bash
hermes skills search kubernetes
# 或对话中
/skills search kubernetes
```

### 启用/禁用技能

```bash
hermes skills enable code-review     # 启用
hermes skills disable code-review    # 禁用
```

技能可按平台分别启用/禁用：
```yaml
# config.yaml
skills:
  cli:
    enabled: [code-review, git-helper]
  telegram:
    enabled: [daily-briefing, reminder]
```

---

## 开发者：创建自定义技能

### 方法一：让 Agent 自动生成

直接让 Agent 执行复杂任务，完成后它会自动提炼为技能。

### 方法二：手动编写

```bash
# 技能目录结构
~/.hermes/skills/my-skill/
├── SKILL.md              # 技能定义
├── scripts/              # 辅助脚本（可选）
│   └── helper.sh
└── templates/            # 模板文件（可选）
    └── report-template.md
```

### SKILL.md 核心字段

```markdown
---
name: 技能唯一名称
description: 简短描述
version: 版本号
author: 作者
dependencies: [列出依赖工具]
---

# 标题

详细说明和步骤...
```

---

## 技能加载机制

Agent 启动时和技能激活时：

1. 扫描已启用的技能目录
2. 读取 SKILL.md 中的 frontmatter
3. 将技能内容注入到系统 Prompt
4. Agent 根据技能指导执行任务

---

## 最佳实践

| 实践 | 说明 |
|------|------|
| 技能粒度 | 一个技能专注一个能力，不要大而全 |
| 描述清晰 | description 要精确，帮助 Agent 正确匹配 |
| 步骤具体 | 避免模糊描述，用可执行的步骤 |
| 版本管理 | 更新技能时增加版本号 |
| 社区贡献 | 好用的技能发布到 agentskills.io |

---

> 上一章：[04-三层记忆系统](./04-三层记忆系统.md)
>
> 下一章：[06-工具系统](./06-工具系统.md)
