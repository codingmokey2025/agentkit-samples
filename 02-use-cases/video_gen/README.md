# 🎬 视频故事生成器

基于火山引擎 AgentKit 构建的智能视频生成智能体,通过生成插图并无缝组合成动画序列,从文本提示词创建完整的故事绘本视频。

## 📋 概述

本用例展示如何构建一个生产级视频生成系统,具备以下能力:

- **生成插画分镜** 根据用户提示词生成四张卡通风格的关键帧
- **创建过渡视频** 在连续帧之间生成流畅的动画过渡
- **组合视频片段** 使用本地 MCP 工具将片段合成为完整作品
- **上传和分享** 通过 TOS 对象存储上传最终视频并生成签名 URL
- **维护会话上下文** 通过短期记忆支持迭代优化

## 🏗️ 架构

```
用户提示词
    ↓
AgentKit 运行时
    ↓
视频故事生成器
    ├── 图像生成工具 (Visual AI)
    ├── 视频生成工具 (Visual AI)
    ├── 文件下载工具 (批量下载)
    ├── 视频拼接工具 (MCP)
    └── TOS 上传工具 (存储与分享)
```

### 核心组件

| 组件 | 描述 |
|-----------|-------------|
| **Agent 服务** | [`agent.py`](agent.py) - 主应用程序,包含 MCP 工具注册 |
| **Agent 配置** | [`agent.yaml`](agent.yaml) - 模型设置、系统指令和工具列表 |
| **自定义工具** | [`tool/`](tool/) - 文件下载和 TOS 上传实用工具 |
| **MCP 集成** | `@pickstar-2002/video-clip-mcp` - 本地视频拼接服务 |
| **短期记忆** | 会话上下文维护以保持对话连续性 |

## 🚀 快速开始

### 前置条件

**1. Node.js 环境**

- 安装 Node.js 18+ 和 npm
- 确保终端中可以使用 `npx` 命令
- MCP 视频拼接工具运行所需

**2. 火山引擎访问凭证**

