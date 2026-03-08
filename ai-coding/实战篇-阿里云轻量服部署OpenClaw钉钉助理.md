# 实战篇：阿里云轻量服部署 OpenClaw 打造专属钉钉助理

> **导读与场景介绍**
> 
> 本文针对已经拥有云服务器和 API 资源的开发者，提供一套**从 0 到 1 的“云端生产级部署”SOP（标准操作流程）**。
> 
> **本次实战的硬件与环境：**
> - **基建**：阿里云轻量应用服务器 (2核2G, 自带公网 IP)。
> - **大脑**：阿里云百炼 (DashScope) 企业级大模型 —— `qwen-max`。
> - **交互枢纽**：钉钉企业内部应用 (采用极简内网安全的 Stream 模式)。
> 
> **终极目标**：在服务器上不仅跑起一个会陪聊的 OpenClaw，更要让它成为一个干活的 Agent！实现高阶场景：**“你在钉钉丢给它一堆乱七八糟的会议记录，它自动归纳提炼后，主动将正式内容转发给你的老板或同事。”**

---

## 一、 服务器登入与环境体检

在云端部署，最怕的就是环境依赖问题，我们先来打通第一关。

### 1. 免密连接你的阿里云服务器
根据你提供的截图，阿里云提供了非常便利的 Web 端工具。
1. 登录阿里云控制台，找到你的实例 `OpenClaw-vnbw`。
2. 在右上角的“远程连接”中，点击 **Workbench 一键连接 (默认)**，直接以 root 或 admin 身份进入系统终端界面。

### 2. 确认 Node.js 环境
在终端里输入以下命令检查环境。OpenClaw 的运行强依赖 Node.js。

```bash
# 检查 node 版本 (建议 v18+)
node -v

# 检查 npm 包管理器
npm -v
```

> **如果提示未找到命令 (command not found)**，请用一行命令安装 Node 环境：
> `curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash - && sudo apt-get install -y nodejs` (以 Ubuntu/Debian 为例)。

### 3. 安装或更新 OpenClaw
既然你的镜像名为“OpenClaw 2026”，系统中可能已经自带了它。我们可以执行一次全局更新确保拿到最新特性：

```bash
npm install -g @openclaw/cli
```

---

## 二、 配置最强大脑：阿里云 `qwen-max` 接入

OpenClaw 默认支持通过标准的 OpenAI API 格式接入各大厂商。阿里云百炼 (DashScope) 完美兼容。

