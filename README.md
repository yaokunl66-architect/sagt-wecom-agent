# sagt-wecom-agent

Sagt(销冠智能体)是一个面向企业微信销售场景的 AI 助手:基于 LangGraph 构建智能体,读取客户的聊天记录、微信客服记录和订单数据,为销售人员生成客户画像、客户标签、聊天话术建议和日程建议,并配套管理后台与侧边栏 Web 界面。

## 功能

- **客户画像**:综合聊天、客服和订单记录生成客户画像。
- **客户标签**:按预先配置的标签体系为客户推荐标签。
- **聊天建议**:分别针对销售与客户的聊天、微信客服会话生成回复建议。
- **日程建议**:从对话中提取跟进事项,生成日程建议。
- **意图识别与闲聊**:自动判断用户要执行的任务,意图不明确时进入对话子图。
- **人工确认**:任务结果可通过 interrupt 机制由用户确认后再生效,并可通过企业微信自建应用发送通知。
- **流式输出**:侧边栏界面以流式方式展示智能体的返回内容。

## 架构

```
 sagt_sidebar (FastAPI + Web UI, :8000) ┐
 sagt_admin   (Flask 管理后台, :5000)   ├──> sagt_agent (LangGraph Server, :2024)
 sagt_client  (演示数据初始化 / 测试)    ┘        │
                                           LLM(OpenAI 兼容接口)、企业微信 API
```

| 目录 | 说明 |
| --- | --- |
| `sagt_agent/` | 核心智能体。`src/graphs/sagt_graph` 是主图,依次加载员工、标签设置、客户、聊天、客服、订单数据,做意图识别后分发到画像、标签、聊天建议、客服聊天建议、日程和闲聊子图;另含 LLM 调用、数据存储、企业微信工具和 LangGraph 鉴权。支持 `langgraph dev` 与 docker compose 部署。 |
| `sagt_client/` | 演示数据初始化脚本(标签、员工与客户、企业微信消息、微信客服消息、订单)以及智能体和数据存储的调用测试,`store_client/data/` 下是演示数据。 |
| `sagt_admin/` | Flask 管理后台:员工、客户、标签、聊天记录、客服记录、订单的查看与管理。 |
| `sagt_sidebar/` | 侧边栏 Web 应用:FastAPI 后端加原生 HTML/CSS/JS 前端,支持登录、底部菜单触发各类建议、流式展示与 interrupt 确认。 |

## 技术栈

Python 3.12、uv、LangGraph / LangChain、FastAPI(uvicorn)、Flask、阿里云百炼(OpenAI 兼容接口,默认 qwen-plus)、LangSmith / Langfuse(可选)、Docker。

## 快速开始

需要先安装 [uv](https://docs.astral.sh/uv/) 和 Python 3.12。

### 1. 准备配置

每个服务目录下都有 `.env.example`,复制为 `.env` 并填写实际值。`.env` 已被 `.gitignore` 排除,请勿提交。

```bash
cp sagt_agent/.env.example   sagt_agent/.env
cp sagt_client/.env.example  sagt_client/.env
cp sagt_admin/.env.example   sagt_admin/.env
cp sagt_sidebar/.env.example sagt_sidebar/.env
```

`sagt_agent` 需要填写大模型的 `API_KEY` 与 `BASE_URL`、LangGraph 鉴权用的演示账号,以及企业微信的 `WXWORK_CORP_ID`、`WXWORK_APP_ID`、`WXWORK_APP_SECRET`。其余服务主要填写智能体服务地址和账号密码。

### 2. 启动智能体

```bash
cd sagt_agent && ./run_sagt_agent      # langgraph dev,监听 2024 端口
```

也可以使用 `sagt_agent/docker-compose.yml` 与 `sagt.dockerfile` 以容器方式部署。

### 3. 初始化演示数据

```bash
cd sagt_client
uv run store_client/sagt_demo_init.py
```

### 4. 启动管理后台与侧边栏

```bash
cd sagt_admin   && ./run_sagt_admin     # http://localhost:5000
cd sagt_sidebar && ./run_sagt_sidebar   # http://localhost:8000
```

## 说明

- 本仓库不包含任何密钥或真实客户信息;`.env.example` 中的值均已留空。
- 测试脚本中的用户 ID、客户 ID 和密码已替换为占位符(如 `your_user_id`、`your_external_userid`),运行前请改成你自己的值。
- `sagt_client/store_client/data/` 下的对话和订单为演示数据。
