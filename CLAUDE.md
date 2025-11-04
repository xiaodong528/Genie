# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

AgentBox 是一个 E2B Sandbox + Claude Agent SDK 集成项目,实现在隔离环境中安全执行 AI 代码生成任务。

**核心架构模式:**

- **宿主机代码** (`examples/apps/`, `src/agent_runner.py`, `src/sandbox_manager.py`): 管理 Sandbox 生命周期
- **Sandbox 代码** (`examples/codes/`): 在隔离环境中执行的 AI Agent 脚本
- **Template 定义** (`src/template.py`): 使用 Python API 定义 Sandbox 环境,替代 Dockerfile
- **AIPEXBASE 模块** (`src/aipexbase.py`): AIPEXBASE MCP 服务器管理工具

## 开发环境设置

### 依赖安装

```bash
pip install -r requirements.txt
```

### 环境配置

1. 复制环境变量模板: `cp .env.example .env`
2. 必需配置:
   - `E2B_API_KEY`: 从 https://e2b.dev/dashboard 获取
   - `ANTHROPIC_AUTH_TOKEN`: Claude API 密钥
3. 可选配置 (AIPEXBASE MCP 服务器):
   - `AIPEXBASE_BASE_URL`: AIPEXBASE 服务器地址 (如 http://your-server:8080)
   - `AIPEXBASE_ADMIN_EMAIL`: 管理员邮箱
   - `AIPEXBASE_ADMIN_PASSWORD`: 管理员密码
   - 其他 AIPEXBASE 相关配置见 `.env.example`

**AIPEXBASE 配置说明**:
AIPEXBASE 是低代码后端平台,为 AI Agent 提供:

- 数据库自动管理 (MySQL)
- RESTful API 自动生成
- MCP 服务器集成

如果您不使用 AIPEXBASE,可以跳过这部分配置,但需要修改示例代码中的 MCP 相关部分。

### Template 构建

```bash
# 首次使用或修改 template.py 后执行
python src/build_template.py

# 成功后会生成 .template_id 文件,包含 Template ID
```

## 核心命令

### 运行应用

```bash
# 运行计算器应用生成器 (Web 服务模式)
python examples/apps/calculator.py

# 直接运行 Agent Runner (自动清理模式)
python src/agent_runner.py

# 使用 AIPEXBASE 模块管理 MCP 服务器
python src/aipexbase.py
```

### 测试

```bash
# 运行所有测试
pytest tests/

# 运行单个测试文件
pytest tests/test_sandbox.py
pytest tests/test_agent_runner.py

# 运行特定测试
pytest tests/test_agent_runner.py::test_function_name

# 带超时控制的测试
pytest tests/ --timeout=300
```

### 代码检查

```bash
# 格式检查 (如果项目配置了)
# flake8 src/ tests/

# 类型检查 (如果项目配置了)
# mypy src/
```

## 架构理解

### 三层执行模型

1. **应用层** (`examples/apps/`):

   - 在宿主机执行
   - 调用 `agent_runner.py` 启动 Sandbox
   - 处理结果和错误
2. **运行器层** (`src/agent_runner.py`, `src/sandbox_manager.py`):

   - 管理 Sandbox 生命周期
   - 上传 `examples/codes/` 中的脚本到 Sandbox
   - 捕获执行结果
3. **Agent 层** (`examples/codes/`):

   - 在 Sandbox 内执行
   - 使用 Claude Agent SDK 生成代码
   - 通过工具调用 (Bash/Read/Write) 创建文件

### 两种运行模式

**自动清理模式:**

```python
# 执行完成后自动关闭 Sandbox
result = await run_code_in_sandbox("script.py")
```

**服务模式:**

```python
# Sandbox 保持运行,返回 Web 服务 URL
result = await run_code_with_service("script.py", service_port=3000)
print(result['service_url'])  # https://xxx.e2b.dev
```

### Template vs Sandbox

- **Template** (`src/template.py`):

  - 环境定义 (类似 Docker 镜像)
  - 使用 Python API 定义,支持动态配置
  - 修改后需要运行 `build_template.py` 重新构建
- **Sandbox** (运行时实例):

  - 基于 Template 创建的隔离环境
  - 每次执行创建新实例
  - 默认超时 3600 秒 (1 小时)

## 文件组织规则

### 添加新的 Agent 脚本

1. 在 `examples/codes/` 创建脚本 (Sandbox 内执行):

```python
# examples/codes/my_agent.py
from claude_agent_sdk import ClaudeSDKClient, ClaudeAgentOptions

async def main():
    options = ClaudeAgentOptions(
        allowed_tools=["Bash", "Read", "Write"],
        permission_mode="bypassPermissions"
    )

    async with ClaudeSDKClient(options) as client:
        await client.query("你的任务描述")
        async for message in client.receive_response():
            print(message)

if __name__ == "__main__":
    import asyncio
    asyncio.run(main())
```

2. 在 `examples/apps/` 创建运行器 (宿主机执行):

```python
# examples/apps/my_agent.py
import asyncio
from agent_runner import run_code_in_sandbox

async def main():
    result = await run_code_in_sandbox("my_agent.py")
    print(f"Exit code: {result['exit_code']}")

if __name__ == "__main__":
    asyncio.run(main())
```

### 不要在错误的地方执行代码

- ❌ 不要在 `examples/codes/` 中导入 `agent_runner` 或 `sandbox_manager` (这些在 Sandbox 中不可用)
- ❌ 不要在 `examples/apps/` 中使用 Claude Agent SDK (应该在 `examples/codes/` 中使用)
- ✅ 理解边界: 宿主机代码管理 Sandbox,Sandbox 代码运行 Agent

## 环境变量管理

### Template 环境变量

在 `template.py` 中通过 `.set_envs()` 设置:

```python
.set_envs({
    "ANTHROPIC_AUTH_TOKEN": os.getenv("ANTHROPIC_AUTH_TOKEN"),
    "WORKSPACE_DIR": "/home/user"
})
```

### Runtime 环境变量

在 `agent_runner.py` 中传递:

```python
await run_code_in_sandbox(
    "script.py",
    env_vars={"CUSTOM_VAR": "value"}  # 覆盖 Template 默认值
)
```

## 常见开发任务

### 修改 Sandbox 环境

1. 编辑 `src/template.py`
2. 运行 `python src/build_template.py`
3. 新的 Template ID 自动保存到 `.template_id`

### 配置 MCP 服务器

在 Sandbox 中配置 MCP 服务器 (如 AIPEXBASE):

**方式 1: 在代码中自动配置** (推荐)

```python
# 在 agent_runner.py 的 run_code_with_service 中已集成
# 会自动执行: claude mcp add --transport sse --scope user ...
```

**方式 2: 手动配置**

```bash
# 在 Sandbox 中执行
claude mcp add --transport sse --scope user aipexbase-mcp-server "http://server:port/mcp/sse?token=xxx"
claude mcp list  # 验证配置
```

**方式 3: 使用 aipexbase.py 模块**

```bash
# 自动从 .env 读取配置并生成 MCP 配置
python src/aipexbase.py
```

### 调试 Agent 执行

查看 `agent_runner.py` 中的实时输出回调:

```python
result = await sandbox.commands.run(
    cmd=command,
    on_stdout=lambda msg: print(f"[STDOUT] {msg}"),
    on_stderr=lambda msg: print(f"[STDERR] {msg}")
)
```

### 下载 Sandbox 生成的文件

```python
async with SandboxManager(template_id) as manager:
    content = await manager.sandbox.files.read("/home/user/output.txt")
    with open("local_output.txt", "w") as f:
        f.write(content)
```

### 调整 Sandbox 超时

```python
# 在 sandbox_manager.py 中修改
self.sandbox = await AsyncSandbox.create(
    template=self.template_id,
    timeout=7200  # 2 小时
)
```

### 使用 aipexbase.py 创建项目

```bash
# 自动从 .env 读取配置并创建 AIPEXBASE 项目
python src/aipexbase.py "我的项目名称"

# 输出示例:
# ✅ 项目创建成功
# 📋 项目信息:
#    项目 ID: 123
#    项目名称: 我的项目名称
# 🔌 MCP URL: http://server:8080/mcp/sse?token=xxx
```

获取 MCP URL 后,需要更新 `src/agent_runner.py` 第 247 行。

## 重要约定

1. **Template ID 管理**:

   - 存储在 `.template_id` 文件 (gitignored)
   - 构建后自动生成,不要手动修改
2. **异步模式**:

   - 所有 Sandbox 操作都是异步的
   - 使用 `async/await` 和 `asyncio.run()`
3. **资源清理**:

   - 使用 `async with SandboxManager()` 确保 Sandbox 自动关闭
   - 或手动调用 `await manager.close()`
4. **工作目录**:

   - Sandbox 内默认工作目录: `/home/user`
   - 上传的脚本路径: `/home/user/script.py`
5. **MCP 服务器配置**:

   - MCP 配置在 `run_code_with_service()` 中自动完成
   - Token 需要在 agent_runner.py 中硬编码或通过环境变量传递
   - 使用 `claude mcp list` 验证配置是否成功

## 故障排查

### Template 构建失败

- 检查 `E2B_API_KEY` 是否正确配置
- 查看构建日志中的具体错误信息

### Sandbox 创建失败

- 确认 `.template_id` 文件存在
- 验证 Template ID 格式正确

### Agent 执行超时

- 增加 Sandbox timeout 参数
- 检查 Agent 脚本中的无限循环

### 找不到生成的文件

- 使用 `await sandbox.files.list("/home/user")` 列出所有文件
- 确认文件路径使用绝对路径

## 开发全栈应用完整流程

本节说明如何从零开始开发一个完整的全栈应用（以计算器为例）。

### 步骤 1: 环境准备

```bash
# 1. 配置环境变量
cp .env.example .env
# 编辑 .env,填入:
#   - E2B_API_KEY
#   - ANTHROPIC_AUTH_TOKEN
#   - AIPEXBASE_BASE_URL
#   - AIPEXBASE_ADMIN_EMAIL
#   - AIPEXBASE_ADMIN_PASSWORD

# 2. 安装依赖
pip install -r requirements.txt
```

### 步骤 2: 创建 AIPEXBASE 项目

```bash
# 使用 aipexbase.py 自动创建
python src/aipexbase.py "计算器项目"

# 记录输出中的 MCP URL:
# 🔌 MCP URL: http://39.106.253.201:8081/mcp/sse?token=kf_api_xxx
```

### 步骤 3: 配置 MCP 服务器

编辑 `src/agent_runner.py`,更新第 247 行的 MCP URL:

```python
# src/agent_runner.py (行 247)
mcp_cmd = (
    'claude mcp add --transport sse --scope user aipexbase-mcp-server '
    '"http://39.106.253.201:8081/mcp/sse?token=kf_api_xxx"'  # 替换为你的 MCP URL
)
```

### 步骤 4: 编写 Agent 脚本

创建 `examples/codes/my_app.py`:

```python
"""我的应用生成器 - 在 E2B Sandbox 中执行"""
import asyncio
from claude_agent_sdk import ClaudeSDKClient, ClaudeAgentOptions
from prompt import append_prompt  # 导入系统提示词

async def main():
    """主函数"""
    options = ClaudeAgentOptions(
        allowed_tools=[
            "Bash",
            "Read",
            "Write",
            "Glob",
            "Grep",
            "execute_sql",         # AIPEXBASE MCP 工具
            "list_tables",         # AIPEXBASE MCP 工具
            "list_dynamic_api"     # AIPEXBASE MCP 工具
        ],
        permission_mode="bypassPermissions",
        cwd="/home/user",
        system_prompt={
            "type": "preset",
            "preset": "claude_code",
            "append": append_prompt  # 嵌入 AIPEXBASE 开发规范
        }
    )

    async with ClaudeSDKClient(options) as client:
        # 发送任务
        await client.query(
            "创建一个待办事项管理应用,包含:\n"
            "1. 数据库表设计(任务表)\n"
            "2. Web 前端界面\n"
            "3. 使用 AIPEXBASE API 进行 CRUD 操作\n"
            "4. 启动 HTTP 服务器在 3000 端口"
        )
      
        # 接收响应
        async for message in client.receive_response():
            print(message)

if __name__ == "__main__":
    asyncio.run(main())
```

**关键点**:

- **必须导入 `prompt.py`**: 提供 AIPEXBASE 开发规范
- **启用 MCP 工具**: `execute_sql`, `list_tables`, `list_dynamic_api`
- **设置 system_prompt**: 嵌入 `append_prompt` 指导 AI

### 步骤 5: 编写应用运行器

创建 `examples/apps/my_app.py`:

```python
"""我的应用运行器 - 在宿主机执行"""
import asyncio
import os
from agent_runner import run_code_with_service

async def main():
    """运行我的应用生成器"""
    print("🚀 我的应用生成器")
    print("=" * 60)
  
    # 运行 Agent 脚本并启动服务
    result = await run_code_with_service(
        code_file="my_app.py",  # 对应 examples/codes/my_app.py
        service_port=3000,
        env_vars={
            "ANTHROPIC_AUTH_TOKEN": os.getenv("ANTHROPIC_AUTH_TOKEN")
        },
        wait_time=5  # 等待服务启动时间
    )
  
    # 输出结果
    print("\n" + "=" * 60)
    print("🌐 Web 服务信息")
    print("=" * 60)
    print(f"✅ 前端地址: {result['service_url']}")
    print(f"✅ Sandbox ID: {result['sandbox_id']}")
    print(f"✅ 生成的文件: {', '.join(result['files'])}")
    print("\n💡 使用提示:")
    print("  1. 在浏览器中打开上述地址访问应用")
    print("  2. Sandbox 将保持运行约 1 小时（3600 秒）")
    print("  3. 服务超时后会自动关闭")
    print("=" * 60)

if __name__ == "__main__":
    asyncio.run(main())
```

### 步骤 6: 构建 Template (首次或修改后)

```bash
# 如果这是首次运行或修改了 template.py
python src/build_template.py

# 成功后会生成 .template_id 文件
```

### 步骤 7: 运行应用

```bash
# 运行应用运行器
python examples/apps/my_app.py

# 预期输出:
# 🚀 我的应用生成器
# 📋 读取 Template ID...
# ✅ Template ID: xxx
# 🚀 创建 Sandbox...
# 🔧 配置 MCP 服务器...
# ✅ MCP 服务器配置成功
# 🚀 执行代码...
# [Agent] 正在设计数据库...
# [Agent] 创建任务表...
# [Agent] 生成前端代码...
# [Agent] 启动 HTTP 服务...
# ✅ 服务 URL: https://xxx.e2b.dev
```

### 步骤 8: 访问和测试

1. 打开浏览器访问输出的服务 URL
2. 测试应用功能
3. 查看数据库数据(通过 AIPEXBASE 后台)

### 关键概念理解

#### prompt.py 的重要性

**`prompt.py` 是什么?**

- 219 行的系统提示词配置文件
- 包含 AIPEXBASE 应用开发的完整规范
- 指导 AI Agent 如何设计数据库、使用 SDK、调用 MCP 工具

**为什么必须使用?**

```python
# ❌ 错误: 不使用 prompt.py
options = ClaudeAgentOptions(
    system_prompt={"type": "preset", "preset": "claude_code"}
    # AI 不知道如何使用 AIPEXBASE
)

# ✅ 正确: 嵌入 prompt.py
from prompt import append_prompt

options = ClaudeAgentOptions(
    system_prompt={
        "type": "preset",
        "preset": "claude_code",
        "append": append_prompt  # AI 获得 AIPEXBASE 开发能力
    }
)
```

**包含的关键内容**:

1. **数据库设计规范**: MySQL 表结构 JSON 格式
2. **DSL 类型系统**: number/string/keyword/datetime/phone/email 等
3. **aipexbase-js SDK**: 前端开发指南
4. **MCP 工具使用**: execute_sql/list_tables/list_dynamic_api 调用方法

**自定义提示词**:
如果开发不同类型的应用,可以修改 `prompt.py`:

```python
# 示例: 为电商应用定制
append_prompt = """
【电商应用开发规范】
- 必须包含商品表、订单表、用户表
- 支付相关字段使用 decimal 类型
- 订单状态使用 keyword 类型
...
"""
```

#### 三层执行模型理解

```
宿主机层 (apps/*.py)
    ↓ 调用
运行器层 (agent_runner.py + sandbox_manager.py)
    ↓ 创建 Sandbox 并上传脚本
Agent 层 (code/*.py)
    ↓ 读取 prompt.py
    ↓ 调用 Claude Agent SDK
    ↓ 使用 MCP 工具
AIPEXBASE 后端层
    ↓ 数据库操作
MySQL 数据库
```

**执行顺序**:

1. 用户运行 `python examples/apps/my_app.py` (宿主机)
2. `agent_runner.py` 创建 E2B Sandbox
3. 上传 `code/my_app.py` 到 Sandbox
4. Sandbox 中配置 MCP 服务器
5. 执行 `code/my_app.py` (Sandbox 内)
6. Agent 读取 `prompt.py` 获取开发规范
7. Agent 调用 MCP 工具创建数据库表
8. Agent 生成前端代码
9. Agent 启动 HTTP 服务器
10. 返回服务 URL 给用户

### 故障排查技巧

#### 问题 1: MCP 服务器配置失败

```
[MCP Error] Failed to add MCP server
```

**解决方法**:

1. 检查 `agent_runner.py` 第 247 行的 MCP URL 是否正确
2. 验证 Token 是否有效
3. 确认 AIPEXBASE 服务器可访问

#### 问题 2: Agent 不知道如何使用 AIPEXBASE

```
[Agent] I don't know how to create a database
```

**解决方法**:
确保在 `code/*.py` 中导入并使用 `prompt.py`:

```python
from prompt import append_prompt

options = ClaudeAgentOptions(
    system_prompt={
        "type": "preset",
        "preset": "claude_code",
        "append": append_prompt  # 关键!
    }
)
```

#### 问题 3: 生成的代码无法连接后端

**解决方法**:

1. 检查前端代码是否使用了 `aipexbase-js` SDK
2. 确认 API 端点 URL 正确
3. 查看 AIPEXBASE 后台的 API 列表

## 相关文档

- `docs/architecture.md`: 完整系统架构设计
- `docs/AIPEXBASE_PYTHON_MODULE_GUIDE.md`: AIPEXBASE 模块使用指南
- `README.md`: 项目快速入门和示例
