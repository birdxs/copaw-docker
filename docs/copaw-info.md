# CoPaw 官方文档信息汇总

> 本文档汇总了 CoPaw 官方文档的关键信息，方便后续调整需求时快速获取。
>
> 官方文档：http://copaw.agentscope.io/docs/
>
> **更新日期**: 2026-03-03

---

## 重要更新 (2026-03-03)

### v0.0.4 新增功能
- **Telegram 频道支持** - 新增 Telegram 机器人频道
- **OpenAI & Azure OpenAI** - 新增内置模型提供商
- **阿里云 coding-plan 提供商** - 新增模型提供商
- **CORS 配置** - 新增 `COPAW_CORS_ORIGINS` 环境变量
- **心跳监控面板** - 控制台新增监控 UI
- **音频文件支持** - 钉钉和飞书频道支持音频文件

### v0.0.3 新增功能
- **MCP 支持** - 连接外部 MCP 服务器扩展能力
- **本地模型支持增强** - llama.cpp、MLX、Ollama 集成
- **一键安装脚本** - 跨平台安装支持
- **阿里云 ECS 一键部署** - 云端部署选项
- **控制台功能增强** - 技能导入/创建、工作区上传下载、运行配置

### CLI 新增命令
- `copaw models download/remove-local` - 本地模型管理 (llama.cpp/MLX)
- `copaw models ollama-pull/ollama-list/ollama-remove` - Ollama 模型管理
- `copaw channels install/add/remove` - 自定义频道管理
- `copaw cron pause/resume/state` - 定时任务状态管理
- `copaw chats create/update/delete` - 会话管理

---

## 项目概述

### CoPaw 是什么？

CoPaw 是一款**个人助理型产品**，部署在你自己的环境中。

- **多通道对话** — 通过钉钉、飞书、QQ、Discord、iMessage 等与你对话
- **定时执行** — 按你的配置自动运行任务
- **能力由 Skills 决定** — 内置定时任务、PDF 与表单、Word/Excel/PPT 文档处理、新闻摘要、文件阅读等，还可在 Skills 中自定义扩展
- **数据全在本地** — 不依赖第三方托管

### 你怎么用 CoPaw？

使用方式可以概括为两类：

1. **在聊天软件里对话** — 在钉钉、飞书、QQ、Discord 或 iMessage（仅 Mac）里发消息，CoPaw 在同一 app 内回复
2. **定时自动执行** — 按设定时间自动运行任务

### 技术基础

