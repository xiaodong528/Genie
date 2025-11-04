# 快速入门

## 概述

本教程将通过**计算器应用示例**,演示如何使用 Genie 项目从零开始开发一个完整的全栈应用,包括:

- 🗄️ 数据库设计和创建
- 🔧 AIPEXBASE 后端集成
- 💻 前端代码生成
- 🌐 Web 服务部署
- ✅ 功能测试

**预计时间**: 15-20 分钟

## 前置条件

### 环境要求

- Python 3.8+
- Node.js 16+ (用于 Claude Code CLI)
- Git
- 文本编辑器 (VS Code 推荐)

### 账号准备

1. **E2B 账号**: https://e2b.dev/dashboard
2. **Claude API**: Anthropic API 密钥或智谱 AI 代理
3. **AIPEXBASE 服务器**: 已部署的 AIPEXBASE 后端

## 第一步: 环境搭建 (5 分钟)

### 1.1 克隆项目

```bash
# 克隆项目
git clone <repository-url>
cd Genie

# 查看项目结构
ls -la
# 应该看到: src/, docs/, tests/, requirements.txt 等
```

### 1.2 安装依赖

```bash
# 创建虚拟环境(推荐)
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate

# 安装依赖
pip install -r requirements.txt

# 验证安装
python -c "import e2b; print('E2B SDK:', e2b.__version__)"
python -c "import claude_agent_sdk; print('Claude Agent SDK installed')"
```

### 1.3 配置环境变量

```bash
# 复制环境变量模板
cp .env.example .env

# 编辑 .env 文件
vim .env  # 或使用其他编辑器
```

**必须配置的变量**:

```bash
# E2B Sandbox 服务
E2B_API_KEY=e2b_xxxxxxxxxxxxxx

# Claude API
ANTHROPIC_AUTH_TOKEN=your_token_here
ANTHROPIC_BASE_URL=https://open.bigmodel.cn/api/anthropic

# AIPEXBASE 后端服务器
AIPEXBASE_BASE_URL=http://your-server:8080
AIPEXBASE_ADMIN_EMAIL=admin@example.com
AIPEXBASE_ADMIN_PASSWORD=your_password
```

**获取 API Keys**:

- E2B_API_KEY: 访问 https://e2b.dev/dashboard → Settings → API Keys
- ANTHROPIC_AUTH_TOKEN: 访问智谱 AI 平台获取

### 1.4 验证配置

```bash
# 测试 E2B 连接
python -c "from e2b import AsyncSandbox; print('E2B configuration OK')"

# 检查环境变量
python -c "import os; from dotenv import load_dotenv; load_dotenv(); print('ANTHROPIC_AUTH_TOKEN:', 'OK' if os.getenv('ANTHROPIC_AUTH_TOKEN') else 'Missing')"
```

## 第二步: 创建 AIPEXBASE 项目 (5 分钟)

### 2.1 运行项目创建脚本

```bash
# 使用 aipexbase.py 创建项目
python src/aipexbase.py "计算器演示项目"
```

**预期输出**:

```
🚀 AIPEXBASE 项目自动创建工具
============================================================
📋 配置信息:
   服务器地址: http://your-server:8080
   管理员邮箱: admin@example.com

🔐 正在登录...
✅ 登录成功

📦 正在创建项目: 计算器演示项目
✅ 项目创建成功
   项目 ID: 123
   项目名称: 计算器演示项目

🔑 正在创建 API 密钥...
✅ API 密钥创建成功
   密钥 ID: 456
   密钥名称: MCP 专用密钥

============================================================
📋 项目创建结果
============================================================
✅ 项目 ID: 123
✅ 项目名称: 计算器演示项目
✅ API 密钥 ID: 456
🔌 MCP URL: http://your-server:8080/mcp/sse?token=kf_api_xxxxxxxxxxxxxxxxxx

============================================================
💡 使用提示
============================================================
1. 将上述 MCP URL 配置到 src/agent_runner.py 第 247 行
2. 运行 python src/build_template.py 构建 E2B Template
3. 运行 python src/apps/calculator.py 启动应用
============================================================
```

### 2.2 记录关键信息

**务必记录以下信息**:

- 项目 ID: `123`
- MCP URL: `http://your-server:8080/mcp/sse?token=kf_api_xxxxxxxxxxxxxxxxxx`

### 2.3 配置 MCP 服务器

