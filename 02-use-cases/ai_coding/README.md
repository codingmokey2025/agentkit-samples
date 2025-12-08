# 🤖 AI 编程助手

基于火山引擎 AgentKit 构建的智能编程助手,通过代码执行和自动化部署到 TOS 对象存储来帮助用户解决编程挑战。

## 📋 概述

本用例展示如何构建一个生产级 AI 编程助手系统,具备以下能力:

- **理解编程需求** 并生成跨多种编程语言的准确代码解决方案
- **在沙箱环境中执行代码** 以验证正确性和运行行为
- **自动部署前端代码** 通过上传 HTML/CSS/JS 到 TOS 并生成预览 URL
- **维护对话上下文** 通过会话记忆和用户历史记录
- **提供可观测性** 集成 OpenTelemetry 追踪和 APMPlus 监控

## 🏗️ 架构

```
用户请求
    ↓
AgentKit 运行时
    ↓
AI 编程助手
    ├── 代码执行工具 (run_code)
    ├── TOS 上传工具 (upload_frontend_code_to_tos)
    └── URL 生成工具 (get_url_of_frontend_code_in_tos)
```

### 核心组件

| 组件 | 描述 |
|-----------|-------------|
| **Agent 服务** | [`agent.py`](agent.py) - 主智能体应用,包含配置和运行逻辑 |
| **工具模块** | [`tools.py`](tools.py) - TOS 上传、URL 生成和实用工具函数 |
| **沙箱执行** | 支持 Python、Java、JavaScript、Go 的安全代码执行环境 |
| **TOS 集成** | 用于托管前端代码并提供公共访问的对象存储服务 |

## 🚀 快速开始

### 前置条件

**1. 火山引擎访问凭证**

1. 登录 [火山引擎控制台](https://console.volcengine.com)
2. 进入"访问控制" → "密钥管理"
3. 点击"创建密钥"生成 Access Key 和 Secret Key
4. 为凭证配置 AgentKit 产品权限:
   - 进入"访问控制" → "策略管理"
   - 添加 AgentKit 和 TOS 读写权限
   - 将策略绑定到您的 Access Key

**2. AgentKit 工具 ID**

1. 登录火山引擎 AgentKit 控制台
2. 进入"工具管理" → "工具列表"
3. 创建新工具:
   - 工具名称: `ai-coding-agent`
   - 描述: AI 编程助手工具
4. 复制生成的工具 ID 用于配置

### 安装依赖

```bash
cd 02-use-cases/ai_coding
uv pip install -r requirements.txt
# 或
pip3 install -r requirements.txt
```

### 配置环境变量

设置以下环境变量:

```bash
export VOLCENGINE_ACCESS_KEY=AK
export VOLCENGINE_SECRET_KEY=SK
export DATABASE_TOS_BUCKET=agentkit-platform-{{your_account_id}}
export AGENTKIT_TOOL_ID={{your_tool_id}}
```

**环境变量说明:**
- `DATABASE_TOS_BUCKET`: 用于存储生成的前端代码的 TOS 存储桶

## 🧪 本地测试

使用 `veadk web` 进行本地调试:

```bash
# 1. 进入上级目录
cd 02-use-cases

# 2. 可选: 创建 .env 文件 (如果已设置环境变量可跳过)
touch .env
echo "VOLCENGINE_ACCESS_KEY=AK" >> .env
echo "VOLCENGINE_SECRET_KEY=SK" >> .env
echo "DATABASE_TOS_BUCKET=agentkit-platform-{{your_account_id}}" >> .env
echo "AGENTKIT_TOOL_ID={{your_tool_id}}" >> .env

# 3. 启动 Web 界面
veadk web
```

服务默认运行在 8000 端口。访问 `http://127.0.0.1:8000`,选择 `ai_coding` 智能体,开始测试。

### 示例提示词

- **前端代码生成**: "请帮我用 JavaScript 写一个防抖函数"
- **Python 代码生成**: "写一个生成斐波那契数列的函数"
- **算法实现**: "用 Python 创建一个二分查找实现"

## 🚢 部署

部署到火山引擎 AgentKit Runtime:

```bash
# 1. 进入项目目录
cd 02-use-cases/ai_coding

# 2. 配置 agentkit
agentkit config \
--agent_name ai_coding \
--entry_point 'agent.py' \
--runtime_envs DATABASE_TOS_BUCKET=agentkit-platform-{{your_account_id}} \
--runtime_envs AGENTKIT_TOOL_ID={{your_tool_id}} \
--launch_type cloud

# 3. 部署到运行时
agentkit launch
```

部署成功后,可在火山引擎 AgentKit 控制台的 Runtime 下查看您的智能体。

## 📁 项目结构

```
ai_coding/
├── agent.py              # 主智能体应用及配置
├── tools.py              # 工具函数 (TOS 上传、URL 生成)
├── requirements.txt      # Python 依赖
└── README.md            # 项目文档
```

## 🔍 主要特性

### 多语言支持
支持 Python、Java、JavaScript、Go 等主流编程语言,具备自动语法验证。

### 沙箱执行
在隔离环境中运行代码,确保安全性并防止系统干扰。

### 自动化部署
前端代码自动上传到 TOS,生成预览 URL 以便立即测试。

### 可观测性
内置 OpenTelemetry 追踪和 APMPlus 监控,支持生产环境调试和性能分析。

## ❓ 常见问题

**错误: `DATABASE_TOS_BUCKET not set`**
- 需通过环境变量设置用于代码上传的 TOS 存储桶名称

**代码执行超时**
- 检查沙箱服务状态和网络连接
- 验证代码复杂度和执行时间要求

**TOS 上传失败**
- 确认 Access Key/Secret Key 具有 TOS 写入权限
- 验证存储桶名称和区域配置

## 🔗 相关资源

- [AgentKit 文档](https://www.volcengine.com/docs/agentkit)
- [TOS 对象存储](https://www.volcengine.com/docs/tos)
- [火山引擎控制台](https://console.volcengine.com)
