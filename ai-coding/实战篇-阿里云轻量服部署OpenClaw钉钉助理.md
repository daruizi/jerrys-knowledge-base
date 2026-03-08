# 实战篇：阿里云轻量服部署 OpenClaw 打造专属钉钉助理

> **导读与场景介绍**
> 
> 本文针对已经拥有云服务器和 API 资源的开发者，提供一套**从 0 到 1 的“云端生产级部署保姆级”SOP（标准操作流程）**。即使你是完全没有编程基础的小白，只要跟着复制粘贴，也能在 1 小时内搞定。
> 
> **本次实战的硬件与环境：**
> - **基建**：阿里云轻量应用服务器 (2核2G, 自带公网 IP)。
> - **大脑**：阿里云百炼 (DashScope) 企业级大模型 —— `qwen-max`。
> - **交互枢纽**：钉钉企业内部应用 (采用极简内网安全的 Stream 模式)。
> 
> **终极目标**：在服务器上跑起一个干活的 Agent！实现高阶场景：**“你在钉钉发给它一段长语音或会议记录，它自动归纳提炼后，主动将正式内容转发到你指定的内部钉钉群。”**
>
> ⚠️ **提前排雷：你需要具备钉钉开发者权限**
> 只有“企业/团队管理员”才能登入开发者后台。如果你平时只用钉钉和朋友聊天，请先在手机钉钉点击 **“消息界面右上角 + 号” -> “创建企业/团队”**（完全免费，随便起个名字创建即可），你就能瞬间获得管理员身份。

---

## 一、 服务器登入与环境体检

在云端部署，最怕的就是环境依赖问题，我们先来打通第一关。

### 1. 免密连接你的阿里云服务器
1. 登录阿里云控制台，找到你的轻量应用服务器实例。
2. 在右上角的“远程连接”中，点击 **Workbench 一键连接 (默认)**，直接以 root 或 admin 身份进入系统终端界面（黑框框）。

### 2. 确认 Node.js 环境
在终端黑框里输入以下命令检查环境。OpenClaw 的运行强依赖 Node.js。

```bash
# 检查 node 版本 (建议 v18+)
node -v

# 检查 npm 包管理器
npm -v
```

> **提示**：如果提示未找到命令 (command not found)，请执行以下命令一键安装 Node 环境 (基于 Ubuntu/Debian)：
> `curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash - && sudo apt-get install -y nodejs`

### 3. 确认 Python 3 环境
我们在后面的高阶步骤中会用到 Python 来运行本地发消息的脚本。现在的阿里云镜像基本都自带了。
```bash
python3 --version
```
> **提示**：如果提示找不到命令，执行 `sudo apt-get install python3` 安装即可。

### 4. 安装全局 OpenClaw
在终端执行以下命令，将 OpenClaw 安装到服务器：

```bash
npm install -g @openclaw/cli
```

---

## 二、 钉钉接血：Stream 模式配置与寻人

钉钉的 Stream (流式) 接入是我们最推荐的，特别是对于轻量服务器，不用去折腾配置 HTTPS 证书和反向代理，也不用在阿里云防火墙里放行端口。