编辑 `src/agent_runner.py`,更新第 247 行:

```python
# src/agent_runner.py (行 245-250)
mcp_cmd = (
    'claude mcp add --transport sse --scope user '
    'aipexbase-mcp-server '
    '"http://your-server:8080/mcp/sse?token=kf_api_xxxxxxxxxxxxxxxxxx"'  # 替换为你的 MCP URL
)
```

**替换为你刚才获取的 MCP URL**!

## 第三步: 构建 E2B Template (3 分钟)

### 3.1 理解 Template

`src/template.py` 定义了 Sandbox 的运行环境,包括:

- 基础镜像: Ubuntu
- 安装的软件: Node.js, Python, Claude Code CLI
- 环境变量: ANTHROPIC_AUTH_TOKEN 等

### 3.2 构建 Template

```bash
# 运行构建脚本
python src/build_template.py
```

**预期输出**:

```
🚀 开始构建 E2B Template...
============================================================
📋 构建配置:
   别名: claude-agent-sandbox
   CPU: 2 核
   内存: 2048 MB
============================================================

📦 正在构建 Template...

[构建日志]
Step 1/10 : FROM base_image
Step 2/10 : RUN apt-get update
...
Step 10/10 : CMD echo "Ready"

✅ Template 构建成功！
Template ID: tpl_xxxxxxxxxxxxxx

✅ Template ID 已保存到 .template_id 文件

============================================================
💡 使用提示
============================================================
Template ID 已保存,可以直接运行应用:
  python src/apps/calculator.py
============================================================
```

### 3.3 验证 Template

```bash
# 查看生成的 .template_id 文件
cat .template_id

# 应该输出:
# TEMPLATE_ID=tpl_xxxxxxxxxxxxxx
```

## 第四步: 理解示例代码 (5 分钟)

### 4.1 计算器 Agent 脚本

查看 `src/code/calculator.py` (关键部分):

```python
# src/code/calculator.py (简化版)
from claude_agent_sdk import ClaudeSDKClient, ClaudeAgentOptions
from prompt import append_prompt  # 导入 AIPEXBASE 开发规范

async def main():
    # 配置 Agent
    options = ClaudeAgentOptions(
        allowed_tools=[
            "Bash", "Read", "Write", "Glob", "Grep",
            "execute_sql",      # AIPEXBASE MCP 工具
            "list_tables",      # AIPEXBASE MCP 工具
            "list_dynamic_api"  # AIPEXBASE MCP 工具
        ],
        permission_mode="bypassPermissions",
        system_prompt={
            "type": "preset",
            "preset": "claude_code",
            "append": append_prompt  # 嵌入开发规范
        }
    )

    # 创建 Client 并发送任务
    async with ClaudeSDKClient(options) as client:
        await client.query(
            "创建一个带 Web 前端的计算器应用，使用 AIPEXBASE 后端存储计算历史记录"
        )

        # 接收响应
        async for message in client.receive_response():
            print(message)
```

**关键点**:

1. **导入 prompt.py**: 提供 AIPEXBASE 开发规范
2. **启用 MCP 工具**: `execute_sql`, `list_tables`, `list_dynamic_api`
3. **任务描述**: 明确要求使用 AIPEXBASE 后端

### 4.2 应用运行器

查看 `src/apps/calculator.py`:

```python
# src/apps/calculator.py (简化版)
from agent_runner import run_code_with_service

async def main():
    print("🧮 计算器应用生成器")

    # 运行 Agent 脚本并启动服务
    result = await run_code_with_service(
        code_file="calculator.py",  # 对应 src/code/calculator.py
        service_port=3000,
        wait_time=5
    )

    # 输出服务 URL
    print(f"✅ 服务 URL: {result['service_url']}")
    print(f"✅ Sandbox ID: {result['sandbox_id']}")
```

**工作流程**:

1. 读取 Template ID
2. 创建 E2B Sandbox
3. 上传 `code/calculator.py` 到 Sandbox
4. 配置 MCP 服务器
5. 执行 Agent 脚本
6. Agent 创建数据库表
7. Agent 生成前端代码
8. Agent 启动 HTTP 服务器
9. 返回外部访问 URL

## 第五步: 运行计算器应用 (5 分钟)

### 5.1 启动应用生成器

```bash
# 运行应用运行器
python src/apps/calculator.py
```