CoPaw 由 [AgentScope 团队](https://github.com/agentscope-ai) 基于以下项目构建：
- [AgentScope](https://github.com/agentscope-ai/agentscope)
- [AgentScope Runtime](https://github.com/agentscope-ai/agentscope-runtime)
- [ReMe](https://github.com/agentscope-ai/ReMe)

部分灵感来源于 [OpenClaw](https://openclaw.ai/)，感谢 [anthropics/skills](https://github.com/anthropics/skills) 提供 Agent Skills 规范与示例。

---

## 快速开始

### 环境要求

- **Python 版本**: >= 3.10, < 3.14

### 安装方式

#### 方式一：一键安装（推荐）

无需预装 Python — 安装脚本通过 `uv` 自动管理一切。

**macOS / Linux：**
```bash
curl -fsSL https://copaw.agentscope.io/install.sh | bash
```

**Windows（PowerShell）：**
```powershell
irm https://copaw.agentscope.io/install.ps1 | iex
```

**可选参数：**
```bash
# 安装指定版本
curl -fsSL ... | bash -s -- --version 0.0.3

# 从源码安装（开发/测试用）
curl -fsSL ... | bash -s -- --from-source

# 安装本地模型支持
bash install.sh --extras llamacpp    # llama.cpp（跨平台）
bash install.sh --extras mlx         # MLX（Apple Silicon）
bash install.sh --extras ollama      # Ollama（需 Ollama 服务运行）
```

#### 方式二：pip 安装

```bash
pip install copaw
```

可选：先创建并激活虚拟环境再安装（`python -m venv .venv`，Linux/macOS 下 `source .venv/bin/activate`，Windows 下 `.venv\Scripts\Activate.ps1`）。

#### 方式三：魔搭创空间一键配置（无需安装）

1. 前往 [魔搭](https://modelscope.cn/register) 注册并登录
2. 打开 [CoPaw 创空间](https://modelscope.cn/studios/fork?target=AgentScope/CoPaw)，一键配置即可使用

> **重要**：使用创空间请将空间设为**非公开**，否则你的 CoPaw 可能被他人操纵。

#### 方式四：Docker

镜像在 **Docker Hub**（`agentscope/copaw`）。镜像 tag：`latest`（稳定版）、`pre`（PyPI 预发布版）。

国内用户也可选用阿里云 ACR：`agentscope-registry.ap-southeast-1.cr.aliyuncs.com/agentscope/copaw`（tag 相同）。

```bash
docker pull agentscope/copaw:latest
docker run -p 8088:8088 -v copaw-data:/app/working agentscope/copaw:latest
```

然后在浏览器打开 http://127.0.0.1:8088/ 进入控制台。配置、记忆与 Skills 保存在 `copaw-data` 卷中。

#### 方式五：部署到阿里云 ECS

打开 [CoPaw 阿里云 ECS 部署链接](https://computenest.console.aliyun.com/service/instance/create/cn-hangzhou?type=user&ServiceId=service-1ed84201799f40879884)，按页面提示填写部署参数。

### 初始化

**方式 1：快速用默认配置（不交互）**
```bash
copaw init --defaults
```

**方式 2：交互式初始化**
```bash
copaw init
```

交互流程按顺序配置：
- 心跳 — 间隔、目标、可选活跃时间段
- 工具详情 — 是否在频道消息中显示工具调用细节
- 语言 — Agent 人设文件使用 zh 或 en
- 频道 — 可选配置 iMessage / Discord / DingTalk / Feishu / QQ / Console
- LLM 提供商 — 选择提供商、输入 API Key、选择模型（必选）
- 技能 — 全部启用 / 不启用 / 自定义选择
- 环境变量 — 可选添加工具所需的键值对
- HEARTBEAT.md — 在默认编辑器中编辑心跳检查清单

### 启动服务

```bash
# 默认 127.0.0.1:8088
copaw app

# 自定义地址
copaw app --host 0.0.0.0 --port 9090

# 代码改动自动重载（开发用）
copaw app --reload

# 多 worker 模式
copaw app --workers 4

# 详细日志
copaw app --log-level debug
```

### 控制台

服务启动后，在浏览器打开 `http://127.0.0.1:8088/` 即可进入**控制台** — 一个用于对话、频道、定时任务、技能、模型等的 Web 管理界面。

### 验证安装

```bash
curl -N -X POST "http://localhost:8088/api/agent/process" \
  -H "Content-Type: application/json" \
  -d '{"input":[{"role":"user","content":[{"type":"text","text":"你好"}]}],"session_id":"session123"}'
```

---

## 工作目录结构

默认工作目录：`~/.copaw`

```
~/.copaw/
├── config.json              # 频道开关与鉴权、心跳设置、语言等
├── HEARTBEAT.md             # 心跳每次要问 CoPaw 的内容
├── jobs.json                # 定时任务列表
├── chats.json               # 会话列表（文件存储模式）
├── active_skills/           # 当前激活的技能
├── customized_skills/       # 用户自定义的技能
├── custom_channels/         # 自定义频道模块
├── memory/                  # Agent 记忆文件（自动管理）
│   ├── MEMORY.md            # 长期有效的关键信息
│   └── YYYY-MM-DD.md        # 每日日志
├── SOUL.md                  # （必需）核心身份与行为原则
├── AGENTS.md                # （必需）详细的工作流程、规则和指南
├── PROFILE.md               # 身份和用户画像
└── mcp_clients/             # MCP 客户端配置
```

### 文件说明

| 文件 | 读写属性 | 核心职责 |
|------|----------|----------|
| **SOUL.md** | 只读 | 定义 Agent 的价值观与行为准则 |
| **PROFILE.md** | 读写 | 记录 Agent 的身份和用户画像 |
| **BOOTSTRAP.md** | 一次性（自删除） | 新 Agent 的首次运行引导流程 |
| **AGENTS.md** | 只读 | Agent 的完整工作规范 |
| **MEMORY.md** | 读写 | 存储 Agent 的工具设置与经验教训 |
| **HEARTBEAT.md** | 读写 | 定义 Agent 的后台巡检任务 |

---

## 环境变量配置

### CoPaw 基础环境变量

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `COPAW_WORKING_DIR` | `~/.copaw` | 工作目录 |
| `COPAW_CONFIG_FILE` | `config.json` | 配置文件名（相对工作目录） |
| `COPAW_HEARTBEAT_FILE` | `HEARTBEAT.md` | 心跳问题文件名 |
| `COPAW_JOBS_FILE` | `jobs.json` | 定时任务文件名 |
| `COPAW_CHATS_FILE` | `chats.json` | 会话列表文件名 |
| `COPAW_LOG_LEVEL` | `info` | 日志级别（debug/info/warning/error/critical） |
| `COPAW_MEMORY_COMPACT_THRESHOLD` | `100000` | 触发记忆压缩的字符阈值 |
| `COPAW_MEMORY_COMPACT_KEEP_RECENT` | `3` | 压缩后保留的最近消息数 |
| `COPAW_MEMORY_COMPACT_RATIO` | `0.7` | 触发压缩的阈值比例（相对于上下文窗口大小） |
| `COPAW_CONSOLE_STATIC_DIR` | （自动检测） | 控制台前端静态文件路径 |

### Embedding 配置

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `EMBEDDING_API_KEY` | （空） | Embedding 服务的 API Key |
| `EMBEDDING_BASE_URL` | `https://dashscope.aliyuncs.com/compatible-mode/v1` | Embedding 服务的 URL |
| `EMBEDDING_MODEL_NAME` | `text-embedding-v4` | Embedding 模型名称 |
| `EMBEDDING_DIMENSIONS` | `1024` | 向量维度 |
| `EMBEDDING_CACHE_ENABLED` | `true` | 是否启用 Embedding 缓存 |
| `FTS_ENABLED` | `true` | 是否启用 BM25 全文检索 |

### 记忆存储后端

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `MEMORY_STORE_BACKEND` | `auto` | 记忆存储后端，可选 auto、local、chroma、sqlite |

> **推荐**：配置 `EMBEDDING_API_KEY` 并保持 `FTS_ENABLED=true`，使用向量 + BM25 混合检索以获得最佳效果。

---

## 频道配置

### 支持的频道

| 频道 | 说明 | 凭据字段 |
|------|------|----------|
| **dingtalk** | 钉钉 | `client_id`, `client_secret` |
| **feishu** | 飞书 / Lark | `app_id`, `app_secret`, `encrypt_key`, `verification_token`, `media_dir` |
| **qq** | QQ 机器人 | `app_id`, `client_secret` |
| **discord** | Discord 机器人 | `bot_token`, `http_proxy`, `http_proxy_auth` |
| **imessage** | macOS iMessage | `db_path`, `poll_sec` |
| **telegram** | Telegram 机器人 | `bot_token` |
| **console** | 控制台 | （只需开关） |

### 频道通用字段

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `enabled` | bool | `false` | 是否启用该频道 |
| `bot_prefix` | string | `""` | 可选命令前缀（如 `[BOT]`） |

### 多模态消息支持

| 频道 | 接收文本 | 接收图片 | 接收视频 | 接收音频 | 接收文件 | 发送文本 | 发送图片 | 发送视频 | 发送音频 | 发送文件 |
|------|:--------:|:--------:|:--------:|:--------:|:--------:|:--------:|:--------:|:--------:|:--------:|:--------:|
| 钉钉 | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| 飞书 | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Discord | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | 🚧 | 🚧 | 🚧 | 🚧 |
| iMessage | ✓ | ✗ | ✗ | ✗ | ✗ | ✓ | ✗ | ✗ | ✗ | ✗ |
| QQ | ✓ | 🚧 | 🚧 | 🚧 | 🚧 | ✓ | 🚧 | 🚧 | 🚧 | 🚧 |
| Telegram | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |

> ✓ = 已支持；🚧 = 施工中；✗ = 不支持

---

## config.json 完整结构

```json
{
  "channels": {
    "imessage": {
      "enabled": false,
      "bot_prefix": "",
      "db_path": "~/Library/Messages/chat.db",
      "poll_sec": 1.0
    },
    "discord": {
      "enabled": false,
      "bot_prefix": "",
      "bot_token": "",
      "http_proxy": "",
      "http_proxy_auth": ""
    },
    "dingtalk": {
      "enabled": false,
      "bot_prefix": "",
      "client_id": "",
      "client_secret": ""
    },
    "feishu": {
      "enabled": false,
      "bot_prefix": "",
      "app_id": "",
      "app_secret": "",
      "encrypt_key": "",
      "verification_token": "",
      "media_dir": "~/.copaw/media"
    },
    "qq": {
      "enabled": false,
      "bot_prefix": "",
      "app_id": "",
      "client_secret": ""
    },
    "console": {
      "enabled": true,
      "bot_prefix": ""
    }
  },
  "agents": {
    "defaults": {
      "heartbeat": {
        "every": "30m",
        "target": "main",
        "activeHours": null
      }
    },
    "running": {
      "max_iters": 50,
      "max_input_length": 131072
    },
    "language": "zh",
    "installed_md_files_language": "zh"
  },
  "last_api": {
    "host": "127.0.0.1",
    "port": 8088
  },
  "last_dispatch": null,
  "show_tool_details": true
}
```

### agents.running 配置

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `max_iters` | int | `50` | ReAct Agent 推理-执行循环的最大轮数（必须 ≥ 1） |
| `max_input_length` | int | `131072` (128K) | 模型上下文窗口的最大输入长度（token 数），记忆压缩将在达到此值的 80% 时触发 |

---

## 心跳配置

「心跳」指的是：按固定间隔，用你写好的一段「问题」去问 CoPaw，并可选择把回复发到你上次对话的频道。

### 心跳字段说明

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `every` | string | `"30m"` | 运行间隔。支持 `Nh`、`Nm`、`Ns` 组合 |
| `target` | string | `"main"` | `"main"` = 不发送；`"last"` = 发到上次对话的频道 |
| `activeHours` | object/null | `null` | 可选活跃时段 |

### activeHours 字段（不为 null 时）

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `start` | string | `"08:00"` | 开始时间（HH:MM，24 小时制） |
| `end` | string | `"22:00"` | 结束时间（HH:MM，24 小时制） |

### 与定时任务的区别

| 特性 | 心跳 | 定时任务 (cron) |
|------|------|-----------------|
| 数量 | 只有一份（HEARTBEAT.md） | 可以建很多个 |
| 间隔 | 一个全局间隔 | 每个独立设定时间 |
| 投递 | 可选发到「上次频道」或不发 | 每个独立指定频道和用户 |

---

## 技能 (Skills)

### 内置 Skills 一览

| Skill 名称 | 说明 | 来源 |
|-----------|------|------|
| **cron** | 定时任务管理 | 自建 |
| **file_reader** | 读取与摘要文本类文件 | 自建 |
| **dingtalk_channel_connect** | 辅助完成钉钉频道接入流程 | 自建 |
| **himalaya** | 通过 CLI 管理邮件（IMAP/SMTP） | openclaw |
| **news** | 从指定新闻站点查询最新新闻并做摘要 | 自建 |
| **pdf** | PDF 相关操作 | anthropics/skills |
| **docx** | Word 文档的创建、阅读、编辑 | anthropics/skills |
| **pptx** | PPT 的创建、阅读、编辑 | anthropics/skills |
| **xlsx** | 表格的读取、编辑、创建与格式整理 | anthropics/skills |
| **browser_visible** | 以可见模式启动真实浏览器窗口 | 自建 |

### 管理技能

**通过控制台：**
1. 打开 控制台 → **智能体 → 技能**
2. 启用/禁用技能
3. 新建/编辑自定义技能
4. 导入 Skills Hub 中的技能

**通过 CLI：**
```bash
copaw skills list     # 查看所有技能及启用/禁用状态
copaw skills config   # 交互式启用/禁用技能
```

### 导入 Skill

当前支持在控制台中导入以下四种来源的 Skills：
- `https://skills.sh/...`
- `https://clawhub.ai/...`
- `https://skillsmp.com/...`
- `https://github.com/...`

### 自定义 Skill

在工作目录 `~/.copaw/customized_skills/` 下新建目录，创建 `SKILL.md`：

```markdown
---
name: my_skill
description: 我的自定义能力说明
---
# 使用说明
本 Skill 用于……
```

---

## MCP (模型上下文协议)

MCP (Model Context Protocol) 允许 CoPaw 连接到外部 MCP 服务器并使用它们的工具。

### 前置要求

- Node.js 18 或更高版本（如果使用 `npx` 运行 MCP 服务器）

### 配置格式

**格式 1：标准 mcpServers 格式（推荐）**
```json
{
  "mcpServers": {
    "client-name": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem"],
      "env": {
        "API_KEY": "your-api-key-here"
      }
    }
  }
}
```

**格式 2：直接键值对格式**
```json
{
  "client-name": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-filesystem"],
    "env": {
      "API_KEY": "your-api-key-here"
    }
  }
}
```

**格式 3：单个客户端格式**
```json
{
  "key": "client-name",
  "name": "My MCP Client",
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-filesystem"],
  "env": {
    "API_KEY": "your-api-key-here"
  }
}
```

### 示例：文件系统 MCP 服务器

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/username/Documents"]
    }
  }
}
```

### 管理 MCP 客户端

在 **控制台 → 智能体 → MCP** 中可以：
- 查看所有客户端（以卡片形式）
- 启用/禁用客户端（快速开关）
- 编辑配置（查看和编辑 JSON）
- 删除客户端

---

## 记忆系统

### 架构概览

记忆系统提供两大核心能力：

1. **上下文管理** — 在上下文窗口溢出前，自动将对话压缩为精华摘要
2. **长期记忆管理** — 通过文件工具将关键信息写入 Markdown 文件，配合语义检索随时召回

### 记忆文件结构

| 文件 | 位置 | 用途 |
|------|------|------|
| **MEMORY.md** | `{working_dir}/MEMORY.md` | 长期有效、极少变动的关键信息 |
| **memory/YYYY-MM-DD.md** | `{working_dir}/memory/` | 每天一页，记录当天的工作与交互 |

### 何时写入记忆？

| 信息类型 | 写入目标 | 操作方式 |
|----------|----------|----------|
| 决策、偏好、持久事实 | MEMORY.md | write / edit 工具 |
| 日常笔记、运行上下文 | memory/YYYY-MM-DD.md | write / edit 工具 |
| 上下文溢出自动摘要 | memory/YYYY-MM-DD.md | 自动触发 |
| 用户说「记住这个」 | 立即写入文件 | write 工具 |

### 搜索记忆

| 方式 | 工具 | 适用场景 |
|------|------|----------|
| 语义搜索 | memory_search | 不确定记在哪个文件，按意图模糊召回 |
| 直接读取 | read_file | 已知具体日期或文件路径，精确查阅 |

### 混合检索原理

记忆搜索默认采用**向量 + BM25 混合检索**：
- 向量权重：0.7
- BM25 权重：0.3

两种检索方式互为补充，无论是「自然语言提问」还是「精确查找」，都能获得可靠的召回结果。

---

## 模型提供商

### 内置提供商

| 提供商 | ID | 默认 Base URL |
|--------|-------|---------------|
| ModelScope（魔搭） | `modelscope` | `https://api-inference.modelscope.cn/v1` |
| DashScope（灵积） | `dashscope` | `https://dashscope.aliyuncs.com/compatible-mode/v1` |
| 自定义 | `custom` | （你自己填） |

### 本地提供商

#### llama.cpp / MLX

```bash
# 安装后端
pip install 'copaw[llamacpp]'  # llama.cpp（跨平台）
pip install 'copaw[mlx]'       # MLX（Apple Silicon）

# 下载模型
copaw models download Qwen/Qwen3-4B-GGUF
copaw models download Qwen/Qwen3-4B --backend mlx

# 从 ModelScope 下载
copaw models download Qwen/Qwen2-0.5B-Instruct-GGUF --source modelscope

# 查看已下载模型
copaw models local

# 删除已下载模型
copaw models remove-local <model_id>
```

**选项说明**：
| 选项 | 简写 | 默认值 | 说明 |
|------|------|--------|------|
| `--backend` | `-b` | `llamacpp` | 目标后端（llamacpp 或 mlx） |
| `--source` | `-s` | `huggingface` | 下载源（huggingface 或 modelscope） |
| `--file` | `-f` | （自动） | 指定文件名，省略时自动选择 |

#### Ollama

Ollama 集成本地 Ollama 守护进程，动态加载其中的模型。

**前置条件**：
- 从 [ollama.com](https://ollama.com/) 安装 Ollama
- 安装 Ollama SDK：`pip install 'copaw[ollama]'`

```bash
# 下载 Ollama 模型
copaw models ollama-pull mistral:7b
copaw models ollama-pull qwen2.5:3b

# 查看 Ollama 模型
copaw models ollama-list

# 删除 Ollama 模型
copaw models ollama-remove mistral:7b

# 在配置流程中使用
copaw models config           # 选择 Ollama → 从模型列表中选择
copaw models set-llm          # 切换到其他 Ollama 模型
```

**与本地模型的区别**：
- 模型来自 Ollama 守护进程（不由 CoPaw 下载）
- 使用 `ollama-pull` / `ollama-remove` 而非 `download` / `remove-local`
- 通过 Ollama CLI 或 CoPaw 添加/删除模型时，模型列表自动更新

支持的热门模型：`mistral:7b`、`qwen3:8b` 等

### CLI 管理命令

```bash
copaw models list                    # 看当前状态
copaw models config                  # 完整交互式配置
copaw models config-key modelscope   # 只配 ModelScope 的 API Key
copaw models set-llm                 # 只切换模型
```

---

## CLI 命令参考

### 快速上手

```bash
# 初始化
copaw init --defaults   # 不交互，用默认值
copaw init              # 交互式初始化

# 启动服务
copaw app
```

### 模型管理

```bash
copaw models list                    # 查看所有提供商
copaw models config                  # 完整交互式配置
copaw models config-key <provider>   # 配置 API Key
copaw models set-llm                 # 切换活跃模型
copaw models download <repo_id>      # 下载本地模型
copaw models local                   # 查看已下载模型
copaw models ollama-pull <model>     # 下载 Ollama 模型
```

### 环境变量管理

```bash
copaw env list                       # 列出所有变量
copaw env set KEY VALUE              # 设置变量
copaw env delete KEY                 # 删除变量
```

### 频道管理

```bash
copaw channels list                  # 查看所有频道（密钥脱敏）
copaw channels config                # 交互式配置
copaw channels install <key>         # 安装自定义频道模块
copaw channels add <key>             # 添加频道到 config
copaw channels remove <key>          # 删除自定义频道（--keep-config 保留配置）
```

**交互式 config 流程**：
依次选择频道 → 启用/禁用 → 填写凭据 → 循环直到选择「保存退出」。

| 频道 | 需要填写的字段 |
|------|---------------|
| iMessage | Bot 前缀、数据库路径、轮询间隔 |
| Discord | Bot 前缀、Bot Token、HTTP 代理、代理认证 |
| DingTalk | Bot 前缀、Client ID、Client Secret |
| Feishu | Bot 前缀、App ID、App Secret |
| QQ | Bot 前缀、App ID、Client Secret |
| Console | Bot 前缀 |

### 定时任务

```bash
copaw cron list                      # 列出所有任务
copaw cron create --type text --name "每日早安" --cron "0 9 * * *" \
  --channel dingtalk --target-user "xxx" --text "早上好！"
copaw cron state <job_id>            # 查看运行状态
copaw cron delete <job_id>           # 删除任务
copaw cron pause <job_id>            # 暂停任务
copaw cron resume <job_id>           # 恢复任务
copaw cron run <job_id>              # 立即执行一次
```

### 会话管理

```bash
copaw chats list                     # 列出所有会话
copaw chats get <id>                 # 查看会话详情
copaw chats create --session-id "xxx" --user-id "xxx" --name "My Chat"
copaw chats update <id> --name "新名称"
copaw chats delete <id>              # 删除会话
```

### 技能管理

```bash
copaw skills list                    # 看有哪些技能
copaw skills config                  # 交互式开关
```

### 维护

```bash
copaw clean                          # 清空工作目录（交互确认）
copaw clean --yes                    # 不确认直接清空
copaw clean --dry-run                # 只列出会被删的内容
```

### 全局选项

| 选项 | 默认值 | 说明 |
|------|--------|------|
| `--host` | `127.0.0.1` | API 地址 |
| `--port` | `8088` | API 端口 |
| `-h / --help` | | 显示帮助 |

---

## API 接口

### Agent 处理接口

- **路径**: `POST /api/agent/process`
- **内容类型**: `application/json`
- **支持**: SSE 流式响应

### 请求示例

```bash
curl -N -X POST "http://localhost:8088/api/agent/process" \
  -H "Content-Type: application/json" \
  -d '{
    "input": [
      {
        "role": "user",
        "content": [{"type": "text", "text": "你好"}]
      }
    ],
    "session_id": "session123"
  }'
```

### 配置管理接口

- `GET /config/channels` — 获取全部频道
- `PUT /config/channels` — 整体覆盖
- `GET /config/channels/{channel_name}` — 获取单个频道
- `PUT /config/channels/{channel_name}` — 更新单个频道

---

## 控制台功能

### 侧边栏结构

| 组 | 功能 | 说明 |
|----|------|------|
| 聊天 | 聊天 | 和 CoPaw 对话、管理会话 |
| 控制 | 频道 | 启用/禁用频道、填入凭据 |
| 控制 | 会话 | 筛选、重命名、删除会话 |
| 控制 | 定时任务 | 创建/编辑/删除任务、立即执行 |
| 智能体 | 工作区 | 编辑人设文件、查看记忆、上传/下载 |
| 智能体 | 技能 | 启用/禁用/创建/导入/删除技能 |
| 智能体 | MCP | 启用/禁用/创建/编辑/删除 MCP |
| 智能体 | 运行配置 | 修改最大迭代次数和最大输入长度 |
| 设置 | 模型 | 配置提供商、管理模型、选择模型 |
| 设置 | 环境变量 | 添加/编辑/删除环境变量 |

---

## 官方文档导航

| 页面 | 说明 | 链接 |
|------|------|------|
| 项目介绍 | CoPaw 是什么、能做什么 | http://copaw.agentscope.io/docs/intro |
| 快速开始 | 安装和启动指南 | http://copaw.agentscope.io/docs/quickstart |
| 控制台 | 控制台使用说明 | http://copaw.agentscope.io/docs/console |
| 频道配置 | 钉钉/飞书/QQ/Discord/iMessage 配置 | http://copaw.agentscope.io/docs/channels |
| Skills | 技能扩展说明 | http://copaw.agentscope.io/docs/skills |
| MCP | MCP 客户端配置 | http://copaw.agentscope.io/docs/mcp |
| 记忆 | 记忆系统说明 | http://copaw.agentscope.io/docs/memory |
| 心跳 | 心跳配置说明 | http://copaw.agentscope.io/docs/heartbeat |
| 配置与工作目录 | 详细配置说明 | http://copaw.agentscope.io/docs/config |
| CLI | 命令行工具说明 | http://copaw.agentscope.io/docs/cli |
| FAQ 常见问题 | 社区常见问题汇总 | http://copaw.agentscope.io/docs/faq |
| 问题反馈与交流 | 社区支持 | http://copaw.agentscope.io/docs/community |
| 开源与贡献 | 贡献指南 | http://copaw.agentscope.io/docs/contributing |

---

## 相关项目

- [CoPaw 官方仓库](https://github.com/agentscope-ai/CoPaw) - CoPaw 主项目
- [AgentScope](https://github.com/agentscope-ai/agentscope)
- [AgentScope Runtime](https://github.com/agentscope-ai/agentscope-runtime)
- [ReMe](https://github.com/agentscope-ai/ReMe)
- [OpenClaw](https://openclaw.ai/) - 部分灵感来源
- [anthropics/skills](https://github.com/anthropics/skills) - Agent Skills 规范与示例

---

## 官方 Docker 镜像

CoPaw 官方也提供 Docker 镜像，可直接使用：

```bash
docker pull agentscope/copaw:latest
docker run -p 8088:8088 -v copaw-data:/app/working agentscope/copaw:latest
```

> **注**：本项目（copaw-docker）与官方镜像的主要区别在于：
> - 官方镜像：由 AgentScope 团队维护，简单直接
> - 本项目：增加了更多自动化功能（自动初始化、健康检查）、工作流测试、镜像发布等

---

## 社区支持

如有问题或交流，可通过以下方式联系官方：

| 平台 | 说明 |
|------|------|
| Discord | [加入 Discord 社区](https://discord.gg/agentscope) |
| 钉钉 | 搜索群组加入 |
| GitHub Issues | [提交问题](https://github.com/agentscope-ai/CoPaw/issues) |

---

## License

CoPaw 采用 [Apache License 2.0](https://github.com/agentscope-ai/CoPaw/blob/main/LICENSE) 开源许可。