1. 登录 [钉钉开发者后台](https://open-dev.dingtalk.com/)。
2. 点击“应用开发” -> “企业内部开发” -> 创建应用 -> 填写应用名称（如“我的专属助理”）和描述。
3. 进入应用详情页，点击左侧“应用功能” -> **“机器人”** -> 添加机器人能力。
4. **最关键的一步**：在机器人配置的“消息接收模式”中，勾选 **Stream 模式**！
5. 点击左侧“基础信息” -> “凭证与基础信息”，获取该应用的 `Client ID` 和 `Client Secret`，复制存好。
6. **上线机器人（极其重要，否则找不到它）**：点击左侧“版本管理与发布” -> “创建新版本” -> 填写版本号（如 1.0.0） -> 点击“保存并发布”。

最后，打开你电脑或手机上的钉钉客户端，在最上方的搜索栏搜索你刚才填写的应用名称（如“我的专属助理”），点击它就可以进入聊天界面了！

---

## 三、 配置最强大脑：阿里云 `qwen-max` 接入

OpenClaw 支持通过标准的 API 格式接入各大厂商。阿里云百炼 (DashScope) 完美兼容。

1. 前往 [阿里云百炼控制台](https://bailian.console.aliyun.com/)。
2. 在右上角头像处点击“API-KEY 管理”，创建一个专属的 API Key（通常以 `sk-` 开头），复制存好。

接下来，我们在服务器上创建 OpenClaw 的核心配置文件 `openclaw.json`。

在终端输入：
```bash
mkdir -p ~/.openclaw
nano ~/.openclaw/openclaw.json
```

将下面的 **完整配置代码** 粘贴进去（请务必替换掉中文部分的真实密钥，不要漏掉任何一个双引号或大括号！）：

```json
{
  "channels": {
    "dingtalk": {
      "clientId": "换成你的钉钉ClientID",
      "clientSecret": "换成你的钉钉ClientSecret"
    }
  },
  "models": {
    "providers": {
      "dashscope": {
        "url": "https://dashscope.aliyuncs.com/compatible-mode/v1",
        "key": "sk-换成你在百炼申请的真实API_KEY"
      }
    }
  },
  "agent": {
    "model": "dashscope/qwen-max",
    "systemPrompt": "你是一个职场办公助理。如果我要求你把总结内容【发送/通报】出去，请严格使用单行文本进行总结（不要包含复杂换行或特殊引号），然后执行 `python3 ~/my-tools/forward.py \"你的纯文本总结\"` 命令来完成自动推送到群的任务。",
    "allowedTools": ["bash", "run_command", "read_file"]
  }
}
```

把这段内容复制后，在服务器那黑乎乎的屏幕里**单击鼠标右键**即可粘贴。
粘贴完成后，我们要保存它：
1. 按下电脑键盘的 `Ctrl + O` (字母O，代表保存)。
2. 会提示“File Name to Write: ...”，直接按回车键 `Enter` 确认。
3. 最后按下 `Ctrl + X` 退出刚刚那个黑乎乎的编辑器。

---

## 四、 高阶实战：让 AI 处理并真实转发消息

你希望：“我发长语音/文本给 AI，AI 分析提炼后，自动转发给李总”。大模型本身只会“说话文本”，它需要一个“手”去帮你发送。

为了让小白 100% 成功，我们使用 **群自定义机器人 Webhook** 来实现转发这只“手”。

### 第一步：在钉钉建一个接收群
1. 在钉钉里建一个普通的内部群（把你和要接收消息的老板/同事拉进去）。
2. 点击群设置 -> “智能群助手” -> “添加机器人” -> 选择“自定义 (通过 Webhook 接入)”。
3. 机器人名字叫“项目通报”。
4. 🔴 **【致命防坑警告】**：在“安全设置”中，**只能勾选“加签”这一个选项！绝对不要勾选“自定义关键词”或“IP地址段”**，否则后续发送 100% 会被钉钉拦截报错！
5. 复制出 `Webhook 地址` 和 `加签密钥 (SEC 开头)`。

### 第二步：在服务器上一键生成发送脚本 (免编辑器版)
很多小白用文本编辑器粘贴 Python 代码时，经常遭遇“格式缩进错乱”导致运行失败。
为了绝对安全，我们**不用任何编辑器**。请直接全选并复制下面这一大块代码，然后去服务器黑终端里**单击鼠标右键粘贴，然后猛敲回车！**

*(注意：在这个大方块里，先把你刚刚获取的 `Webhook` 和 `加签密钥` 替换进去，再去服务器里粘贴执行👇)*

```bash
mkdir -p ~/my-tools
cat << 'EOF' > ~/my-tools/forward.py
import sys
import json
import urllib.request
import time
import hmac
import hashlib
import base64
import urllib.parse

# ===== 请务必确认下面两行填上了你真实的 URL 和 SEC 密钥 =====
WEBHOOK_URL = "https://oapi.dingtalk.com/robot/send?access_token=你的Token"
SECRET = "SEC你的加签密钥"
# =======================================================