### 5.2 观察执行过程

**阶段 1: 初始化** (10-15 秒)

```
🧮 计算器应用生成器
============================================================
✅ 环境变量检查通过
📋 读取 Template ID...
✅ Template ID: tpl_xxxxxxxxxxxxxx
📄 读取代码文件: calculator.py
✅ 代码大小: 15255 字节
🚀 创建 Sandbox...
✅ Sandbox 已创建 (ID: sbx_xxxxxxxxxxxxxx)
```

**阶段 2: MCP 配置** (5 秒)

```
📤 上传代码到 Sandbox...
✅ 代码文件已上传

🔧 配置 MCP 服务器...
[MCP] Added MCP server 'aipexbase-mcp-server'
✅ MCP 服务器配置成功
```

**阶段 3: Agent 执行** (1-3 分钟)

```
🚀 执行代码: python /home/user/calculator.py

==================================================
[Agent] 我将创建一个带 Web 前端的计算器应用...
[Agent] 首先设计数据库表结构...
[Agent] 创建 calculations 表用于存储计算历史...
[Agent] 执行 SQL: CREATE TABLE IF NOT EXISTS calculations (...)
[Agent] ✅ 数据库表创建成功
[Agent] 查询可用的 API 端点...
[Agent] 生成前端 HTML 文件...
[Agent] 添加样式和 JavaScript...
[Agent] 使用 aipexbase-js SDK 连接后端...
[Agent] 启动 HTTP 服务器在 3000 端口...
==================================================
```

**阶段 4: 服务启动** (5 秒)

```
✅ 执行完成 (退出码: 0)
⏳ 等待服务启动 (5 秒)...
🌐 获取服务 URL (端口 3000)...
✅ 服务 URL: https://sbx-xxxxxxxxxxxxxx-3000.e2b.dev

📂 检查生成的文件...
✅ 生成的文件:
  - index.html
  - README.md

============================================================
🌐 Web 服务信息
============================================================
✅ 前端地址: https://sbx-xxxxxxxxxxxxxx-3000.e2b.dev
✅ Sandbox ID: sbx_xxxxxxxxxxxxxx
💡 使用提示:
  1. 在浏览器中打开上述地址访问计算器应用
  2. Sandbox 将保持运行约 1 小时（3600 秒）
  3. 服务超时后会自动关闭
============================================================
```

### 5.3 访问应用

1. 复制输出的服务 URL: `https://sbx-xxxxxxxxxxxxxx-3000.e2b.dev`
2. 在浏览器中打开
3. 看到计算器界面

## 第六步: 测试应用功能 (3 分钟)

### 6.1 功能测试

**测试计算功能**:

1. 输入第一个数字: `10`
2. 选择运算符: `+`
3. 输入第二个数字: `5`
4. 点击 `=` 按钮
5. 查看结果: `15`

**测试历史记录**:

1. 执行几次计算 (如 10+5, 20-3, 6*7)
2. 查看计算历史列表
3. 确认所有计算都被记录

### 6.2 数据库验证

访问 AIPEXBASE 后台:

1. 打开 AIPEXBASE 管理界面
2. 选择 "计算器演示项目"
3. 查看 `calculations` 表
4. 确认有数据记录

**预期表结构**:

| 字段名     | 类型        | 说明                |
| ---------- | ----------- | ------------------- |
| id         | bigint      | 主键                |
| num1       | double      | 第一个数字          |
| num2       | double      | 第二个数字          |
| operator   | varchar(10) | 运算符 (+, -, *, /) |
| result     | double      | 计算结果            |
| created_at | datetime    | 创建时间            |

**数据示例**:

| id | num1 | num2 | operator | result | created_at          |
| -- | ---- | ---- | -------- | ------ | ------------------- |
| 1  | 10.0 | 5.0  | +        | 15.0   | 2024-01-15 10:30:00 |
| 2  | 20.0 | 3.0  | -        | 17.0   | 2024-01-15 10:30:15 |

### 6.3 前端代码审查

下载生成的 `index.html`:

```bash
# 通过 E2B Dashboard 或 API 下载文件
# 或查看 Sandbox 文件系统
```

**关键代码片段**:

