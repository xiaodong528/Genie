# prompt.py 系统提示词使用指南

## 概述

`src/prompt.py` 是 AIPEXBASE 应用开发的系统提示词配置文件,包含 219 行的详细开发规范,用于指导 Claude Agent 生成符合 AIPEXBASE 标准的全栈应用。

## 为什么需要 prompt.py？

### 问题

当 AI Agent 接收到"创建一个计算器应用"这样的任务时,它需要知道:
- 如何设计数据库表结构？
- 使用什么格式的 JSON Schema？
- 哪些字段类型可用？
- 如何调用 AIPEXBASE 后端 API？
- 如何使用 MCP 工具操作数据库？

### 解决方案

`prompt.py` 提供了完整的开发规范作为系统提示词,让 AI Agent 获得 AIPEXBASE 应用开发能力。

## 核心内容

### 1. 数据库设计规范

#### JSON Schema 格式

```json
[
  {
    "tableName": "users",
    "description": "用户表",
    "columns": [
      {
        "tableName": "users",
        "columnName": "id",
        "columnComment": "用户ID",
        "columnType": "bigint",
        "dslType": "Long",
        "defaultValue": null,
        "isPrimary": true,
        "isNullable": false,
        "referenceTableName": null
      },
      {
        "tableName": "users",
        "columnName": "username",
        "columnComment": "用户名",
        "columnType": "varchar(50)",
        "dslType": "String",
        "defaultValue": "",
        "isPrimary": false,
        "isNullable": false,
        "referenceTableName": null
      }
    ]
  }
]
```

#### 字段说明

| 字段名 | 类型 | 说明 | 示例 |
|--------|------|------|------|
| `tableName` | string | 表名 | "users" |
| `description` | string | 表描述 | "用户表" |
| `columnName` | string | 字段名 | "username" |
| `columnComment` | string | 字段说明 | "用户名" |
| `columnType` | string | MySQL 字段类型 | "varchar(50)" |
| `dslType` | string | DSL 类型 (见下节) | "String" |
| `defaultValue` | any | 默认值 | "" 或 null |
| `isPrimary` | boolean | 是否主键 | true/false |
| `isNullable` | boolean | 是否可为空 | true/false |
| `referenceTableName` | string | 外键表名 | "users" 或 null |

### 2. DSL 类型系统

DSL (Domain Specific Language) 类型定义了字段的语义和前端展示方式。

#### 基础类型

| DSL 类型 | 说明 | MySQL 类型 | 使用场景 |
|----------|------|-----------|----------|
| `number` | 整数 | int/bigint | 计数、数量 |
| `double` | 小数 | double | 评分、分数 |
| `decimal` | 高精度小数 | decimal | 金额、价格 |
| `String` | 文本 | varchar | 姓名、标题 (≤512字符) |
| `keyword` | 关键字 | varchar | 分类、状态码 (≤256字符) |
| `longtext` | 长文本 | longtext | 文章内容、描述 |

**关键区别: String vs keyword**

```python
# String: 模糊查询,支持 LIKE %query%
{
    "columnName": "title",
    "dslType": "String",  # 适用于: 标题、姓名、地址
    # 查询方式: WHERE title LIKE '%计算器%'
}

# keyword: 精确匹配,不支持模糊查询,查询效率更高
{
    "columnName": "status",
    "dslType": "keyword",  # 适用于: 状态码、分类、SN码
    # 查询方式: WHERE status = 'active'
}
```

#### 日期时间类型

| DSL 类型 | 格式 | MySQL 类型 | 说明 |
|----------|------|-----------|------|
| `date` | YYYY-MM-DD | date | 出生日期、生效日期 |
| `datetime` | YYYY-MM-DD HH:mm:ss | datetime | 创建时间、更新时间 |
| `time` | HH:mm:ss | time | 营业时间、课程时间 |

**最佳实践**: `datetime` 字段建议设置为不可为空,系统自动填充当前时间:
```json
{
    "columnName": "created_at",
    "columnType": "datetime",
    "dslType": "datetime",
    "isNullable": false,
    "defaultValue": null  // 系统自动设置为 CURRENT_TIMESTAMP
}
```

#### 特殊类型

| DSL 类型 | 验证规则 | 示例 |
|----------|---------|------|
| `password` | 自动加密 | 用户密码 |
| `phone` | 手机号格式验证 | 13800138000 |
| `email` | 邮箱格式验证 | user@example.com |

#### 文件类型

| DSL 类型 | 说明 | 配置参数 |
|----------|------|---------|
| `images` | 图片列表 | max_size, min_size |
| `videos` | 视频列表 | max_size, min_size |
| `files` | 文件列表 | max_size, min_size (不支持图片/视频) |

**示例**:
```json
{
    "columnName": "attachments",
    "dslType": "files",
    "max_size": 5,  // 最多 5 个文件
    "min_size": 1   // 至少 1 个文件
}
```

### 3. aipexbase-js SDK 使用指南

