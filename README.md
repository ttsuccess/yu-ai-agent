# yu-ai-agent

基于 **Spring AI + Spring Boot 3** 构建的 AI 智能体平台。包含一个 AI 助手应用（多轮对话、RAG 知识库问答、工具调用、MCP 服务集成）与一个具备自主规划能力的智能体（ReAct 模式），可自主推理、规划并执行任务直至完成目标。

---

## 项目功能

- **AI 助手应用**
  - 多轮对话，支持对话记忆持久化
  - RAG 知识库问答（基于自定义知识库检索增强生成）
  - 工具调用（联网搜索、文件操作、网页抓取、资源下载、终端操作、PDF 生成）
  - MCP 服务调用（模型上下文协议）
- **自主规划智能体 YuManus**
  - 基于 ReAct（Reason + Act）模式，内置推理循环
  - 自主分步规划并执行任务，直至目标完成
- **大模型接入**
  - 阿里云百炼（DashScope / Qwen）接入
  - 本地模型（Ollama）部署
- **SSE 异步流式推送**
- **Knife4j / OpenAPI 3 接口文档**

---

## 技术栈

- Java 21、Spring Boot 3、Maven
- Spring AI 1.0.0 + Spring AI Alibaba 1.0.0.2
- LangChain4j（社区版 DashScope）
- RAG 知识库 + PGvector 向量数据库
- MCP 客户端（SSE + stdio）与 MCP 服务（图片搜索）
- Tool Calling 工具调用
- ReAct Agent 智能体构建
- 阿里云百炼（DashScope SDK，Qwen）+ Ollama 本地模型
- Kryo 序列化（文件会话记忆持久化）
- Jsoup 网页抓取、iText PDF 生成、Knife4j / OpenAPI 3 接口文档
- 前端：Vue 3 + Vite + Vue Router + Axios

---

## 仓库目录结构

```
.
├── src/                                   # 后端（Spring Boot）
│   └── main/
│       ├── java/com/yupi/yuaiagent/
│       │   ├── agent/                     # 智能体实现（ToolCallAgent、ReActAgent、YuManus）
│       │   ├── advisor/                   # 自定义 Spring AI Advisor
│       │   ├── app/                       # 应用服务
│       │   ├── chatmemory/                # 对话记忆（持久化，Kryo 序列化）
│       │   ├── config/                    # 配置类
│       │   ├── constant/
│       │   ├── controller/                # REST 接口（AiController、HealthController）
│       │   ├── demo/                      # Demo / 示例代码
│       │   ├── rag/                       # RAG 流程
│       │   └── tools/                     # 工具实现（WebSearch、PDF、Terminal 等）
│       └── resources/
│           ├── application.yml            # 主配置（端口 8123，/api）
│           ├── application-prod.yml       # 生产环境配置
│           ├── mcp-servers.json           # stdio MCP 服务定义
│           └── document/                  # RAG 知识库文档
├── yu-ai-agent-frontend/                  # 前端（Vue 3 + Vite）
└── yu-image-search-mcp-server/            # MCP 服务：从指定网站搜索图片
```

---

## 快速启动

### 环境要求

- JDK 21
- Maven（或使用内置的 `mvnw`）
- Node.js 16+ 与 npm（前端）
- （可选）PostgreSQL + PGvector（RAG 用）
- （可选）Ollama（本地模型）
- 阿里云百炼 DashScope API Key

### 1. 后端

```bash
# 在 src/main/resources/application.yml 中配置：
#   spring.ai.dashscope.api-key  -> 你的 DashScope key
#   spring.ai.ollama.*           -> 本地模型配置（可选）

./mvnw spring-boot:run
# 或
mvn spring-boot:run
```

- 服务运行于 `http://localhost:8123/api`
- Swagger / Knife4j 文档：`http://localhost:8123/api/swagger-ui.html`

### 2. 前端

```bash
cd yu-ai-agent-frontend
npm install
npm run dev
```

### 3. MCP 图片搜索服务

```bash
cd yu-image-search-mcp-server
./mvnw package
```

打包后的 jar 可在 `src/main/resources/mcp-servers.json` 中引用。

### RAG + PGvector

启用 `application.yml` 中被注释的 `spring.ai.vectorstore.pgvector` 配置，并准备可用的 PostgreSQL + PGvector 数据库。

---

## 配置说明

运行配置位于 `src/main/resources/application.yml`：

| 配置项 | 说明 |
| --- | --- |
| `spring.ai.dashscope.api-key` | 阿里云百炼 DashScope（Qwen）API Key |
| `spring.ai.ollama.base-url` | 本地 Ollama 地址（`http://localhost:11434`） |
| `spring.ai.mcp.client` | MCP 客户端连接（SSE / stdio） |
| `spring.ai.vectorstore.pgvector` | RAG 向量存储 PGvector 配置 |
| `server.port` / `server.servlet.context-path` | `8123` / `/api` |
| `search-api.api-key` | 联网搜索工具使用的搜索 API Key |

> **安全提醒**：请勿提交真实 API Key 或凭据，建议使用环境变量或本地 profile（并加入 `.gitignore`）。
