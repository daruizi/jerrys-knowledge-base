# Markdown 从入门到精通

> 一份全面、详细的 Markdown 语法指南，涵盖所有常用和高级语法。

---

## 目录

1. [什么是 Markdown](#什么是-markdown)
2. [基础语法](#基础语法)
   - [标题](#标题)
   - [段落](#段落)
   - [换行](#换行)
   - [强调](#强调)
   - [引用](#引用)
   - [列表](#列表)
   - [代码](#代码)
   - [分割线](#分割线)
   - [链接](#链接)
   - [图片](#图片)
   - [转义字符](#转义字符)
3. [扩展语法](#扩展语法)
   - [表格](#表格)
   - [任务列表](#任务列表)
   - [删除线](#删除线)
   - [代码块与语法高亮](#代码块与语法高亮)
   - [脚注](#脚注)
   - [定义列表](#定义列表)
   - [缩写](#缩写)
   - [Emoji 表情](#emoji-表情)
   - [GitHub Alerts 告警提示框](#github-alerts-告警提示框)
4. [高级技巧](#高级技巧)
   - [HTML 元素](#html-元素)
   - [数学公式](#数学公式)
   - [Mermaid 图表](#mermaid-图表)
   - [目录生成](#目录生成)
5. [最佳实践](#最佳实践)
6. [常用编辑器与工具](#常用编辑器与工具)

---

## 什么是 Markdown

Markdown 是一种轻量级标记语言，由 John Gruber 于 2004 年创建。它允许人们使用易读易写的纯文本格式编写文档，然后转换成有效的 HTML 文档。

**优点：**
- 纯文本格式，兼容性强
- 语法简单，学习成本低
- 关注内容而非格式
- 可版本控制
- 跨平台使用

**常见应用场景：**
- GitHub / GitLab 项目文档（README、Wiki、Issue）
- 技术博客写作（Hugo、Hexo、Jekyll）
- 笔记管理（Obsidian、Notion、Logseq）
- 即时通讯（Slack、Discord、飞书）
- API 文档与知识库

---

## 基础语法

### 标题

Markdown 支持六级标题，使用 `#` 号表示。

**语法：**
```markdown
# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题
```

**演示效果：**

# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题

> **注意：** `#` 号后面需要加一个空格，这是标准 Markdown 语法。

---

### 段落

段落由一行或多行文本组成，段落之间用空行分隔。

**语法：**
```markdown
这是第一个段落。段落可以包含多行文本，
只要没有空行分隔，就属于同一个段落。

这是第二个段落。两个段落之间有一个空行。
```

**演示效果：**

这是第一个段落。段落可以包含多行文本，
只要没有空行分隔，就属于同一个段落。

这是第二个段落。两个段落之间有一个空行。

---

### 换行

在段落内换行有两种方式：

**方式一：使用两个空格 + 回车**
```markdown
第一行（末尾添加两个空格）  
第二行
```

> **提示：** 两个尾随空格在源码中不可见，部分编辑器会自动去除尾随空格，因此推荐使用方式二。

**方式二：使用 HTML 的 `<br>` 标签**
```markdown
第一行<br>第二行
```

**演示效果：**

第一行  
第二行

---

### 强调

Markdown 提供三种强调方式：粗体、斜体、粗斜体。

**语法：**
```markdown
**粗体文本**
__粗体文本__

*斜体文本*
_斜体文本_

***粗斜体文本***
___粗斜体文本___

**粗体中包含*斜体*文本**
```

**演示效果：**

**粗体文本**
__粗体文本__

*斜体文本*
_斜体文本_

***粗斜体文本***
___粗斜体文本___

**粗体中包含*斜体*文本**

---

### 引用

使用 `>` 符号创建引用块，支持嵌套。

**语法：**
```markdown
> 这是一级引用
>
> > 这是二级引用
> >
> > > 这是三级引用

> 引用中可以包含其他 Markdown 元素
>
> - 列表项一
> - 列表项二
>
> **粗体文本**
```

**演示效果：**

> 这是一级引用
>
> > 这是二级引用
> >
> > > 这是三级引用

> 引用中可以包含其他 Markdown 元素
>
> - 列表项一
> - 列表项二
>
> **粗体文本**

---

### 列表

#### 无序列表

使用 `*`、`+` 或 `-` 作为列表标记。

**语法：**
```markdown
* 项目一
* 项目二
  * 子项目 2.1
  * 子项目 2.2
* 项目三

+ 项目 A
+ 项目 B

- 项目 X
- 项目 Y
```

**演示效果：**

* 项目一
* 项目二
  * 子项目 2.1
  * 子项目 2.2
* 项目三

#### 有序列表

使用数字加点表示有序列表。

**语法：**
```markdown
1. 第一项
2. 第二项
3. 第三项
   1. 子项 3.1
   2. 子项 3.2
4. 第四项
```

**演示效果：**

1. 第一项
2. 第二项
3. 第三项
   1. 子项 3.1
   2. 子项 3.2
4. 第四项

> **提示：** 列表序号可以不按顺序，Markdown 会自动排序。

#### 列表嵌套

**语法：**
```markdown
1. 第一项
   - 子项 1.1
   - 子项 1.2
2. 第二项
   - 子项 2.1
     - 子子项 2.1.1
```

**演示效果：**

1. 第一项
   - 子项 1.1
   - 子项 1.2
2. 第二项
   - 子项 2.1
     - 子子项 2.1.1

---

### 代码

#### 行内代码

使用反引号包裹代码片段。

**语法：**
```markdown
使用 `printf()` 函数输出内容。
```

**演示效果：**

使用 `printf()` 函数输出内容。

#### 代码块

使用三个反引号包裹代码块，可指定语言实现语法高亮。

**语法：**
````markdown
```python
def hello_world():
    print("Hello, World!")

hello_world()
```
````

**演示效果：**
```python
def hello_world():
    print("Hello, World!")

hello_world()
```

> **提示：** 更多语言标识符和多语言示例，请参阅下方 [代码块与语法高亮](#代码块与语法高亮) 部分。

---

### 分割线

使用三个或更多的 `*`、`-` 或 `_` 创建分割线。

**语法：**
```markdown
***

---

___
```

**演示效果：**

上面的分割线

***

中间的分割线

---

下面的分割线

___

---

### 链接

#### 基本链接

**语法：**
```markdown
[链接文本](URL)

[GitHub](https://github.com)
```

**演示效果：**

[GitHub](https://github.com)

#### 带标题的链接

**语法：**
```markdown
[链接文本](URL "标题")

[GitHub](https://github.com "访问 GitHub")
```

**演示效果：**

[GitHub](https://github.com "访问 GitHub")

#### 引用式链接

**语法：**
```markdown
[链接文本][引用ID]

[引用ID]: URL "可选标题"

这是一个[引用式链接][google]

[google]: https://www.google.com "Google 搜索"
```

**演示效果：**

这是一个[引用式链接][google]

[google]: https://www.google.com "Google 搜索"

#### 直接链接

**语法：**
```markdown
<https://www.example.com>

<email@example.com>
```

**演示效果：**

<https://www.example.com>

<email@example.com>

---

### 图片

#### 基本图片

**语法：**
```markdown
![替代文本](图片URL)

![Markdown Logo](https://markdown.com.cn/hero.png)
```

**演示效果：**

![Markdown Logo](https://markdown.com.cn/hero.png)

#### 带标题的图片

**语法：**
```markdown
![替代文本](图片URL "标题")

![示例图片](https://via.placeholder.com/150 "这是一个示例图片")
```

**演示效果：**

![示例图片](https://via.placeholder.com/150 "这是一个示例图片")

#### 引用式图片

**语法：**
```markdown
![替代文本][图片ID]

[图片ID]: 图片URL "标题"

![Markdown Logo][md-logo]

[md-logo]: https://markdown.com.cn/hero.png "Markdown Logo"
```

**演示效果：**

![Markdown Logo][md-logo]

[md-logo]: https://markdown.com.cn/hero.png "Markdown Logo"

#### 图片链接

**语法：**
```markdown
[![图片替代文本](图片URL)](链接URL)

[![GitHub](https://github.githubassets.com/images/modules/logos_page/GitHub-Mark.png)](https://github.com)
```

**演示效果：**

[![GitHub](https://github.githubassets.com/images/modules/logos_page/GitHub-Mark.png)](https://github.com)

#### 控制图片大小

纯 Markdown 不支持控制图片大小，可使用 HTML `<img>` 标签实现：

```html
<img src="https://markdown.com.cn/hero.png" alt="Markdown Logo" width="200">
```

**演示效果：**

<img src="https://markdown.com.cn/hero.png" alt="Markdown Logo" width="200">

---

### 转义字符

使用反斜杠转义 Markdown 特殊字符。

**语法：**
```markdown
\* 不是斜体
\# 不是标题
\[ 不是链接
```

**演示效果：**

\* 不是斜体
\# 不是标题
\[ 不是链接

**可转义的字符：**

| 字符 | 名称 |
|------|------|
| \\ | 反斜杠 |
| \` | 反引号 |
| \* | 星号 |
| \_ | 下划线 |
| \{ \} | 花括号 |
| \[ \] | 方括号 |
| \( \) | 圆括号 |
| \# | 井号 |
| \+ | 加号 |
| \- | 减号 |
| \. | 英文句号 |
| \! | 感叹号 |
| \| | 竖线 |

---

## 扩展语法

### 表格

使用竖线和连字符创建表格。

**语法：**
```markdown
| 表头1 | 表头2 | 表头3 |
|-------|-------|-------|
| 内容1 | 内容2 | 内容3 |
| 内容4 | 内容5 | 内容6 |
```

**演示效果：**

| 表头1 | 表头2 | 表头3 |
|-------|-------|-------|
| 内容1 | 内容2 | 内容3 |
| 内容4 | 内容5 | 内容6 |

#### 对齐方式

**语法：**
```markdown
| 左对齐 | 居中对齐 | 右对齐 |
|:-------|:--------:|-------:|
| 内容   | 内容     | 内容   |
| 数据   | 数据     | 数据   |
```

**演示效果：**

| 左对齐 | 居中对齐 | 右对齐 |
|:-------|:--------:|-------:|
| 内容   | 内容     | 内容   |
| 数据   | 数据     | 数据   |

---

### 任务列表

创建带有复选框的任务列表。

**语法：**
```markdown
- [x] 已完成的任务
- [ ] 未完成的任务
- [ ] 另一个未完成的任务
  - [x] 子任务已完成
  - [ ] 子任务未完成
```

**演示效果：**

- [x] 已完成的任务
- [ ] 未完成的任务
- [ ] 另一个未完成的任务
  - [x] 子任务已完成
  - [ ] 子任务未完成

---

### 删除线

使用双波浪线表示删除线（GFM 扩展语法）。

**语法：**
```markdown
~~这段文字被删除了~~
```

**演示效果：**

~~这段文字被删除了~~

---

### 代码块与语法高亮

支持多种编程语言的语法高亮。

**常用语言列表：**

| 语言 | 标识符 |
|------|--------|
| JavaScript | `javascript` 或 `js` |
| Python | `python` 或 `py` |
| Java | `java` |
| C | `c` |
| C++ | `cpp` 或 `c++` |
| C# | `csharp` 或 `cs` |
| PHP | `php` |
| Ruby | `ruby` |
| Go | `go` |
| Rust | `rust` |
| Swift | `swift` |
| Kotlin | `kotlin` |
| TypeScript | `typescript` 或 `ts` |
| HTML | `html` |
| CSS | `css` |
| SQL | `sql` |
| Shell | `shell` 或 `bash` |
| JSON | `json` |
| YAML | `yaml` |
| XML | `xml` |
| Markdown | `markdown` 或 `md` |

**多语言示例：**

```javascript
// JavaScript 示例
const greeting = "Hello, Markdown!";
console.log(greeting);
```

```java
// Java 示例
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello, Markdown!");
    }
}
```

```typescript
// TypeScript 示例
interface User {
  name: string;
  age: number;
}

const greet = (user: User): string => {
  return `Hello, ${user.name}!`;
};
```

```go
// Go 示例
package main

import "fmt"

func main() {
    fmt.Println("Hello, Markdown!")
}
```

```rust
// Rust 示例
fn main() {
    println!("Hello, Markdown!");
}
```

```sql
-- SQL 示例
SELECT * FROM users WHERE status = 'active';
```

```json
{
  "name": "Markdown Tutorial",
  "version": "1.0.0",
  "author": "Jerry"
}
```

```bash
# Bash 脚本示例
#!/bin/bash
echo "Hello, Markdown!"
```

---

### 脚注

添加脚注引用。

**语法：**
```markdown
这是一个脚注引用[^1]。

[^1]: 这是脚注的内容。
```

**演示效果：**

这是一个脚注引用[^1]。

[^1]: 这是脚注的内容。

---

### 定义列表

部分 Markdown 解析器支持定义列表。

**语法：**
```markdown
术语 1
: 定义 1

术语 2
: 定义 2a
: 定义 2b
```

**演示效果：**

术语 1
: 定义 1

术语 2
: 定义 2a
: 定义 2b

---

### 缩写

部分 Markdown 解析器支持缩写定义。

**语法：**
```markdown
HTML 是一种标记语言。

*[HTML]: Hyper Text Markup Language
```

**演示效果：**

HTML 是一种标记语言。

*[HTML]: Hyper Text Markup Language

---

### Emoji 表情

许多 Markdown 解析器（如 GitHub、Slack）支持 Emoji 短代码。

**语法：**
```markdown
:smile: :rocket: :+1: :heart: :fire: :star:
```

**常用 Emoji 短代码：**

| Emoji | 短代码 | 含义 |
|-------|--------|------|
| 😄 | `:smile:` | 笑脸 |
| 🚀 | `:rocket:` | 火箭 |
| 👍 | `:+1:` | 点赞 |
| ❤️ | `:heart:` | 爱心 |
| 🔥 | `:fire:` | 火焰 |
| ⭐ | `:star:` | 星星 |
| ⚠️ | `:warning:` | 警告 |
| ✅ | `:white_check_mark:` | 完成 |
| ❌ | `:x:` | 错误 |
| 📝 | `:memo:` | 备忘 |

> **提示：** 也可以直接在 Markdown 中粘贴 Unicode Emoji，如 😀 🎉 💡，无需短代码。

---

### GitHub Alerts 告警提示框

GitHub Flavored Markdown 支持五种告警提示框样式，用于突出重要信息。

**语法：**
```markdown
> [!NOTE]
> 有用的补充信息。

> [!TIP]
> 更好的操作建议。

> [!IMPORTANT]
> 用户需要知道的关键信息。

> [!WARNING]
> 需要用户注意的潜在问题。

> [!CAUTION]
> 可能导致严重后果的操作提醒。
```

**演示效果：**

> [!NOTE]
> 有用的补充信息。

> [!TIP]
> 更好的操作建议。

> [!IMPORTANT]
> 用户需要知道的关键信息。

> [!WARNING]
> 需要用户注意的潜在问题。

> [!CAUTION]
> 可能导致严重后果的操作提醒。

---

## 高级技巧

### HTML 元素

Markdown 支持内嵌 HTML 元素。

**常用 HTML 元素示例：**

```markdown
<div style="color: red; font-size: 20px;">
  这是红色大号文字
</div>

<kbd>Ctrl</kbd> + <kbd>C</kbd> 复制

<mark>高亮文本</mark>

<sub>下标</sub> 和 <sup>上标</sup>
```

**演示效果：**

<div style="color: red; font-size: 20px;">
  这是红色大号文字
</div>

<kbd>Ctrl</kbd> + <kbd>C</kbd> 复制

<mark>高亮文本</mark>

H<sub>2</sub>O 和 X<sup>2</sup>

#### 折叠/展开内容

使用 `<details>` 和 `<summary>` 标签创建可折叠内容，适合隐藏冗长的细节。

**语法：**
```html
<details>
<summary>点击展开详情</summary>

这里是被折叠的内容。

- 支持 Markdown 语法
- 可以包含代码块、列表等

</details>
```

**演示效果：**

<details>
<summary>点击展开详情</summary>

这里是被折叠的内容。

- 支持 Markdown 语法
- 可以包含代码块、列表等

</details>

---

### 数学公式

使用 LaTeX 语法编写数学公式（需要支持 MathJax 或 KaTeX 的解析器）。

#### 行内公式

**语法：**
```markdown
质能方程 $E = mc^2$ 是物理学中最著名的公式之一。
```

**演示效果：**

质能方程 $E = mc^2$ 是物理学中最著名的公式之一。

#### 块级公式

**语法：**
```markdown
$$
\frac{-b \pm \sqrt{b^2 - 4ac}}{2a}
$$
```

**演示效果：**

$$
\frac{-b \pm \sqrt{b^2 - 4ac}}{2a}
$$

#### 更多公式示例

**求和公式：**
$$
\sum_{i=1}^{n} x_i = x_1 + x_2 + \cdots + x_n
$$

**积分公式：**
$$
\int_{a}^{b} f(x) \, dx
$$

**矩阵：**
$$
\begin{bmatrix}
a & b \\
c & d
\end{bmatrix}
$$

---

### Mermaid 图表

使用 Mermaid 语法绘制各类图表（需要解析器支持，如 GitHub、Typora、Obsidian 等）。

#### 流程图

**语法：**
````markdown
```mermaid
graph TD
    A[开始] --> B{是否继续?}
    B -->|是| C[执行操作]
    B -->|否| D[结束]
    C --> D
```
````

**演示效果：**

```mermaid
graph TD
    A[开始] --> B{是否继续?}
    B -->|是| C[执行操作]
    B -->|否| D[结束]
    C --> D
```

#### 时序图

**语法：**
````markdown
```mermaid
sequenceDiagram
    participant A as 用户
    participant B as 服务器
    participant C as 数据库
    A->>B: 发送请求
    B->>C: 查询数据
    C-->>B: 返回数据
    B-->>A: 响应结果
```
````

**演示效果：**

```mermaid
sequenceDiagram
    participant A as 用户
    participant B as 服务器
    participant C as 数据库
    A->>B: 发送请求
    B->>C: 查询数据
    C-->>B: 返回数据
    B-->>A: 响应结果
```

#### 饼图

**语法：**
````markdown
```mermaid
pie title 编程语言使用比例
    "JavaScript" : 40
    "Python" : 30
    "Java" : 20
    "其他" : 10
```
````

**演示效果：**

```mermaid
pie title 编程语言使用比例
    "JavaScript" : 40
    "Python" : 30
    "Java" : 20
    "其他" : 10
```

#### 甘特图

**语法：**
````markdown
```mermaid
gantt
    title 项目开发计划
    dateFormat  YYYY-MM-DD
    section 设计阶段
    需求分析     :a1, 2024-01-01, 7d
    系统设计     :a2, after a1, 5d
    section 开发阶段
    前端开发     :b1, after a2, 14d
    后端开发     :b2, after a2, 14d
    section 测试阶段
    功能测试     :c1, after b1, 7d
    上线部署     :c2, after c1, 3d
```
````

**演示效果：**

```mermaid
gantt
    title 项目开发计划
    dateFormat  YYYY-MM-DD
    section 设计阶段
    需求分析     :a1, 2024-01-01, 7d
    系统设计     :a2, after a1, 5d
    section 开发阶段
    前端开发     :b1, after a2, 14d
    后端开发     :b2, after a2, 14d
    section 测试阶段
    功能测试     :c1, after b1, 7d
    上线部署     :c2, after c1, 3d
```

#### 类图

**语法：**
````markdown
```mermaid
classDiagram
    Animal <|-- Duck
    Animal <|-- Fish
    Animal <|-- Zebra
    Animal : +int age
    Animal : +String gender
    Animal: +isMammal()
    Animal: +mate()
    class Duck{
        +String beakColor
        +swim()
        +quack()
    }
    class Fish{
        -int sizeInFeet
        -canEat()
    }
    class Zebra{
        +bool is_wild
        +run()
    }
```
````

**演示效果：**

```mermaid
classDiagram
    Animal <|-- Duck
    Animal <|-- Fish
    Animal <|-- Zebra
    Animal : +int age
    Animal : +String gender
    Animal: +isMammal()
    Animal: +mate()
    class Duck{
        +String beakColor
        +swim()
        +quack()
    }
    class Fish{
        -int sizeInFeet
        -canEat()
    }
    class Zebra{
        +bool is_wild
        +run()
    }
```

---

### 目录生成

部分 Markdown 解析器支持自动生成目录。

**语法：**
```markdown
[toc]
```

或手动创建目录：

```markdown
## 目录

- [标题一](#标题一)
  - [子标题](#子标题)
- [标题二](#标题二)
```

---

## 最佳实践

### 1. 保持一致性

- 统一使用一种列表标记（推荐 `-`）
- 统一标题风格
- 统一代码块语言标识

### 2. 可读性优先

- 适当使用空行分隔内容
- 避免过长的行
- 合理使用标题层级

### 3. 链接和图片

- 使用引用式链接保持正文简洁
- 为图片提供有意义的替代文本
- 检查链接有效性

### 4. 代码块

- 始终指定语言以启用语法高亮
- 保持代码简洁、可运行
- 添加必要的注释

### 5. 表格

- 保持列对齐
- 避免过宽的表格
- 复杂数据考虑使用列表

### 6. 兼容性考虑

- 不同平台支持的语法可能不同
- 测试在目标平台的渲染效果
- 必要时使用 HTML 作为补充

---

## 常用编辑器与工具

### 桌面编辑器

| 编辑器 | 特点 | 平台 |
|--------|------|------|
| [VS Code](https://code.visualstudio.com/) | 强大的扩展生态，内置预览 | Windows / macOS / Linux |
| [Typora](https://typora.io/) | 所见即所得，极简体验 | Windows / macOS / Linux |
| [Obsidian](https://obsidian.md/) | 双向链接，知识图谱 | Windows / macOS / Linux |
| [Mark Text](https://marktext.app/) | 开源免费，实时预览 | Windows / macOS / Linux |

### 在线工具

| 工具 | 特点 |
|------|------|
| [StackEdit](https://stackedit.io/) | 支持同步 Google Drive、GitHub |
| [Dillinger](https://dillinger.io/) | 简洁的在线 Markdown 编辑器 |
| [HackMD](https://hackmd.io/) | 多人协作编辑 |

---

## 快速参考卡片

````
# 标题
## 二级标题
### 三级标题

**粗体** *斜体* ***粗斜体***

~~删除线~~

> 引用

- 无序列表
1. 有序列表

- [ ] 任务列表
- [x] 已完成

`行内代码`

```语言
代码块
```

[链接](URL)
![图片](URL)

| 表格 | 表格 |
|------|------|
| 内容 | 内容 |

---

*斜体* 或 _斜体_
**粗体** 或 __粗体__

[引用式链接][id]
[id]: URL

脚注[^1]
[^1]: 脚注内容

:emoji_name:

> [!NOTE]
> 告警提示框
````

---

## 结语

Markdown 是一种强大而简洁的标记语言，掌握这些语法后，你可以：

- 编写清晰的技术文档
- 创建专业的 README 文件
- 撰写博客文章
- 记录学习笔记
- 编写书籍

**推荐资源：**

- [Markdown 官方文档](https://daringfireball.net/projects/markdown/)
- [GitHub Flavored Markdown](https://github.github.com/gfm/)
- [CommonMark 规范](https://commonmark.org/)
- [Emoji 速查表](https://www.webfx.com/tools/emoji-cheat-sheet/)

---

*文档创建时间：2024年*
*最后更新：2026年3月*
*作者：Jerry*