```html
<!DOCTYPE html>
<html>
<head>
    <title>计算器</title>
    <script src="https://cdn.jsdelivr.net/npm/aipexbase-js/dist/index.js"></script>
</head>
<body>
    <div id="calculator">
        <!-- 计算器界面 -->
    </div>

    <script>
        // 初始化 AIPEXBASE Client
        const client = new AIPEXBASEClient({
            baseURL: 'http://your-server:8080',
            projectId: '123',
            token: 'xxx'
        });

        // 计算函数
        async function calculate(num1, num2, operator) {
            const result = eval(`${num1} ${operator} ${num2}`);

            // 保存到数据库
            await client.create('calculations', {
                num1, num2, operator, result
            });

            return result;
        }

        // 加载历史记录
        async function loadHistory() {
            const history = await client.list('calculations', {
                orderBy: 'created_at DESC',
                limit: 10
            });

            // 显示在界面上
            renderHistory(history);
        }
    </script>
</body>
</html>
```

## 常见问题解答

### Q1: Template 构建失败

```
Error: E2B_API_KEY is invalid
```

**解决方法**:

1. 检查 `.env` 文件中的 `E2B_API_KEY` 是否正确
2. 访问 https://e2b.dev/dashboard 验证 API Key 是否有效
3. 重新复制 API Key

### Q2: MCP 配置失败

```
[MCP Error] Failed to add MCP server
```

**解决方法**:

1. 检查 `agent_runner.py` 第 247 行的 MCP URL 是否正确
2. 确认 AIPEXBASE 服务器可访问
3. 验证 Token 是否有效

### Q3: Agent 不知道如何使用 AIPEXBASE

```
[Agent] I don't know how to create a database
```

**解决方法**:

确认 `code/calculator.py` 中正确导入并使用了 `prompt.py`:

```python
from prompt import append_prompt

options = ClaudeAgentOptions(
    system_prompt={
        "type": "preset",
        "preset": "claude_code",
        "append": append_prompt  # 必须有这一行!
    }
)
```

### Q4: 服务 URL 访问失败

```
This site can't be reached
```

**解决方法**:

1. 确认 Sandbox 仍在运行(默认 1 小时超时)
2. 检查 HTTP 服务器是否正常启动
3. 查看 Agent 执行日志中是否有错误

### Q5: 计算结果不保存到数据库

**解决方法**:

1. 检查 AIPEXBASE 后台是否有 `calculations` 表
2. 查看浏览器控制台是否有 API 调用错误
3. 确认 Token 权限正确

## 下一步

### 扩展功能

尝试修改任务描述,添加更多功能:

```python
# src/code/advanced_calculator.py
await client.query("""
创建一个高级计算器应用,包含:
1. 基础运算 (+, -, *, /)
2. 科学计算 (sin, cos, log, sqrt)
3. 历史记录列表(可删除)
4. 数据导出功能(CSV)
5. 深色模式切换
""")
```

### 创建其他应用

参考计算器示例,创建其他应用:

**待办事项管理器**:

```bash
# 1. 创建 AIPEXBASE 项目
python src/aipexbase.py "待办事项项目"

# 2. 更新 MCP URL

# 3. 创建 src/code/todo.py
# 4. 创建 src/apps/todo.py
# 5. 运行
python src/apps/todo.py
```

**博客系统**:

```bash
python src/aipexbase.py "博客项目"
# 创建 blog.py,实现文章发布和评论功能
```

### 学习资源

- [PROMPT_GUIDE.md](PROMPT_GUIDE.md) - 深入了解 prompt.py
- [MCP_INTEGRATION_GUIDE.md](MCP_INTEGRATION_GUIDE.md) - MCP 服务器详解
- [architecture.md](architecture.md) - 系统架构设计
- [CLAUDE.md](../CLAUDE.md) - 开发指南

## 总结

恭喜!你已经完成了第一个全栈应用的开发:

✅ 环境搭建和配置
✅ AIPEXBASE 项目创建
✅ E2B Template 构建
✅ Agent 脚本理解
✅ 应用运行和测试

**关键收获**:

1. **三层执行模型**: 宿主机 → Sandbox → AIPEXBASE
2. **MCP 工具集成**: 让 Agent 能操作数据库
3. **prompt.py 重要性**: 提供开发规范
4. **自动化流程**: 从代码到数据库到前端,全自动生成

**继续探索**:

- 尝试修改提示词,定制不同类型的应用
- 研究生成的代码,理解 AIPEXBASE API 调用
- 阅读其他文档,深入理解系统架构

Happy Coding! 🎉