### 1. 获取百炼 API Key
前往 [阿里云百炼控制台](https://bailian.console.aliyun.com/)，在右上角（或 API-KEY 管理界面）生成一个专属的 API Key，通常以 `sk-` 开头。

### 2. 编写 `openclaw.json` 配置文件
进入你服务器的操作目录（如 `~/.openclaw`），创建或覆盖配置文件。

```bash
mkdir -p ~/.openclaw
nano ~/.openclaw/openclaw.json
```

将以下核心脑波配置复制进去。这里我们告诉 OpenClaw 去找 `dashscope` 兼容节点，并指定调用 `qwen-max`。

```json
{
  "models": {
    "providers": {
      "dashscope": {
        "url": "https://dashscope.aliyuncs.com/compatible-mode/v1",
        "key": "sk-这里填入你在百炼申请的真实API_KEY"
      }
    }
  },
  "agent": {
    "model": "dashscope/qwen-max",
    "systemPrompt": "你是一个严谨的职场办公助理，负责帮我整理杂乱的文本并执行转发任务。"
  }
}
```
保存并退出 (`Ctrl+O`, `Enter`, `Ctrl+X`)。

---

## 三、 钉钉接血：Stream 模式畅通无阻

钉钉的极速接入是我们最推荐的，特别是对于轻量服务器，不用去折腾配置 HTTPS 证书和反向代理，也不用在阿里云防火墙安全组里放行端口。

1. 登录 [钉钉开发者后台](https://open-dev.dingtalk.com/)。
2. 创建“企业内部应用” -> 添加“机器人”能力。
3. **最关键的一步**：在机器人配置的“消息接收模式”中，**必须选择 Stream 模式**！
4. 获取该应用的 `Client ID` 和 `Client Secret`。

回到你的服务器，再次打开 `openclaw.json`，在最外层加入 `channels` 配置：

```json
{
  "channels": {
    "dingtalk": {
      "clientId": "钉钉获取的ClientID",
      "clientSecret": "钉钉获取的ClientSecret"
    }
  },
  "models": { ... },
  "agent": { ... }
}
```

---

## 四、 高阶实战：让 AI 处理并转发消息

到了最激动人心的环节。你希望：“我发长语音/文本给 AI，AI 分析提炼后，自动转发给李总”。

由于大模型本身就像个“被绑在椅子上的智者”，它只会说话，你需要给它递一个“专门用来转发消息的对讲机”。在 Agent 工程师嘴里，这叫**赋予工具使用能力 (Tool Use)**。

### 方案：为 OpenClaw 挂载“人工转发 Tool”

为了实现你的需求，我们需要在服务器上写一个极简的脚本，然后告诉大模型：“当你要转发消息时，去跑这行脚本就行”。如果你使用的是完整的企业自建应用，我们可以直接通过钉钉 OpenAPI 来发送消息。

*(本节为原理示范，实际操作中你可以通过 Python 调用钉钉开放平台的 `v1.0/robot/oToMessages/batchSend` 接口)*

1.  在服务器建一个目录放我们的定制工具：`mkdir -p ~/my-tools && cd ~/my-tools`
2.  新建一个名叫 `forward.py` 的脚本：

```bash
nano forward.py
```
粘入以下测试代码（这是一个 Mock 示例，真实环境需替换为钉钉 Server API 的网络请求）：
```python
import sys
import json

# 接收大模型传给我们的参数
target_person = sys.argv[1]
refined_message = sys.argv[2]

# 这里本来是调钉钉 API 给某人发消息的代码。
# 为了演示，我们先把它伪装成发送成功，并打在屏幕上给大模型看。
print(f"【系统回执】：已成功通过钉钉接口将消息发送给 {target_person}。内容：{refined_message}")
```

3.  **在 `openclaw.json` 中给大模型配枪（授予权限）**：

在 `openclaw.json` 的 `agents` 节点下，我们需要使用 `run_command` 工具并给予 AI 系统权限：

```json
{
  "agent": {
    "model": "dashscope/qwen-max",
    "systemPrompt": "你是一个职场办公助理。你的核心工作是帮我提炼长段的会议纪要或凌乱的想法。如果我要求你把提炼后的内容发给某某某（如：李总、张三），请你主动思考提要，并通过执行 `python3 ~/my-tools/forward.py \"接收人\" \"整理后的内容\"` 命令来完成转发。",
    "allowedTools": ["bash", "run_command", "read_file"]
  }
}
```

**实战推演：**
- **你（在钉钉上发）**：“把这段话提炼成三条汇报重点，然后发给王总：‘今天这几个项目的进度太慢搞不完啊服务器又宕机了产品需求一改再改不过下周三应该能交付第一版’”。
- **qwen-max (收到后思考)**：*这人说了一堆废话。重点是1.项目进度滞后；2.服务器和需求导致；3.预计下周三交付初版。现在需要发给王总。我手里刚好有 `run_command` 工具可用。*
- **qwen-max (后台调用工具)**：执行 `bash -c "python3 ~/my-tools/forward.py '王总' '1.项目因服务器与需求变更暂时滞后；2.预计于下周三交付第一版'" `。
- **你的 Python 脚本**：调用真实的钉钉发消息 API，把这条优雅的结论推到王总手机上。
- **qwen-max (回复你的钉钉)**：“✅ 已经为您整理完毕，并成功转发给王总。”

---

## 五、 保姆级护航：部署 24 小时守护进程

我们绝不接受把 SSH 终端一关，钉钉机器人也跟着掉线的窘境。既然它是部署在轻量服务器上，我们就用 `pm2` 来把它固化为**系统级守护程序**。

1. **安装 PM2 进程管理器**
```bash
npm install -g pm2
```

2. **启动并监听 OpenClaw** (让它躲到系统后台跑)
```bash
pm2 start pi --name aliyun-assistant -- gateway
```

3. **设置开机自启** (哪怕阿里云轻量服重启，它也会自动诈尸拉起)
```bash
pm2 startup
pm2 save
```

4. **查看它的体检报告 (日常排障必备)**
如果你在钉钉说话它不理你，或者你想看看它的“心路历程 (比如它是怎么调用 forward.py 的)”，掏出这行命令：
```bash
pm2 logs aliyun-assistant
```

🎉 **恭喜！至此，你通过** `阿里云服务器 + DashScope(qwen-max) + 钉钉 Stream` **三剑客，打通了一个坚如磐石的 24 小时 AI 自动流转通道。**
