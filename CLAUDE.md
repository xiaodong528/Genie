# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

Genie 是一个 E2B Sandbox + Claude Agent SDK 集成项目,在隔离环境中安全执行 AI 代码生成任务。

**核心架构 - 三层执行模型:**

```
宿主机层          → examples/apps/*.py (管理 Sandbox 生命周期)
  ↓
运行器层          → src/agent_runner.py + src/sandbox_manager.py (创建/管理 Sandbox)
  ↓
Agent 层 (Sandbox) → examples/codes/*.py (运行 Claude Agent SDK)
  ↓
AIPEXBASE 后端    → MCP 服务器 (数据库操作/API 生成)
```

**关键概念:**
- **Template** (`src/templates/sandbox_claude_template.py`): 使用 Python API 定义 Sandbox 环境(替代 Dockerfile)
- **Sandbox**: 基于 Template 创建的隔离运行时环境,默认超时 3600 秒
- **MCP 服务器**: 为 Agent 提供数据库操作工具(`execute_sql`, `list_tables`, `list_dynamic_api`)
- **prompt.py**: 219 行系统提示词,指导 AI Agent 如何使用 AIPEXBASE 开发全栈应用

## 开发环境设置

### 首次设置

```bash
# 1. 安装依赖
pip install -r requirements.txt

# 2. 配置环境变量
cp .env.example .env
# 必需: E2B_API_KEY, ANTHROPIC_AUTH_TOKEN
# 可选: AIPEXBASE_BASE_URL, AIPEXBASE_ADMIN_EMAIL, AIPEXBASE_ADMIN_PASSWORD

# 3. 构建 E2B Template (仅首次或修改 template.py 后)
python scripts/build_sandbox_claude_template.py
# 成功后生成 .template_id 文件
```

### 配置 AIPEXBASE MCP 服务器

```bash
# 1. 创建 AIPEXBASE 项目并获取 MCP URL
python src/aipexbase.py "项目名称"
# 输出: 🔌 MCP URL: http://server:port/mcp/sse?token=xxx

# 2. 更新 MCP URL 到代码中
# 编辑 src/agent_runner.py 第 247 行,替换为实际 MCP URL
```

## 核心开发命令

### 运行应用

```bash
# Web 服务模式 (Sandbox 保持运行,返回 URL)
python examples/apps/calculator.py

# 自动清理模式 (执行完成后关闭 Sandbox)
python src/agent_runner.py
```

### 测试

```bash
# 运行所有测试
pytest tests/

# 运行单个测试文件
pytest tests/test_sandbox.py

# 运行特定测试
pytest tests/test_agent_runner.py::test_function_name

# 带超时控制
pytest tests/ --timeout=300
```

### 重新构建 Template

```bash
# 何时需要重建?
# - 修改了 src/templates/sandbox_claude_template.py
# - 需要更新 Sandbox 环境的依赖或环境变量

python scripts/build_sandbox_claude_template.py
```

## 文件组织和职责边界

### 关键决策树

**Q: 我的代码应该放在哪里?**

```
代码需要访问 agent_runner/sandbox_manager?
├─ Yes → 放在 examples/apps/ (宿主机执行)
└─ No  → 代码需要使用 Claude Agent SDK?
         ├─ Yes → 放在 examples/codes/ (Sandbox 内执行)
         └─ No  → 放在 src/ (工具/管理代码)
```

**Q: 何时需要重新构建 Template?**

```
修改了什么?
├─ src/templates/sandbox_claude_template.py → 需要重建
├─ examples/codes/*.py                      → 不需要 (每次上传)
├─ examples/apps/*.py                       → 不需要 (宿主机运行)
└─ src/agent_runner.py                      → 不需要 (宿主机运行)
```

**Q: 使用哪种运行模式?**

```
需要 Web 服务访问?
├─ Yes → run_code_with_service() (返回 https://xxx.e2b.dev)
└─ No  → run_code_in_sandbox()   (自动清理)
```

### 不要在错误的地方执行代码

**❌ 常见错误:**
- 在 `examples/codes/` 中导入 `agent_runner` (Sandbox 中不可用)
- 在 `examples/apps/` 中使用 Claude Agent SDK (应在 Sandbox 内使用)
- 在 `examples/codes/` 中忘记导入 `prompt.py` (AI 不知道如何使用 AIPEXBASE)

**✅ 正确模式:**
```python
# examples/apps/my_app.py (宿主机)
from agent_runner import run_code_with_service

# examples/codes/my_agent.py (Sandbox)
from claude_agent_sdk import ClaudeSDKClient
from prompt import append_prompt  # 必需!
```

## 添加新应用的标准流程

### 1. 创建 Agent 脚本 (Sandbox 内执行)

```python
# examples/codes/my_agent.py
from claude_agent_sdk import ClaudeSDKClient, ClaudeAgentOptions
from prompt import append_prompt  # ← 必须导入!

async def main():
    options = ClaudeAgentOptions(
        allowed_tools=[
            "Bash", "Read", "Write", "Glob", "Grep",
            "execute_sql",      # AIPEXBASE MCP 工具
            "list_tables",      # AIPEXBASE MCP 工具
            "list_dynamic_api"  # AIPEXBASE MCP 工具
        ],
        permission_mode="bypassPermissions",
        cwd="/home/user",
        system_prompt={
            "type": "preset",
            "preset": "claude_code",
            "append": append_prompt  # ← 嵌入 AIPEXBASE 开发规范
        }
    )

    async with ClaudeSDKClient(options) as client:
        await client.query("你的任务描述")
        async for message in client.receive_response():
            print(message)

if __name__ == "__main__":
    import asyncio
    asyncio.run(main())
```