def send_dingtalk_msg(content):
    timestamp = str(round(time.time() * 1000))
    secret_enc = SECRET.encode('utf-8')
    string_to_sign = '{}\n{}'.format(timestamp, SECRET)
    string_to_sign_enc = string_to_sign.encode('utf-8')
    hmac_code = hmac.new(secret_enc, string_to_sign_enc, digestmod=hashlib.sha256).digest()
    sign = urllib.parse.quote_plus(base64.b64encode(hmac_code))
    
    url = f"{WEBHOOK_URL}&timestamp={timestamp}&sign={sign}"
    
    headers = {'Content-Type': 'application/json'}
    # 在 Python 里安全地拼接 AI 的总结抬头，绝不可让在 Shell 下传复杂的换行符
    final_text = "【🤖 AI提炼总结】\n\n" + content
    data = {
        "msgtype": "text",
        "text": {"content": final_text}
    }
    
    req = urllib.request.Request(url=url, data=json.dumps(data).encode('utf-8'), headers=headers)
    try:
        urllib.request.urlopen(req)
        print("【系统回执】：消息已成功推送到钉钉群！")
    except Exception as e:
        print(f"【系统回执】：发送失败！错误信息：{e}")

if __name__ == "__main__":
    if len(sys.argv) > 1:
        # sys.argv[1] 接收的是 AI 传过来的纯净文本
        send_dingtalk_msg(sys.argv[1])
    else:
        print("未提供消息内容")
EOF
```

如果你粘贴完按了回车，且没有报错，说明文件已经完美生成！

### 插入验证：手动试运行脚本 (极其重要！)
在把这支“枪”交给大模型之前，我们先自己开一枪测试一下。在终端里直接敲击：
```bash
python3 ~/my-tools/forward.py "这是一条来自我亲手测试的服务器消息！"
```
如果按下回车后，你的钉钉群里立刻弹出了类似“【🤖 AI提炼总结】 这是一条来自...”的卡片，并且黑框提示“消息已成功推送”，说明通道彻底打通！

### 第三步：为什么大模型会乖乖干活？（原理解析）

你可能会问：大模型怎么知道要用这个脚本发消息？
其实，重点在我们刚刚在**第三大步**配置 `openclaw.json` 时，写进去的那段 `systemPrompt`（系统提示词）。

> *"如果我要求你把总结内容【发送/通报】出去... 然后执行 `python3 ~/my-tools/forward.py \"你的纯文本总结\"` 命令..."*

这段话就像是大模型潜意识里的“出厂准则”。当你给它下达带“发送”字眼的命令时，它大脑里的这根弦就会被触发，从而自动在服务器后台使用 `bash` 工具执行上述命令。这也是所有 AI Agent 能感知和操控物理世界的**核心奥秘**！

---

## 五、 点火测试与 24 小时守护进程部署

激动人心的时候到了！绝不盲目瞎跑，我们先做安全测试！

### 1. 前置点火测试 (Pre-flight Check)
在终端里输入：
```bash
pi gateway
```
如果看到屏幕上出现绿色的类似 `[DingTalk Stream] Connected` 的字样，说明成功连上钉钉了！
现在，打开你个人的钉钉，找到你之前搜索出来的那个机器人助手（如果没有，在搜索栏搜你的应用名），发一句：
> “把这段废话提炼出3条重点，并通报：今天这项目进度实在太慢搞不完啊服务器又宕机了产品需求一改再改不过下周三应该能交付第一版”

然后盯着服务器的黑框框！你会看到大模型在一通分析，并且**自动敲下了 `python3 ~/my-tools/forward.py ...` 的命令**。
接着，你的那个钉钉群里，会瞬间弹出一条极其专业的【AI提炼总结】卡片！

测试爽了之后，在终端按下 `Ctrl + C` 停止它。

### 2. 部署后台常驻 (进程保活)
既然它部署在轻量服务器上，我们就用 `pm2` 来把它固化为**系统级守护程序**，就算关了电脑，它也在云端为你 24 小时待命！

```bash
# 安装 PM2 进程管理器
npm install -g pm2

# 启动 OpenClaw 并让它躲到系统后台跑
pm2 start pi --name aliyun-assistant -- gateway

# 设置开机自启 (哪怕阿里云服务器重启，它也能自动拉起)
pm2 startup
pm2 save
```

### 3. 查看体检报告 (日常排障必备)
如果你在钉钉说话它突然不理你了，掏出这行命令查看它脑子里在想什么（有没有报错，是不是余额不足了）：
```bash
pm2 logs aliyun-assistant
```

🎉 **恭喜！至此，你通过** `阿里云服务器 + DashScope(qwen-max) + 钉钉 Stream` **三剑客，打通了一个坚如磐石且能真实落地干活的 24 小时 AI 代理助理。**
