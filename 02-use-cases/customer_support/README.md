# 💬 客服智能体

基于火山引擎 AgentKit 构建的生产级客服智能体,通过智能路由、知识库集成和 CRM 工具连接,同时处理产品咨询和售后支持场景。

## 📋 概述

本用例展示如何构建一个企业级客服支持系统,具备以下能力:

- **智能对话路由** 在导购咨询和售后支持子智能体之间进行智能分流
- **知识库集成** 整合产品信息、政策文档和故障排查指南
- **CRM 系统连接** 查询客户数据、购买记录、保修状态和服务工单
- **维护对话上下文** 通过短期和长期记忆 (Viking 或 Mem0) 保持会话连续性
- **验证用户身份** 在访问敏感信息或执行操作前进行身份校验
- **提供结构化响应** 确保专业、一致的客户交互体验

## 🏗️ 架构

```
客户咨询
    ↓
AgentKit 运行时
    ↓
客服智能体 (主路由器)
    ├── 导购咨询子智能体
    │   ├── 产品知识库
    │   └── 客户信息工具
    └── 售后支持子智能体
        ├── 保修与政策知识
        ├── 故障排查指南
        └── 服务工单管理 (CRUD)
```

### 核心组件

| 组件 | 描述 |
|-----------|-------------|
| **Agent 服务** | [`agent.py`](agent.py) - 主应用程序,通过 `AgentkitAgentServerApp` 实现子智能体编排 |
| **CRM 工具** | [`tools/crm_mock.py`](tools/crm_mock.py) - 模拟 CRM API,提供客户信息、购买、保修和工单增删改查 |
| **知识库** | [`pre_build/knowledge/`](pre_build/knowledge/) - 产品指南、政策文档和故障排查文档 |
| **短期记忆** | 本地会话上下文,保持对话连续性 |
| **长期记忆** | Viking 向量数据库或 Mem0,持久化用户历史记录 |

## 🚀 快速开始

### 前置条件

**Python 版本**
- 需要 Python 3.12 或更高版本