### 2. 创建应用运行器 (宿主机执行)

```python
# examples/apps/my_app.py
import asyncio
import os
from agent_runner import run_code_with_service

async def main():
    result = await run_code_with_service(
        code_file="my_agent.py",  # 对应 examples/codes/my_agent.py
        service_port=3000,
        env_vars={"ANTHROPIC_AUTH_TOKEN": os.getenv("ANTHROPIC_AUTH_TOKEN")},
        wait_time=5
    )

    print(f"✅ 服务 URL: {result['service_url']}")
    print(f"✅ 生成的文件: {', '.join(result['files'])}")

if __name__ == "__main__":
    asyncio.run(main())
```

### 3. 运行应用

```bash
python examples/apps/my_app.py
```

## prompt.py 的重要性

**为什么必须使用 prompt.py?**

`prompt.py` 是 219 行的系统提示词配置,包含:
- 数据库设计规范 (MySQL 表结构 JSON Schema)
- DSL 类型系统 (number/string/keyword/datetime/phone/email 等)
- aipexbase-js SDK 使用指南
- MCP 工具调用方法

**如果不使用 prompt.py:**
```python
# ❌ AI 不知道如何使用 AIPEXBASE
options = ClaudeAgentOptions(
    system_prompt={"type": "preset", "preset": "claude_code"}
)
```

**正确使用:**
```python
# ✅ AI 获得 AIPEXBASE 开发能力
from prompt import append_prompt

options = ClaudeAgentOptions(
    system_prompt={
        "type": "preset",
        "preset": "claude_code",
        "append": append_prompt  # ← 关键!
    }
)
```

## 环境变量管理

### Template 环境变量 (构建时设置)

在 `src/templates/sandbox_claude_template.py` 中通过 `.set_envs()` 设置:

```python
.set_envs({
    "ANTHROPIC_AUTH_TOKEN": os.getenv("ANTHROPIC_AUTH_TOKEN"),
    "WORKSPACE_DIR": "/home/user"
})
```

### Runtime 环境变量 (运行时覆盖)

在调用 `run_code_in_sandbox()` 或 `run_code_with_service()` 时传递:

```python
await run_code_with_service(
    "script.py",
    env_vars={"CUSTOM_VAR": "value"}  # 覆盖 Template 默认值
)
```

## 常见开发任务

### 调试 Agent 执行

查看 `agent_runner.py` 中的实时输出:

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

在 `src/sandbox_manager.py` 中修改:

```python
self.sandbox = await AsyncSandbox.create(
    template=self.template_id,
    timeout=7200  # 2 小时 (默认 3600 秒)
)
```

### 列出 Sandbox 中的所有文件

```python
files = await sandbox.files.list("/home/user")
for file in files:
    print(file)
```

## 重要约定

1. **Template ID 管理**:
   - 存储在 `.template_id` 文件 (gitignored)
   - 构建后自动生成,不要手动修改

2. **异步模式**:
   - 所有 Sandbox 操作都是异步的
   - 使用 `async/await` 和 `asyncio.run()`

3. **资源清理**:
   - 使用 `async with SandboxManager()` 确保自动清理
   - 或手动调用 `await manager.close()`

4. **工作目录**:
   - Sandbox 内默认: `/home/user`
   - 上传的脚本: `/home/user/script.py`

5. **MCP 服务器配置**:
   - MCP URL 硬编码在 `src/agent_runner.py` 第 247 行
   - 需要手动更新为实际的 AIPEXBASE MCP URL
   - 使用 `claude mcp list` 在 Sandbox 中验证配置

## 故障排查快速参考

### Template 构建失败
- 检查 `E2B_API_KEY` 是否正确配置
- 查看构建日志中的具体错误信息

### Sandbox 创建失败
- 确认 `.template_id` 文件存在
- 验证 Template ID 格式正确

### MCP 服务器配置失败
- 检查 `src/agent_runner.py` 第 247 行的 MCP URL 是否正确
- 验证 Token 是否有效
- 确认 AIPEXBASE 服务器可访问

### Agent 不知道如何使用 AIPEXBASE
- 确保在 `examples/codes/*.py` 中导入了 `from prompt import append_prompt`
- 确保在 `ClaudeAgentOptions` 中设置了 `system_prompt.append = append_prompt`

### 找不到生成的文件
- 使用 `await sandbox.files.list("/home/user")` 列出所有文件
- 确认文件路径使用绝对路径

## 相关文档

- `docs/tech/architecture.md` - 完整系统架构设计
- `docs/llm_guide/MCP_INTEGRATION_GUIDE.md` - MCP 集成详细指南
- `docs/llm_guide/AIPEXBASE_PYTHON_MODULE_GUIDE.md` - AIPEXBASE 模块使用指南
- `README.md` - 项目快速入门