`prompt.py` 包含了完整的前端 SDK 使用说明,指导 AI 如何生成前端代码。

#### 初始化

```javascript
import AIPEXBASEClient from 'aipexbase-js';

const client = new AIPEXBASEClient({
    baseURL: 'http://server:port',
    token: 'your_api_token'
});
```

#### CRUD 操作示例

```javascript
// 创建记录
await client.create('users', {
    username: '张三',
    email: 'zhang@example.com'
});

// 查询记录
const users = await client.list('users', {
    filter: { status: 'active' },
    page: 1,
    pageSize: 10
});

// 更新记录
await client.update('users', id, {
    email: 'newemail@example.com'
});

// 删除记录
await client.delete('users', id);
```

### 4. MCP 工具调用说明

`prompt.py` 还包含了 MCP 工具的使用方法,指导 AI 如何操作数据库。

#### execute_sql

```json
{
    "tool": "execute_sql",
    "parameters": {
        "sql": "CREATE TABLE IF NOT EXISTS tasks (...)",
        "project_id": "123"
    }
}
```

#### list_tables

```json
{
    "tool": "list_tables",
    "parameters": {
        "project_id": "123"
    }
}
```

#### list_dynamic_api

```json
{
    "tool": "list_dynamic_api",
    "parameters": {
        "project_id": "123"
    }
}
```

## 使用方式

### 基础用法

```python
# examples/codes/my_app.py
from claude_agent_sdk import ClaudeSDKClient, ClaudeAgentOptions
from prompt import append_prompt  # 导入系统提示词

async def main():
    options = ClaudeAgentOptions(
        allowed_tools=["Bash", "Read", "Write", "execute_sql", "list_tables"],
        system_prompt={
            "type": "preset",
            "preset": "claude_code",
            "append": append_prompt  # 嵌入 AIPEXBASE 开发规范
        }
    )

    async with ClaudeSDKClient(options) as client:
        await client.query("创建一个学生管理系统")
        async for message in client.receive_response():
            print(message)
```

### 效果对比

#### ❌ 不使用 prompt.py

```python
options = ClaudeAgentOptions(
    system_prompt={"type": "preset", "preset": "claude_code"}
)

# AI 的响应:
# "I'll create a student management system with a database..."
# ❌ 不知道使用什么 JSON Schema 格式
# ❌ 不知道如何调用 AIPEXBASE API
# ❌ 可能使用其他数据库或自建后端
```

#### ✅ 使用 prompt.py

```python
from prompt import append_prompt

options = ClaudeAgentOptions(
    system_prompt={
        "type": "preset",
        "preset": "claude_code",
        "append": append_prompt
    }
)

# AI 的响应:
# "我将使用 AIPEXBASE 后端创建学生管理系统..."
# ✅ 自动使用正确的 JSON Schema 格式
# ✅ 调用 execute_sql 创建数据库表
# ✅ 使用 aipexbase-js SDK 编写前端
# ✅ 符合所有 AIPEXBASE 规范
```

## 自定义提示词

### 场景 1: 电商应用

```python
# src/custom_prompts/ecommerce_prompt.py
append_prompt = """
【电商应用开发规范】

1. 必需数据表:
   - products (商品表)
   - orders (订单表)
   - users (用户表)
   - cart_items (购物车表)

2. 字段类型规范:
   - 价格字段: 必须使用 decimal 类型 (高精度)
   - 订单状态: 使用 keyword 类型 (pending/paid/shipped/completed)
   - 商品图片: 使用 images 类型,max_size=10

3. 业务规则:
   - 订单金额 = 商品单价 × 数量
   - 支持优惠券和折扣计算
   - 库存数量实时更新

4. 前端要求:
   - 购物车实时计算总价
   - 订单列表支持状态筛选
   - 商品列表支持分类和搜索
"""
```

使用:
```python
from custom_prompts.ecommerce_prompt import append_prompt

options = ClaudeAgentOptions(
    system_prompt={
        "type": "preset",
        "preset": "claude_code",
        "append": append_prompt  # 使用电商专用提示词
    }
)
```

### 场景 2: 内容管理系统

```python
# src/custom_prompts/cms_prompt.py
append_prompt = """
【内容管理系统开发规范】

1. 必需数据表:
   - articles (文章表)
   - categories (分类表)
   - tags (标签表)
   - users (用户表)

2. 字段类型规范:
   - 文章内容: 使用 longtext 类型
   - 文章状态: 使用 keyword 类型 (draft/published/archived)
   - 发布时间: 使用 datetime 类型
   - 封面图片: 使用 images 类型,max_size=1

3. 业务规则:
   - 文章支持多分类和多标签
   - 支持草稿保存和定时发布
   - 富文本编辑器支持图片上传

4. 权限控制:
   - 管理员: 所有权限
   - 编辑: 编辑和发布文章
   - 作者: 仅编辑自己的文章
"""
```

### 场景 3: 数据分析应用

