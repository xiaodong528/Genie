# Genie v1.0.0 产品需求文档 (PRD)

---

## 📋 目录

1. [产品概述](#1-产品概述)
2. [用户故事与使用场景](#2-用户故事与使用场景)
3. [功能需求](#3-功能需求)
4. [技术架构](#4-技术架构)
5. [数据模型](#5-数据模型)
6. [通信协议规范](#6-通信协议规范)
7. [用户交互流程](#7-用户交互流程)
8. [非功能需求](#8-非功能需求)
9. [风险与限制](#9-风险与限制)
10. [实施路线图](#10-实施路线图)
11. [附录](#11-附录)

---

## 1. 产品概述

### 1.1 产品定位

**Genie** 是一个 AI 驱动的应用生成平台，用户通过自然语言描述需求，系统自动完成从需求分析、架构设计、代码生成、测试到云端部署的全流程，并返回可直接访问的应用链接。

### 1.2 核心价值主张

- **零代码启动**: 无需编写任何代码，通过对话即可生成完整应用
- **专业级质量**: 多智能体协作 + 质量评估循环，确保输出符合工程标准
- **即时部署**: 自动部署到 E2B 沙箱，生成可分享的在线链接
- **可控可回溯**: 支持中断、恢复、回滚，用户全程掌控生成过程

### 1.3 目标用户

- **开发者**: 快速生成 MVP 或原型验证想法
- **创业者**: 将商业想法快速转化为可演示的产品
- **产品经理**: 生成交互式原型进行需求验证
- **学习者**: 通过实际项目学习软件开发最佳实践

---

## 2. 用户故事与使用场景

### 2.1 用户故事

#### US-01: 快速生成 Web 应用

> **作为** 一名创业者
> **我想要** 通过描述"一个任务管理工具"就能生成可用的 Web 应用
> **以便** 快速验证产品想法，无需雇佣开发团队

**验收标准**:

- 用户输入需求后，系统自动生成 PRD 文档
- 系统根据 PRD 设计技术架构（选择框架、数据库等）
- 系统生成完整的前后端代码
- 系统自动部署到云端并返回访问链接
- 整个流程在 15 分钟内完成

#### US-02: 多轮对话澄清需求

> **作为** 一名产品经理
> **我想要** 在需求分析阶段与 AI 进行多轮对话
> **以便** 确保 PRD 文档准确反映我的产品设想

**验收标准**:

- 需求分析智能体会主动提问澄清模糊点
- 用户可以修改或补充需求描述
- 系统会生成结构化的 PRD 文档供用户确认
- 只有用户确认后才进入下一阶段

#### US-03: 中断并恢复生成过程

> **作为** 一名开发者
> **我想要** 在代码生成过程中临时中断
> **以便** 处理其他紧急事务后继续生成

**验收标准**:

- 用户可以在任意时刻点击"中断"按钮
- 系统保存当前状态并停止执行
- 用户稍后可以从中断点继续执行
- 中断前已完成的工作不会丢失

#### US-04: 回滚到上一轮会话

> **作为** 一名用户
> **我想要** 回滚到上一轮成功部署的版本
> **以便** 撤销不满意的修改重新生成

**验收标准**:

- 系统记录每轮会话的完整状态
- 用户可以查看历史版本列表
- 点击"回滚"后，代码和状态恢复到指定版本
- Messages 历史也同步回滚

#### US-05: 查看实时进度和日志

> **作为** 一名用户
> **我想要** 实时查看每个智能体的工作进度
> **以便** 了解当前生成状态和预计完成时间

**验收标准**:

- 系统通过 AG-UI 事件推送当前执行的节点名称
- 系统推送每个智能体的流式输出（思考过程、生成的代码片段）
- 系统推送 Reflect 评估的分数和反馈
- 系统推送预计剩余时间

### 2.2 典型使用场景

#### 场景 A: 从零开始生成全新应用

1. 用户输入: "我想要一个类似 Notion 的笔记应用，支持 Markdown 编辑和多人协作"
2. 需求分析智能体与用户多轮对话，确认功能细节
3. 架构师智能体设计技术栈（React + FastAPI + PostgreSQL）
4. 代码生成主智能体生成完整代码
5. 测试工程师智能体执行 E2E 测试
6. DevOps 智能体部署到 E2B 沙箱
7. 返回访问链接

#### 场景 B: 迭代优化已生成应用

1. 用户查看第一次生成的应用，发现 UI 不满意
2. 用户输入: "请将主题色改为蓝色，并增加夜间模式"
3. 系统识别这是修改请求，跳过需求分析和架构设计
4. 直接从代码生成阶段开始，修改样式代码
5. 重新测试和部署
6. 返回新版本链接

#### 场景 C: 中断后恢复

1. 用户在代码生成阶段（进度 60%）点击中断
2. 系统保存当前 checkpoint 并停止
3. 用户 1 小时后回来，点击"继续"
4. 系统从 checkpoint 恢复，继续生成剩余代码
5. 完成部署

#### 场景 D: 回滚到历史版本

1. 第一轮: 用户生成了一个博客应用（版本 v1）
2. 第二轮: 用户修改了评论功能（版本 v2）
3. 第三轮: 用户发现 v2 有 bug，点击"回滚到 v1"
4. 系统将代码回滚到 v1 的 Git commit
5. 系统将 Messages 历史截断到 v1 结束时
6. 用户可以重新修改评论功能

---

## 3. 功能需求

### 3.1 核心功能

#### F1: 自然语言需求输入

- 用户通过文本输入描述应用需求
- 支持上传参考文档（PDF、图片、URL）
- 支持语音输入（未来版本）

#### F2: 智能需求分析

- **需求分析智能体**通过多轮对话澄清需求
- 自动生成结构化 PRD 文档
- PRD 包含: 功能列表、用户故事、技术约束、非功能需求
- Reflect 节点评估 PRD 完整性（评分 > 0.7 通过）

#### F3: 自动架构设计

- **架构师智能体**根据 PRD 设计技术架构
- 输出: 技术栈选型、系统架构图、数据模型、API 设计
- 考虑因素: 性能、可扩展性、开发复杂度
- Reflect 节点评估架构合理性

#### F4: 代码生成

- **代码生成主智能体**（基于 Claude Agent SDK）生成完整项目代码
- 支持的项目类型:
  - Web 应用（React/Vue + FastAPI/Express）
  - CLI 工具（Python/Node.js）
  - API 服务（FastAPI/Express）
- 自动生成单元测试代码
- Reflect 节点评估代码质量（Linter + Type Checker）

#### F5: 自动化测试

- **测试工程师智能体**执行 E2E 测试
- 使用 Playwright MCP 工具进行浏览器自动化测试
- 测试用例自动生成（覆盖核心用户流程）
- 测试失败时生成详细报告，回退到代码生成阶段重试
- Reflect 节点评估测试覆盖率和通过率

#### F6: 云端部署

- 创建 E2B 沙箱（基于预定义模板）
- **DevOps 智能体**通过 E2B Python SDK 部署服务（配置 Claude Agent Skill）
- 自动配置环境变量、安装依赖、启动服务
- 返回可访问的 HTTPS 链接
- 沙箱过期时间: 12 小时
- Reflect 节点验证部署成功性（健康检查）

#### F7: 中断与恢复

- **中断机制**:

  - 接收客户端中断请求
  - 设置中断标志，LangGraph 在下个节点检查时停止
  - Claude Agent SDK 的 `interrupt()` 方法停止当前智能体的流式输出
  - 向客户端发送工作流完成事件（标记为中断状态）
- **恢复机制**:

  - 接收客户端恢复请求
  - 从 LangGraph checkpoint 恢复状态
  - 从中断节点继续执行

#### F8: 会话回滚

- **回滚粒度**: 会话轮次（session_round）

  - 第1轮: 初次生成
  - 第2轮: 第一次修改
  - 第3轮: 第二次修改
- **回滚机制**:

  1. 从 LangGraph checkpoint 恢复 AgentState
  2. 执行 Git 版本回滚
  3. 截断 Messages 历史到指定轮次
  4. 向客户端推送回滚后的状态信息
- **回滚限制**:

  - 如果还在需求分析阶段（未生成代码），无需回滚
  - 只能回滚到已完成部署的轮次

### 3.2 辅助功能

#### F9: 实时进度推送

- 推送当前执行节点名称（STEP_STARTED 事件）
- 推送节点进度百分比（CUSTOM 事件）
- 推送每个智能体的流式输出（TEXT_MESSAGE_CONTENT 事件）
- 推送 Reflect 评估结果（CUSTOM 事件）

#### F10: 历史版本管理

- 提供 API 获取所有会话轮次列表
- 提供每轮次的 Git commit 信息
- 支持通过 API 查询历史版本的 PRD、架构文档、代码
- 支持下载历史版本的完整项目

#### F11: 错误处理与重试

- 自动检测错误并重试（最多 3 次）
- 通过 AG-UI 事件推送详细的错误信息和调试日志
- 提供 API 支持手动重试失败的节点

---

## 4. 技术架构

### 4.1 整体架构

#### 系统分层

> **说明**: Genie 项目负责后端服务开发，前端应用由其他项目组开发。双方通过 AG-UI Protocol 标准协议进行通信对接。

```
AG-UI Protocol 通信层 (WebSocket/SSE)
    ↓
后端服务层 (Backend Server)
    ↓
工作流编排层 (LangGraph Orchestrator)
    ↓
智能体执行层 (7个专业子智能体 + 5个Reflect节点)
    ↓
基础设施层 (Git、E2B、数据库、MCP工具)
```

#### 核心组件

**后端服务**

- AG-UI Server：处理 WebSocket/SSE 连接，转换事件格式
- API 服务器：处理中断、回滚等控制请求

**工作流编排**

- LangGraph StateGraph：定义 12 个节点的工作流拓扑
- Checkpointer：状态快照机制，支持断点续传和回滚
- 条件边：Reflect 评估逻辑和回退控制

**智能体节点**

- 需求分析智能体（Claude Agent SDK 子智能体）
- 架构设计智能体（Claude Agent SDK 子智能体）
- 代码生成主智能体（Claude Agent SDK + MCP 工具）
- 测试工程师智能体（Claude Agent SDK + Playwright MCP）
- DevOps 智能体（Claude Agent SDK + E2B SDK）
- Reflect 智能体（Claude Agent SDK + Sequential MCP）

**支持服务**

- Git 版本控制：每个节点后自动 commit
- E2B Sandbox Manager：沙箱生命周期管理
- State Persistence：PostgreSQL/Redis 状态持久化

**外部服务**

- Claude Code（Agent）SDK Python
- E2B Sandbox（代码执行环境）
- Playwright MCP（浏览器自动化）
- Sequential MCP（结构化推理）

### 4.2 技术栈

#### 后端

- **Web 框架**: FastAPI (Python 3.12)
- **编排引擎**: LangGraph (>=0.2.74)
- **智能体框架**: Claude Agent SDK (>=0.1.5)
- **AG-UI 服务端**: 自定义实现（基于 AG-UI Protocol 规范）
- **数据库**: PostgreSQL (生产) / SQLite (开发)
- **缓存**: Redis (Checkpoint 存储)

#### 基础设施

- **沙箱环境**: E2B (>=2.6.4)
- **版本控制**: Git
- **部署**: Docker + Kubernetes (可选)
- **监控**: Prometheus + Grafana

#### MCP 工具

- **Playwright MCP**: 浏览器自动化测试
- **Sequential MCP**: 结构化推理（用于 Reflect 评估）
- **Context7 MCP**: 技术文档查询（可选）

### 4.3 LangGraph 工作流设计

#### 节点拓扑结构

工作流包含 12 个节点：

1. **requirement_analysis**: 需求分析节点
2. **reflect_requirement**: 需求评估节点
3. **architecture_design**: 架构设计节点
4. **reflect_architecture**: 架构评估节点
5. **code_generation**: 代码生成节点
6. **reflect_code**: 代码评估节点
7. **e2e_testing**: 端到端测试节点
8. **reflect_testing**: 测试评估节点
9. **create_sandbox**: 创建沙箱节点
10. **deploy_service**: 部署服务节点
11. **reflect_deployment**: 部署评估节点
12. **END**: 结束节点

#### 条件边逻辑

每个 Reflect 节点后都有条件边：

- **评分 >= 0.7**: 继续下一阶段
- **评分 < 0.7 且重试次数 < 3**: 回退到上一节点重试
- **重试次数 >= 3**: 强制进入下一阶段（记录警告）

#### 中断机制设计

**后端中断处理流程**:

1. 接收客户端中断请求（POST /api/interrupt）
2. 设置中断标志（Redis 或内存）
3. LangGraph 在下一个节点开始时检查中断标志
4. 检测到中断后，保存 checkpoint 并停止执行
5. 向客户端发送 RUN_FINISHED 事件（outcome="interrupt"）

**后端恢复执行流程**:

1. 接收客户端恢复请求（POST /api/sessions/{threadId}/start，携带 resumeFromCheckpoint）
2. 从 checkpoint 恢复 AgentState
3. 清除中断标志
4. 继续执行后续节点
5. 向客户端发送后续工作流事件

### 4.4 回滚机制设计

#### 状态保存策略

- LangGraph Checkpointer 在每个节点后自动保存状态
- Git 在每个节点后自动 commit 代码
- 数据库记录 checkpoint 与 Git commit 的映射关系

#### 回滚执行流程

1. **提供历史版本列表**: 响应 GET /api/sessions/{threadId}/versions 请求，返回可回滚的版本列表
2. **接收回滚请求**: 接收客户端回滚请求（POST /api/sessions/{threadId}/rollback）
3. **执行回滚操作**:
   - 从 Checkpointer 恢复 AgentState 到目标轮次
   - 执行 Git 版本回滚到对应 commit
   - 截断 Messages 历史到目标轮次
4. **发送状态更新**: 向客户端发送回滚后的状态快照和代码信息

---

## 5. 数据模型

### 5.1 AgentState 核心字段

#### 会话元数据

- `thread_id`: 会话线程ID
- `run_id`: 当前运行ID
- `session_round`: 会话轮次（用于回滚粒度）
- `created_at`, `updated_at`: 时间戳

#### 用户输入

- `user_input`: 用户的需求描述
- `user_messages`: 完整的 Messages 历史（AG-UI 格式）

#### 项目信息

- `project_name`: 项目名称
- `project_folder`: 项目文件夹名称
- `project_path`: 完整项目路径
- `project_type`: 项目类型（web_app, cli_tool, api_service）

#### Git 版本控制

- `git_initialized`: 是否已初始化 Git
- `git_current_commit`: 当前 commit ID
- `git_history`: Git 历史记录（每个节点一个 commit）

#### E2B 沙箱

- `sandbox_id`: E2B 沙箱ID
- `sandbox_url`: 沙箱访问链接
- `sandbox_expires_at`: 沙箱过期时间（12小时后）
- `deployment_port`: 部署端口号
- `access_url`: 最终访问链接

#### 智能体输出文档

- `requirement_doc`: PRD 文档内容（Markdown 格式）
- `architecture_doc`: 架构设计文档
- `code_files`: 生成的代码文件（字典：路径→内容）
- `test_results`: 测试结果
- `deployment_logs`: 部署日志

#### Reflect 评估结果

- `reflect_results`: 各阶段评估结果
  - `score`: 0-1 分数
  - `feedback`: 具体改进建议
  - `decision`: "pass" | "retry"
  - `retry_count`: 当前重试次数

#### 工作流状态

- `current_node`: 当前执行的节点名称
- `completed_nodes`: 已完成的节点列表
- `node_start_times`, `node_end_times`: 节点时间记录

#### 中断和错误处理

- `interrupt_requested`: 是否请求中断
- `interrupt_reason`: 中断原因
- `error`, `error_node`: 错误信息和发生位置

#### 回滚相关

- `checkpoint_id`: 当前 checkpoint ID
- `rollback_available`: 是否可回滚
- `rollback_target_round`: 回滚目标轮次

#### 执行日志

- `logs`: 执行日志数组，每条包含 timestamp、level、node、message、data

---

## 6. 通信协议规范

### 6.1 AG-UI Protocol 集成

Genie 采用 **AG-UI Protocol** 作为前后端通信协议，确保标准化和可扩展性。

#### 协议选择

- **传输方式**: WebSocket
- **连接 URL**: `wss://api.genie.app/v1/agui`
- **认证**: Bearer Token
- **消息格式**: JSON-encoded AG-UI events

#### 核心事件类型

**生命周期事件**:

- `RUN_STARTED`: 工作流启动（包含 threadId, runId）
- `STEP_STARTED`: 节点开始执行（包含 stepName）
- `STEP_FINISHED`: 节点执行完成
- `RUN_FINISHED`: 工作流完成（包含 result 或 outcome="interrupt"）
- `RUN_ERROR`: 执行错误（包含 message, code）

**消息事件**:

- `TEXT_MESSAGE_START`: 开始新消息（包含 messageId, role）
- `TEXT_MESSAGE_CONTENT`: 流式内容块（包含 delta）
- `TEXT_MESSAGE_END`: 消息结束

**工具调用事件**:

- `TOOL_CALL_START`: 工具调用开始（包含 toolCallId, toolCallName）
- `TOOL_CALL_ARGS`: 工具参数（流式传输）
- `TOOL_CALL_END`: 工具调用结束

**状态管理事件**:

- `STATE_SNAPSHOT`: 完整状态快照
- `STATE_DELTA`: 增量状态更新（JSON Patch 格式）
- `MESSAGES_SNAPSHOT`: 完整消息历史

**自定义事件**:

- `CUSTOM`: 自定义事件（如进度百分比、Reflect 评分）

### 6.2 REST API 规范

#### POST `/api/sessions`

创建新会话，返回 threadId

#### POST `/api/sessions/{threadId}/start`

启动工作流，支持从 checkpoint 恢复

#### POST `/api/sessions/{threadId}/interrupt`

中断执行，保存当前 checkpoint

#### GET `/api/sessions/{threadId}/versions`

获取可回滚的历史版本列表

#### POST `/api/sessions/{threadId}/rollback`

回滚到指定版本

#### GET `/api/sessions/{threadId}/state`

获取当前状态和进度

---

## 7. 用户交互流程

### 7.1 首次生成流程

1. **用户输入需求** → 前端发送创建会话请求
2. **创建会话** → 获取 thread_id
3. **启动工作流** → 建立 WebSocket 连接
4. **接收 RUN_STARTED 事件** → 显示"开始生成..."
5. **接收 STEP_STARTED 事件** → 显示当前节点名称
6. **接收 TEXT_MESSAGE_CONTENT 事件** → 实时显示智能体输出
7. **接收 CUSTOM 事件** → 显示 Reflect 评分和反馈
8. **接收 RUN_FINISHED 事件** → 显示完成状态和访问链接

### 7.2 中断与恢复流程

**中断流程**:

1. 用户点击"中断"按钮
2. 前端发送 POST /api/interrupt 请求
3. 后端设置中断标志
4. LangGraph 检测到中断，停止执行
5. 前端接收 RUN_FINISHED（outcome="interrupt"）
6. 显示"已中断"状态 + "继续"按钮

**恢复流程**:

1. 用户点击"继续"按钮
2. 前端发送 POST /api/sessions/{threadId}/start（携带 resumeFromCheckpoint）
3. 后端从 checkpoint 恢复状态
4. 继续执行后续节点
5. 前端继续接收事件流

### 7.3 回滚流程

1. **查看历史版本** → GET /api/sessions/{threadId}/versions
2. **选择目标版本** → 用户选择回滚到第1轮
3. **执行回滚** → POST /api/sessions/{threadId}/rollback
4. **更新 UI** → 前端显示回滚后的状态和代码
5. **重新生成** → 用户可以输入新需求继续生成

---

## 8. 非功能需求

### 8.1 性能要求

| 指标                | 目标值    | 说明                                   |
| ------------------- | --------- | -------------------------------------- |
| 单次生成时间        | < 15 分钟 | 从需求输入到部署完成（中等复杂度应用） |
| WebSocket 延迟      | < 100ms   | 事件从后端发送到前端接收的延迟         |
| Checkpoint 保存时间 | < 1 秒    | 每个节点后保存状态的时间               |
| 回滚操作时间        | < 3 秒    | 从请求回滚到完成的时间                 |
| 并发会话数          | ≥ 100    | 单机支持的同时活跃会话数               |

### 8.2 可靠性要求

- **断点续传**: 支持网络中断后自动重连并恢复状态
- **错误重试**: 节点执行失败时自动重试（最多 3 次）
- **数据持久化**: 所有 AgentState 和 Messages 持久化到数据库
- **Checkpoint 备份**: 每个节点后自动保存 checkpoint

### 8.3 安全性要求

- **沙箱隔离**: 所有代码在 E2B 沙箱中执行，与主机隔离
- **代码审计**: 生成的代码通过 Linter 和 SAST 工具检查
- **敏感信息过滤**: 自动检测并过滤 API Key、密码等敏感信息

### 8.4 可观测性要求

- **日志系统**: 完整记录每个节点的输入输出和执行时间
- **Metrics**: 监控 LLM token 使用量、API 调用次数、错误率
- **Tracing**: 支持分布式追踪

### 8.5 可扩展性要求

- **水平扩展**: 支持通过增加服务器节点提升并发能力
- **插件机制**: 支持添加新的智能体类型和 MCP 工具
- **多语言支持**: 架构支持扩展到其他编程语言（Java、Go 等）

---

## 9. 风险与限制

### 9.1 技术风险

| 风险                                        | 影响                   | 缓解措施                                                 |
| ------------------------------------------- | ---------------------- | -------------------------------------------------------- |
| **LLM 生成代码质量不稳定**            | 生成的代码可能存在 bug | 1. 多层 Reflect 评估 2. 自动化测试 3. 限制项目复杂度     |
| **E2B 沙箱过期（12小时）**            | 用户链接失效           | 1. 提醒用户沙箱时效 2. 提供导出代码功能 3. 支持重新部署  |
| **Reflect 循环导致长时间执行**        | 用户等待时间过长       | 1. 设置最大重试次数（3次） 2. 允许用户跳过 Reflect       |
| **LangGraph checkpoint 存储膨胀**     | 数据库存储压力         | 1. 定期清理旧 checkpoint 2. 压缩存储格式                 |
| **Claude Agent SDK interrupt() 限制** | 只能在流式模式中断     | 1. 确保所有智能体使用流式模式 2. 结合 LangGraph 节点检查 |

### 9.2 业务风险

| 风险                     | 影响           | 缓解措施                                          |
| ------------------------ | -------------- | ------------------------------------------------- |
| **LLM API 成本高** | 运营成本压力   | 1. 实现 token 限制 2. 缓存常见需求 3. 优化 prompt |
| **用户期望过高**   | 用户满意度下降 | 1. 明确功能边界 2. 提供示例和最佳实践             |

### 9.3 当前限制

1. **项目复杂度**: 适用于中小型应用（< 10 个文件，< 2000 行代码）
2. **技术栈**: 当前支持 React + FastAPI/Express，未来扩展其他栈
3. **部署环境**: 仅支持 E2B 沙箱，不支持自定义服务器
4. **多人协作**: v1.0 不支持多用户协作编辑同一项目
5. **实时预览**: 代码生成过程中不提供实时预览，需等待完成

---

## 10. 实施路线图

### Phase 1: 核心工作流搭建 (4 周)

**目标**: 实现基础的 LangGraph 多智能体编排

**交付物**:

- 可运行的 LangGraph 工作流
- 完整的节点实现代码
- 测试覆盖率 > 80%

**关键任务**:

- 实现 LangGraph StateGraph 定义
- 实现 7 个子智能体节点
- 实现条件边逻辑（Reflect 评估和回退）
- 集成 PostgreSQL Checkpointer
- 单元测试和集成测试

### Phase 2: AG-UI Protocol 集成 (3 周)

**目标**: 实现前后端通信协议

**交付物**:

- AG-UI Server 实现
- AG-UI 事件流文档
- WebSocket API 文档

**关键任务**:

- 实现 FastAPI WebSocket 服务器
- 实现 LangGraph 事件到 AG-UI 事件的转换器
- 实现 AG-UI 标准事件推送
- 实现 Messages 和 State 同步机制

### Phase 3: 中断和回滚机制 (2 周)

**目标**: 实现用户控制功能

**交付物**:

- 中断恢复功能
- 回滚功能
- 用户操作文档

**关键任务**:

- 实现中断标志存储
- 实现节点级中断检查
- 实现回滚 API
- 实现 Git 版本回滚逻辑
- 实现 Messages 历史截断

### Phase 4: 测试和优化 (2 周)

**目标**: 性能优化和 Bug 修复

**交付物**:

- 稳定的 v1.0.0 版本
- 完整的技术文档
- 用户指南和视频教程

**关键任务**:

- 端到端测试（10+ 场景）
- 性能测试和优化
- 错误处理完善
- 文档完善
- Beta 用户测试

---

## 11. 附录

### 11.1 术语表

| 术语                                   | 定义                                                          |
| -------------------------------------- | ------------------------------------------------------------- |
| **AG-UI Protocol**               | Agent-User Interaction Protocol，标准化的智能体与前端通信协议 |
| **LangGraph**                    | LangChain 出品的图编排框架，用于构建复杂的智能体工作流        |
| **Reflect 节点**                 | 质量评估节点，负责评估上一个智能体的输出并决定是否重试        |
| **Checkpoint**                   | LangGraph 的状态快照机制，用于断点续传和回滚                  |
| **E2B Sandbox**                  | 云端代码执行环境，提供隔离的运行环境                          |
| **Claude Agent SDK**             | Anthropic 官方的 Claude 智能体开发工具包                      |
| **MCP (Model Context Protocol)** | 智能体工具调用标准协议                                        |
| **会话轮次 (Session Round)**     | 用户每次提交需求或修改意见算一轮                              |

### 11.2 参考文档

- **AG-UI Protocol 官方文档**: [https://docs.ag-ui.com/introduction](https://docs.ag-ui.com/introduction)
- **LangGraph 官方文档**: [https://langchain-ai.github.io/langgraph/](https://langchain-ai.github.io/langgraph/)
- **Claude Agent SDK Python**: [https://docs.claude.com/zh-CN/docs/agent-sdk/python](https://docs.claude.com/zh-CN/docs/agent-sdk/python)
- **E2B Python SDK**: [https://e2b.dev/docs/sdk-reference/python-sdk/v2.6.4/sandbox_async](https://e2b.dev/docs/sdk-reference/python-sdk/v2.6.4/sandbox_async)

### 11.3 相关文档

- **技术设计文档 (TDD)**: 详细的技术实现规范和代码示例（待编写）
- **API 接口文档**: REST API 和 WebSocket 事件的完整规范（待编写）
- **部署运维文档**: 生产环境部署和运维指南（待编写）

---

## 文档变更历史

| 版本   | 变更说明                                                       |
| ------ | -------------------------------------------------------------- |
| v1.0.2 | 删除前端开发内容，明确Genie仅负责后端服务和AG-UI协议服务端实现 |
| v1.0.1 | 删除代码实现细节，精简为纯需求文档                             |
| v1.0.0 | 初版 PRD，包含完整的技术架构和实施计划                         |

---

**下一步行动**:

1. 技术团队评审 PRD 可行性
2. 与前端团队对齐 AG-UI Protocol 接口规范
3. 编写技术设计文档 (TDD)，包含详细的代码实现
4. 确定开发资源和时间表
5. 开始 Phase 1 开发
