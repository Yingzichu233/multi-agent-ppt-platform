# 多智能体 PPT 生成平台（Multi-Agent PPT Platform）

基于多智能体编排的 AI PPT 创作平台。用户只需输入主题，平台即可通过多个 AI 专家智能体协同工作，自动完成大纲规划、内容撰写、联网检索、图表生成、配图搜索与排版美化，最终输出一份可下载的 `.pptx` 演示文稿。

## 功能特性

- **多智能体协同编排**：基于 LangGraph 的 Supervisor（主管）架构，将任务分发给不同专家智能体：
  - 大纲专家（Outline）：规划 PPT 整体结构
  - 内容专家（Content）：撰写每页幻灯片文案
  - 检索专家（Research）：通过 Tavily 联网检索补充实时资料
  - 图表专家（Chart）：使用 Matplotlib 生成数据图表
  - 配图专家（Image）：通过 Pexels API 搜索高质量配图
  - 美化专家（Beautify）：排版与样式美化
- **流式交互**：生成过程实时推送，可查看每个智能体的执行日志
- **大纲人工审阅**：支持 Human-in-the-loop，大纲生成后可人工确认再继续
- **PPT 模板系统**：支持模板选择与自定义模板管理
- **用户体系**：注册、登录、会话管理、历史记录，含管理后台页面
- **可观测性**：可选接入 LangSmith，追踪 LLM 调用与智能体行为

## 技术栈

| 端 | 技术 |
| --- | --- |
| 前端 | Vue 3 · Vite · TypeScript · Ant Design Vue · Pinia · Vue Router · Axios |
| 后端 | Python 3.12 · FastAPI · LangChain · LangGraph · python-pptx · Matplotlib |
| 存储 | MySQL · Redis |
| LLM | DashScope（通义千问），兼容 OpenAI 接口 |

## 项目结构

```
├── ppt生成前端/          # Vue 3 前端
│   ├── src/
│   │   ├── api/         # 后端接口封装
│   │   ├── pages/       # 页面（首页 / 对话生成 / PPT 管理 / 用户 / 管理后台）
│   │   ├── components/  # 通用组件
│   │   ├── stores/      # Pinia 状态管理
│   │   └── router/      # 路由
│   └── vite.config.ts
│
└── ppt生成后端/          # FastAPI 后端
    └── app/
        ├── agents/      # 多智能体（专家智能体 / Supervisor 编排 / workflow 图）
        ├── agent/       # 单智能体实现（ReAct / Plan-Execute）
        ├── tools/       # 智能体工具（PPT 生成 / 图表 / 配图 / 检索等）
        ├── controller/  # API 路由（用户 / 对话 / PPT / 模板）
        ├── services/    # 业务逻辑层
        ├── models/      # 数据库模型
        ├── schemas/     # Pydantic 请求/响应模型
        └── core/        # 配置 / 数据库 / Redis / 异常 / LangSmith
```

## 快速开始

### 环境要求

- Python ≥ 3.12（建议使用 [uv](https://docs.astral.sh/uv/) 管理依赖）
- Node.js ≥ 18
- MySQL ≥ 5.7、Redis ≥ 5

### 1. 启动后端

```bash
cd ppt生成后端

# 安装依赖（二选一）
uv sync
# 或
pip install .

# 配置环境变量：在项目根目录创建 .env（见下方说明）

# 启动服务（默认端口 8180，可在 .env 中修改 SERVER_PORT）
python -m app.main
# 或
uvicorn app.main:app --reload --port 8180
```

启动后可访问接口文档：`http://localhost:8180/docs`

### 2. 启动前端

```bash
cd ppt生成前端
npm install
npm run dev
```

前端开发服务器运行在 `http://localhost:8200`，`/api` 请求会代理到后端（默认 `http://localhost:8001`，可在 `vite.config.ts` 中修改为实际后端地址）。

### 3. 配置环境变量

在 `ppt生成后端/` 目录下创建 `.env` 文件：

```ini
# 服务
SERVER_HOST=0.0.0.0
SERVER_PORT=8180

# MySQL
DB_HOST=localhost
DB_PORT=3306
DB_NAME=ppt_creator
DB_USER=root
DB_PASSWORD=你的密码

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_DB=0
REDIS_PASSWORD=

# 安全
PASSWORD_SALT=随机字符串
TOKEN_MAX_AGE=2592000

# LLM（DashScope 通义千问）
DASHSCOPE_API_KEY=你的API密钥
DASHSCOPE_MODEL=qwen-plus

# 可选：联网检索 / 配图
TAVILY_API_KEY=
PEXELS_API_KEY=

# 可选：多智能体开关与编排模式
USE_MULTI_AGENT=true
AGENT_MODE=workflow            # workflow | subagents
OUTLINE_REVIEW_ENABLED=false   # 大纲生成后是否等待人工审阅

# 可选：LangSmith 追踪（默认关闭）
LANGSMITH_ENABLED=false
LANGSMITH_API_KEY=
```

> ⚠️ `.env` 已在 `.gitignore` 中排除，请勿提交到仓库。

## 常用命令

| 位置 | 命令 | 说明 |
| --- | --- | --- |
| 前端 | `npm run dev` | 启动开发服务器 |
| 前端 | `npm run build` | 类型检查并构建生产包 |
| 前端 | `npm run lint` | ESLint 检查并自动修复 |
| 后端 | `uvicorn app.main:app --reload` | 启动后端开发服务 |

## License

MIT