**火山引擎服务**
1. **访问凭证**: 登录 [火山引擎控制台](https://console.volcengine.com)
   - 进入"访问控制" → "密钥管理"
   - 创建具有以下权限的 Access Key 和 Secret Key:
     - AgentKit (智能体运行时和工具)
     - TOS (知识库文件存储)
     - Viking (知识和记忆的向量数据库)

2. **知识库** (首次运行自动配置):
   - 如未设置 `DATABASE_VIKING_COLLECTION`,智能体将自动:
     - 上传 `pre_build/knowledge/` 中的文件到 TOS
     - 创建 Viking 集合
     - 导入知识库内容
   - 生产环境建议手动创建知识库并设置集合名称

### 安装依赖

```bash
cd 02-use-cases/customer_support
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

# 可选: 使用已有知识库
export DATABASE_VIKING_COLLECTION=<existing_knowledge_index>

# 可选: 长期记忆 (二选一)
# 选项 1: Viking 记忆
export DATABASE_VIKINGMEM_COLLECTION=<mem_index>
export DATABASE_VIKINGMEM_MEMORY_TYPE=<memory_type>

# 选项 2: Mem0
export DATABASE_MEM0_BASE_URL=<mem0_base_url>
export DATABASE_MEM0_API_KEY=<mem0_api_key>
```

**环境变量说明:**
- `DATABASE_TOS_BUCKET`: 用于自动知识库初始化所需 (如未设置 `DATABASE_VIKING_COLLECTION`)
- `DATABASE_VIKING_COLLECTION`: 预创建的知识库集合名称 (生产环境推荐)
- 模型默认为 `deepseek-v3-1-terminus` (可在代码中配置)

## 🧪 本地测试

使用 `veadk web` 进行本地调试:

```bash
# 1. 进入上级目录
cd 02-use-cases

# 2. 可选: 创建 .env 文件 (如果已设置环境变量可跳过)
touch .env
echo "VOLCENGINE_ACCESS_KEY=AK" >> .env
echo "VOLCENGINE_SECRET_KEY=SK" >> .env
# 推荐: 在 AgentKit 控制台手动创建知识库并设置集合名称
echo "DATABASE_VIKING_COLLECTION=agentkit_customer_support" >> .env
# 可选: 如果使用自动初始化,设置 TOS 存储桶用于上传知识库文件
echo "DATABASE_TOS_BUCKET=agentkit-platform-{{your_account_id}}" >> .env

# 3. 启动 Web 界面
veadk web
```

服务默认运行在 8000 端口。访问 `http://127.0.0.1:8000`,选择 `customer_support` 智能体,在输入面板中开始测试。

### 示例提示词

**售后场景:**
- "你好,我之前买的电视坏了"
- "我的邮箱是 zhang.ming@example.com,电视序列号是 SN20240001"
- "我需要帮助排查手机故障 - 无法开机"

**导购咨询:**
- "我想买一款客厅用的智能电视,主要用来打游戏,预算 3000 元以内"
- "你们的智能手机保修政策是什么?"
- "能推荐一款续航好的手机吗?"

**预期行为:**
- 智能体自动识别"导购"与"售后"意图并路由到相应的子智能体
- 基于工具和知识库返回结构化响应
- 必要时引导用户进行身份验证
- 检索购买历史和保修状态
- 仅在获得用户明确同意且信息完整时创建/更新/删除服务工单

## 🚢 部署

部署到火山引擎 AgentKit Runtime:

```bash
# 1. 进入项目目录
cd 02-use-cases/customer_support

# 2. 配置 agentkit
agentkit config \
--agent_name customer_support \
--entry_point 'agent.py' \
--runtime_envs DATABASE_TOS_BUCKET=agentkit-platform-{{your_account_id}} \
--launch_type cloud

# 3. 部署到运行时
agentkit launch
```

部署成功后,可在火山引擎 AgentKit 控制台的 Runtime 下查看您的智能体。

## 📁 项目结构

```
customer_support/
├── agent.py                          # 主智能体,包含子智能体编排
├── tools/
│   └── crm_mock.py                   # 模拟 CRM 工具 (客户、购买、保修、工单)
├── pre_build/
│   └── knowledge/                    # 知识库文件
│       ├── policies.md               # 退换货与保修政策
│       ├── shopping_guide.md         # 产品咨询知识
│       ├── troubleshooting_for_phone.md   # 手机故障排查指南
│       └── troubleshooting_for_tv.md      # 电视故障排查指南
├── requirements.txt                  # Python 依赖
├── README.zh.md                     # 项目文档 (中文)
└── .dockerignore                     # Docker 构建排除项
```

## 🔍 主要特性

### 智能意图路由
自动区分导购咨询和售后支持请求,路由到配备相关工具和知识的专业子智能体。

### 知识库集成
结合结构化文档与向量搜索,为产品问题、政策查询和故障排查场景提供准确的上下文感知响应。

### CRM 工具连接
模拟 CRM 接口展示集成模式:
- 客户信息检索
- 购买历史查询
- 保修状态验证
- 服务工单 CRUD 操作

### 身份验证
在访问敏感数据或执行账户操作前,通过邮箱确认验证用户身份。

### 记忆管理
- **短期记忆**: 在会话内维护对话上下文
- **长期记忆**: 通过 Viking 或 Mem0 跨会话持久化用户偏好和历史记录

### 可扩展架构
工具和知识解耦,允许无缝替换为真实 CRM API 或添加更多业务集成。

## ❓ 常见问题

**错误: `DATABASE_TOS_BUCKET not set`**
- 自动知识库初始化所需
- 设置用于上传知识文件的 TOS 存储桶名称
- 替代方案: 手动创建知识库并使用 `DATABASE_VIKING_COLLECTION`

**知识库未初始化**
- 如果未设置 `DATABASE_VIKING_COLLECTION`,首次运行会触发自动导入
- 确保 TOS 配置正确且账户具有权限
- 在 AgentKit 控制台检查导入任务状态

**默认测试用户 `CUST001`**
- 演示数据绑定到此客户 ID
- 生产环境部署应在请求头中传递 `user_id` 并集成真实身份系统

**将模拟 CRM 替换为真实 API**
- 修改 [`tools/crm_mock.py`](tools/crm_mock.py) 以调用实际 CRM 端点
- 保持一致的接口语义 (查询/创建/更新/删除)
- 维护参数名称和返回值结构

**服务工单操作需要用户同意**
- 创建/更新/删除工单需要获得用户明确批准
- 确保在操作前收集所有必需字段 (customer_id、product_sn、issue_description)

## 🔗 相关资源

- [AgentKit 文档](https://www.volcengine.com/docs/agentkit)
- [Viking 向量数据库](https://www.volcengine.com/docs/viking)
- [TOS 对象存储](https://www.volcengine.com/docs/tos)
- [Mem0 记忆管理](https://mem0.ai)