```python
# src/custom_prompts/analytics_prompt.py
append_prompt = """
【数据分析应用开发规范】

1. 数据表设计:
   - 强调数据记录表 (支持时间序列分析)
   - 聚合统计表 (提高查询性能)

2. 字段类型:
   - 数值指标: 使用 double 或 decimal
   - 时间戳: 使用 datetime,确保时区处理
   - 分类维度: 使用 keyword (便于分组统计)

3. 前端展示:
   - 使用 Chart.js 或 ECharts 图表库
   - 支持日期范围筛选
   - 支持数据导出 (CSV/Excel)

4. 性能优化:
   - 大数据量使用分页加载
   - 复杂统计使用后端聚合
   - 考虑数据缓存策略
"""
```

## 最佳实践

### 1. 提示词版本管理

```bash
src/prompts/
├── base_prompt.py          # 基础 AIPEXBASE 规范
├── ecommerce_prompt.py     # 电商应用扩展
├── cms_prompt.py           # CMS 应用扩展
├── analytics_prompt.py     # 数据分析扩展
└── __init__.py
```

### 2. 提示词组合

```python
from prompts.base_prompt import base_rules
from prompts.ecommerce_prompt import ecommerce_rules

append_prompt = f"""
{base_rules}

--- 电商应用特定规范 ---
{ecommerce_rules}
"""
```

### 3. 提示词测试

创建测试脚本验证提示词效果:

```python
# tests/test_prompt.py
import asyncio
from claude_agent_sdk import ClaudeSDKClient, ClaudeAgentOptions
from prompt import append_prompt

async def test_database_design():
    """测试 AI 是否正确设计数据库"""
    options = ClaudeAgentOptions(
        system_prompt={
            "type": "preset",
            "preset": "claude_code",
            "append": append_prompt
        }
    )

    async with ClaudeSDKClient(options) as client:
        await client.query("设计一个用户表,包含用户名、邮箱、密码")
        async for message in client.receive_response():
            # 验证输出是否包含正确的 JSON Schema
            assert "tableName" in message
            assert "dslType" in message
            print("✅ 数据库设计测试通过")

asyncio.run(test_database_design())
```

## 常见问题

### Q1: 为什么 AI 生成的数据库格式不对？

**A**: 检查是否正确导入并使用了 `prompt.py`:

```python
# ❌ 错误
from prompt import append_prompt  # 导入了
options = ClaudeAgentOptions(
    system_prompt={"type": "preset", "preset": "claude_code"}
    # ❌ 但没有使用!
)

# ✅ 正确
from prompt import append_prompt
options = ClaudeAgentOptions(
    system_prompt={
        "type": "preset",
        "preset": "claude_code",
        "append": append_prompt  # ✅ 正确使用
    }
)
```

### Q2: 如何验证提示词是否生效？

**A**: 查看 AI 的响应中是否提到了 AIPEXBASE 相关的术语:

```python
# 生效的标志:
# - "我将使用 AIPEXBASE 后端..."
# - "设计数据库表结构 JSON..."
# - "调用 execute_sql 工具..."
# - "使用 aipexbase-js SDK..."

# 未生效的标志:
# - "I'll create a database with..."
# - "Let me set up a backend server..."
# - 没有提到 AIPEXBASE 或 MCP 工具
```

### Q3: 可以同时使用多个提示词吗？

**A**: 可以,通过字符串拼接:

```python
from prompt import append_prompt as base_prompt
from custom_prompts.ecommerce_prompt import append_prompt as ecommerce_prompt

combined_prompt = f"{base_prompt}\n\n{ecommerce_prompt}"

options = ClaudeAgentOptions(
    system_prompt={
        "type": "preset",
        "preset": "claude_code",
        "append": combined_prompt
    }
)
```

### Q4: prompt.py 太长,会影响性能吗？

**A**:
- `prompt.py` 约 219 行,对于 Claude 的 200K token 上下文来说很小
- 系统提示词只在会话开始时加载一次
- 性能影响可忽略不计
- 带来的规范性和准确性提升远大于性能开销

## 总结

`prompt.py` 是 AIPEXBASE 应用开发的核心配置文件,通过提供详细的开发规范,让 AI Agent 能够:

1. ✅ 自动设计符合规范的数据库结构
2. ✅ 使用正确的字段类型和验证规则
3. ✅ 调用 MCP 工具操作数据库
4. ✅ 生成使用 aipexbase-js SDK 的前端代码
5. ✅ 遵循最佳实践和性能优化建议

**关键要点**:
- 必须在 `ClaudeAgentOptions` 的 `system_prompt.append` 中使用
- 可以针对不同应用场景自定义提示词
- 通过测试脚本验证提示词效果
- 使用版本管理和组合策略管理多个提示词

## 参考资源

- [AIPEXBASE 官方文档](docs/AIPEXBASE_PYTHON_MODULE_GUIDE.md)
- [MCP 集成指南](docs/MCP_INTEGRATION_GUIDE.md)
- [全栈开发快速入门](docs/QUICKSTART_FULL_STACK.md)
- [系统架构设计](docs/architecture.md)
