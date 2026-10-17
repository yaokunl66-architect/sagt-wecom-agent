# sagt-wecom-agent

[中文](README.md) | **English**

Sagt (销冠智能体, "top-sales agent") is an AI assistant for sales teams working in WeCom (WeChat Work). Built on LangGraph, the agent reads a customer's chat history, WeChat customer-service records, and order data, then generates customer profiles, customer tags, suggested replies, and schedule suggestions for the salesperson. It ships with an admin console and a sidebar web UI.

## Features

- **Customer profile**: combines chat, customer-service, and order records into a single customer profile.
- **Customer tags**: recommends tags for a customer based on a pre-configured tag taxonomy.
- **Reply suggestions**: generates reply suggestions separately for salesperson-customer chats and for WeChat customer-service conversations.
- **Schedule suggestions**: extracts follow-up items from conversations and proposes calendar entries.
- **Intent recognition and small talk**: automatically determines which task the user wants; when the intent is unclear, it falls back to a conversation subgraph.
- **Human confirmation**: task results can be held with LangGraph's `interrupt` mechanism until the user confirms them, and notifications can be sent through a WeCom self-built app.
- **Streaming output**: the sidebar UI streams the agent's responses as they are generated.

## Architecture

```
sagt_sidebar (FastAPI + Web UI, :8000) ┐
sagt_admin   (Flask admin console, :5000) ├──> sagt_agent (LangGraph Server, :2024)
sagt_client  (demo data init / tests)  ┘            │
                                          LLM (OpenAI-compatible API), WeCom API
```

| Directory | Description |
| --- | --- |
| `sagt_agent/` | The core agent. The main graph at `src/graphs/sagt_graph` loads employee, tag-setting, customer, chat, customer-service, and order data in turn, runs intent recognition, then routes to the profile, tag, chat-suggestion, customer-service chat-suggestion, schedule, and small-talk subgraphs. It also contains the LLM client, data storage, WeCom tools, and LangGraph authentication. Runs with `langgraph dev` or `docker compose`. |
| `sagt_client/` | Demo data initialization scripts (tags, employees and customers, WeCom messages, WeChat customer-service messages, orders) and tests that call the agent and the data store. Demo data lives in `store_client/data/`. |
| `sagt_admin/` | Flask admin console for viewing and managing employees, customers, tags, chat records, customer-service records, and orders. |
| `sagt_sidebar/` | Sidebar web app: a FastAPI backend with a vanilla HTML/CSS/JS frontend. Supports login, a bottom menu that triggers each kind of suggestion, streaming display, and `interrupt` confirmation. |

## Tech Stack

Python 3.12, uv, LangGraph / LangChain, FastAPI (uvicorn), Flask, Alibaba Cloud Bailian (OpenAI-compatible API, default model `qwen-plus`), LangSmith / Langfuse (optional), Docker.

## Quick Start

Prerequisites: [uv](https://docs.astral.sh/uv/) and Python 3.12.

### 1. Configure

Each service directory contains a `.env.example`. Copy it to `.env` and fill in real values. `.env` is excluded by `.gitignore`, so never commit it.

```bash
cp sagt_agent/.env.example sagt_agent/.env
cp sagt_client/.env.example sagt_client/.env
cp sagt_admin/.env.example sagt_admin/.env
cp sagt_sidebar/.env.example sagt_sidebar/.env
```

`sagt_agent` needs the LLM `API_KEY` and `BASE_URL`, the demo account used for LangGraph authentication, and the WeCom credentials `WXWORK_CORP_ID`, `WXWORK_APP_ID`, and `WXWORK_APP_SECRET`. The other services mainly need the agent service address and account credentials.

### 2. Start the agent

```bash
cd sagt_agent && ./run_sagt_agent   # runs langgraph dev, listening on port 2024
```

You can also deploy it as a container using `sagt_agent/docker-compose.yml` and `sagt.dockerfile`.

### 3. Initialize demo data

```bash
cd sagt_client
uv run store_client/sagt_demo_init.py
```

### 4. Start the admin console and the sidebar

```bash
cd sagt_admin && ./run_sagt_admin       # http://localhost:5000
cd sagt_sidebar && ./run_sagt_sidebar   # http://localhost:8000
```

## Notes

- This repository contains no secrets or real customer information. All values in `.env.example` are left blank.
- User IDs, customer IDs, and passwords in the test scripts have been replaced with placeholders (such as `your_user_id` and `your_external_userid`). Replace them with your own values before running.
- The conversations and orders under `sagt_client/store_client/data/` are demo data.
-