1. 登录 [火山引擎控制台](https://console.volcengine.com)
2. 进入"访问控制" → "密钥管理"
3. 创建具有以下权限的 Access Key 和 Secret Key:
   - Visual AI (图像和视频生成)
   - TOS 读写访问权限

### 安装依赖

```bash
cd 02-use-cases/video_gen

# 如果未安装 uv,请先安装(任选其一):
# macOS / Linux (官方安装脚本)
curl -LsSf https://astral.sh/uv/install.sh | sh
# 或使用 Homebrew (macOS)
brew install uv

# 初始化项目依赖
uv sync
source .venv/bin/activate

# 或使用传统的 pip
pip3 install -r requirements.txt
```

**注意:** MCP 视频工具 (`@pickstar-2002/video-clip-mcp`) 在智能体运行时会通过 `npx` 自动启动。无需手动安装。

### 配置环境变量

设置以下环境变量:

```bash
export VOLCENGINE_ACCESS_KEY=<您的_AK>
export VOLCENGINE_SECRET_KEY=<您的_SK>
export DATABASE_TOS_BUCKET=<您的_存储桶名称>

# 可选: 指定下载目录 (默认为项目根目录)
export DOWNLOAD_DIR=/tmp
```

**TOS 存储桶配置:**
- 默认存储桶: `agentkit-platform-{{your_account_id}}`
- 若需自定义,可在 [`tool/tos_upload.py`](tool/tos_upload.py) 中修改 `bucket_name` 参数或在工具调用时传入

## 🧪 本地测试

### 方法 1: 直接 API 调用

启动智能体服务:

```bash
uv run agent.py
# 服务默认监听 0.0.0.0:8000
```

**步骤 1: 获取应用名称**

智能体名称与 [`agent.yaml`](agent.yaml) 中的 `name` 字段一致,即 `storybook_illustrator`。

```bash
curl --location 'http://localhost:8000/list-apps'
```

**步骤 2: 创建会话**

```bash
curl --location --request POST 'http://localhost:8000/apps/storybook_illustrator/users/u_123/sessions/s_123' \
--header 'Content-Type: application/json' \
--data ''
```

**步骤 3: 发送消息**

```bash
curl --location 'http://localhost:8000/run_sse' \
--header 'Content-Type: application/json' \
--data '{
    "appName": "storybook_illustrator",
    "userId": "u_123",
    "sessionId": "s_123",
    "newMessage": {
        "role": "user",
        "parts": [{
            "text": "请根据寓言《狐假虎威》生成绘本故事视频"
        }]
    },
    "streaming": true
}'
```

### 方法 2: 使用 veadk web

使用 Web 界面进行便捷调试:

```bash
# 进入 use-cases 根目录
cd 02-use-cases

# 启动 veadk web 服务
veadk web --port 8000
```

在浏览器中访问 `http://localhost:8000`,选择 `video_gen` 智能体,输入提示词并点击"Send"。

### 示例提示词

- **中国成语**: "后羿射日,嫦娥奔月,吴刚伐木真人版"
- **经典故事**: "愚公移山与精卫填海绘本故事"
- **武侠小说**: "射雕英雄传的真人版视频故事"
- **玄幻小说**: "凡人修仙传韩立结婴"
- **3D 动画**: "凡人修仙传虚天殿大战,3D 动漫风格"

**预期行为:**
1. 生成 4 张插画分镜帧
2. 在连续帧之间创建 3 段过渡视频
3. 启动本地 MCP 工具拼接视频
4. 上传最终视频到 TOS
5. 返回用于观看的签名 URL

## 🚢 部署

部署到火山引擎 AgentKit Runtime:

```bash
# 1. 进入项目目录
cd 02-use-cases/video_gen

# 2. 配置并部署
agentkit config \
--agent_name storybook_illustrator \
--entry_point 'agent.py' \
--runtime_envs DATABASE_TOS_BUCKET=<您的_存储桶名称> \
--launch_type cloud

# 3. 部署到运行时
agentkit launch
```

部署成功后:
1. 访问 [火山引擎 AgentKit 控制台](https://console.volcengine.com/agentkit/region:agentkit+cn-beijing/runtime)
2. 点击 **Runtime** 查看已部署的智能体 `storybook_illustrator`
3. 获取公网访问域名 (例如: `https://xxxxx.apigateway-cn-beijing.volceapi.com`) 和 API Key

### 测试已部署的智能体

**创建会话:**
```bash
curl --location --request POST 'https://xxxxx.apigateway-cn-beijing.volceapi.com/apps/storybook_illustrator/users/u_123/sessions/s_124' \
--header 'Content-Type: application/json' \
--header 'Authorization: <您的_api_key>' \
--data ''
```

**发送消息:**
```bash
curl --location 'https://xxxxx.apigateway-cn-beijing.volceapi.com/run_sse' \
--header 'Authorization: <您的_api_key>' \
--header 'Content-Type: application/json' \
--data '{
    "appName": "storybook_illustrator",
    "userId": "u_123",
    "sessionId": "s_124",
    "newMessage": {
        "role": "user",
        "parts": [{
            "text": "请根据寓言《狐假虎威》生成绘本故事视频"
        }]
    },
    "streaming": false
}'
```

## 📁 项目结构

```
video_gen/
├── agent.py              # Agent 入口,包含 MCP 集成
├── agent.yaml            # Agent 配置 (模型、指令、工具)
├── tool/                 # 自定义工具实现
│   ├── file_download.py  # 批量文件下载工具
│   └── tos_upload.py     # TOS 上传及签名 URL 生成
├── requirements.txt      # Python 依赖
├── pyproject.toml        # 项目配置 (uv/pip 依赖与元数据)
├── __init__.py           # 包初始化文件
├── .python-version       # Python 版本声明 (开发环境)
├── README.md            # 项目文档
└── .dockerignore         # Docker 构建排除项
```

## 🔍 主要特性

### 智能分镜生成
自动将叙事分解为 4 个视觉关键帧,保持风格一致性和角色连续性。

### 无缝视频过渡
使用先进的视觉 AI 模型在帧之间生成流畅的过渡视频。

### 本地 MCP 工具集成
利用模型上下文协议进行高效的本地视频处理,无需云端依赖。

### 自动上传与分享
将完成的视频上传到 TOS,并生成限时签名 URL 以安全分享。

### 迭代优化
维护对话上下文,允许用户请求对风格、节奏或内容进行调整。

## ❓ 常见问题

**错误: `npx` 命令未找到**
- 安装 Node.js 18+ 和 npm
- 在终端中验证 `npx --version` 可以正常运行

**TOS 上传失败**
- 确认已设置 `VOLCENGINE_ACCESS_KEY` 和 `VOLCENGINE_SECRET_KEY`
- 验证您的账户具有 TOS 存储桶访问权限

**视频生成超时**
- 复杂场景可能需要更长的生成时间
- 检查 Visual AI 服务配额和速率限制

**MCP 工具连接错误**
- 确保默认 MCP 端口没有冲突
- 查看 Node.js 进程日志以获取详细错误信息

**使用自定义 TOS 存储桶**
- 通过环境变量设置: `export DATABASE_TOS_BUCKET="agentkit-platform-{{account_id}}"`
- 或在 [`tool/tos_upload.py`](tool/tos_upload.py) 中修改默认值

**uv sync 失败**
- 确保已安装 Python 3.12+
- 检查 `.python-version` 文件与您的 Python 安装版本是否匹配
- 尝试使用 `uv sync --refresh` 重新构建依赖

## 🔗 相关资源

- [AgentKit 文档](https://www.volcengine.com/docs/agentkit)
- [Visual AI 服务](https://www.volcengine.com/docs/visual-ai)
- [TOS 对象存储](https://www.volcengine.com/docs/tos)
- [模型上下文协议 (MCP)](https://modelcontextprotocol.io)
- [`agent.yaml`](agent.yaml) - 从分镜到视频上传的完整工作流定义
