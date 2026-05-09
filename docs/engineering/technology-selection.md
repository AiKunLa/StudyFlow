# Technology Selection

## Purpose

本文档定义 StudyFlow 当前阶段的推荐技术选型，用于把产品约束、架构边界和实施顺序落成一套可执行的工程方案。

当前仓库仍处于前代码阶段，因此这里优先回答三个问题：

1. 首版应该用什么技术，才能最快验证学习闭环。
2. 哪些技术现在就该定下来，避免后续返工。
3. 哪些能力应该刻意延后，避免把 MVP 做成重平台。

## Selection Principles

- 优先最短学习闭环，而不是最强通用 AI 平台。
- 优先清晰分层和可维护性，而不是一次性引入过多抽象。
- 优先可自托管、可迁移、可追溯的基础设施。
- 优先显式业务规则，不把核心状态流转埋进 prompt 或黑盒工作流。
- 允许首版算法简单，但不牺牲后续扩展边界。

## Recommended Stack

### Frontend

- Framework: `Next.js` with App Router
- UI: `React + TypeScript`
- Styling: `Tailwind CSS`
- Component primitives: `shadcn/ui`
- Server-state fetching: `TanStack Query`
- Local UI state: `Zustand`

为什么这样选：

- 适合快速搭建“项目列表 -> 资料导入 -> 学习空间 -> 小测 -> 任务页”这类产品页面。
- TypeScript 能让前后端接口、状态流转和表单结构更稳定。
- App Router 适合后续加入流式响应、分段渲染和服务端数据获取。
- `TanStack Query` 适合处理处理状态、任务轮询、详情刷新等服务端状态。
- `Zustand` 只保留给轻量本地交互状态，避免前端状态管理过重。

### Backend API

- Framework: `FastAPI`
- Validation and schemas: `Pydantic v2`
- ORM and DB access: `SQLAlchemy 2`
- Migrations: `Alembic`

为什么这样选：

- FastAPI 非常适合文档驱动、结构化 API、异步接口和流式响应场景。
- Pydantic 适合定义学习项目、资料、学习单元、小测和复习任务等明确实体。
- SQLAlchemy 2 和 Alembic 适合在数据模型还会演进的阶段保持迁移可控。

### Database And Search

- Relational database: `PostgreSQL`
- Vector search: `pgvector`
- Keyword retrieval: `PostgreSQL Full Text Search`

为什么这样选：

- 当前核心问题是强关系和强状态，而不是只做向量检索。
- `LearningProject`、`LearningMaterial`、`LearningUnit`、`Quiz`、`ReviewTask`、`MasteryState` 都天然适合关系型建模。
- 首版问答以“当前学习单元原文”为主，项目级 Ask/Search 再逐步引入混合检索即可。
- PostgreSQL 同时覆盖事务、筛选、统计、全文检索和向量检索，能减少系统拆分。

### File Storage

- Object storage interface: `S3-compatible storage`
- Local/self-hosted default: `MinIO`

为什么这样选：

- 资料上传、原始文件保留、导入产物归档都需要独立于应用容器的文件存储。
- S3 兼容接口便于本地开发和后续上云切换。

### Async Jobs

- Broker / cache: `Redis`
- Task queue: `Celery`

建议异步化的任务：

- 文档文本抽取
- 学习单元切分
- embeddings 生成
- 小测批量生成
- 复习任务批处理

为什么这样选：

- 首版已经存在多个天然长任务，不能阻塞主请求。
- `Redis + Celery` 成熟、简单、容易自托管，足以支撑 MVP。
- 现在还不需要把所有任务上升成复杂工作流引擎。

### AI Integration

- Provider abstraction: project-local provider interface
- First provider: `OpenAI`
- APIs to use first:
  - text generation and grounded answers: `Responses API`
  - embeddings: `Embeddings API`

建议保留的 provider interface：

- `generate_explanation`
- `answer_question`
- `generate_quiz`
- `embed_chunks`

为什么这样选：

- 当前最重要的是把“基于原文解释、基于原文问答、基于原文出题”做成稳定能力。
- 首版不需要过早引入复杂多模型管理后台，但接口层要预留扩展点。
- 供应商抽象应该是薄层，不要让 provider 适配复杂度提前吞掉 MVP 节奏。

### Streaming

- Preferred transport: `Server-Sent Events (SSE)`

适用场景：

- AI 讲解流式输出
- 学习单元内问答流式输出
- 简单的长任务状态推送前期也可先用轮询

为什么这样选：

- SSE 比 WebSocket 简单，足够覆盖当前以单向流式文本为主的交互。
- 只有在后续出现强双向实时协作或复杂事件流时，再考虑 WebSocket。

### Tooling

- JavaScript package manager: `pnpm`
- Python package manager and virtualenv workflow: `uv`
- Monorepo layout: keep `apps/` and `packages/` structure from the architecture draft

### Testing

- Backend tests: `pytest`
- Frontend and end-to-end tests: `Playwright`

优先覆盖：

- 资料导入与状态流转
- 学习单元切分结果可用性
- 讲解和问答引用格式
- 小测提交与反馈
- 复习任务生成规则

### Observability

- Structured logging in API and workers
- Tracing and metrics baseline: `OpenTelemetry`

最低要求：

- 每个异步任务有可查询状态
- 每次 AI 调用可关联到项目、资料、单元
- 错误返回统一结构

## Recommended Code Ownership

技术栈需要和既定目录边界对齐：

- `apps/web`
  - 页面、交互、轮询、流式展示
- `apps/api`
  - HTTP API、鉴权入口、响应编排
- `packages/domain`
  - 核心实体、掌握状态、状态流转规则
- `packages/ingestion`
  - 上传、抽取、切分、分块、索引
- `packages/ai`
  - prompt 约束、provider 适配、引用拼装
- `packages/scheduler`
  - 复习任务生成、任务推进规则

## Decisions To Avoid For MVP

### 1. Do Not Start With A Graph Workflow Engine

当前不建议把 `LangGraph` 作为主链路核心依赖。

原因：

- MVP 流程是确定性的产品流程，不是开放式 agent 任务网络。
- 现阶段更重要的是显式状态机、任务队列和引用约束。
- 如果后续增加高级 Transformation、多步研究流程、播客流水线，再评估是否引入图编排。

### 2. Do Not Start With A Niche Primary Database

当前不建议以 `SurrealDB` 作为首版主数据库。

原因：

- 项目当前重点是关系完整性、迁移稳定性、规则查询和任务状态追踪。
- PostgreSQL 在团队协作、迁移、查询生态和自托管稳定性上更稳。

### 3. Do Not Over-Build Multi-Model Management

当前不建议一开始做完整的多供应商模型广场、模型发现中心或复杂凭证编排。

首版建议：

- 先支持一个主 provider
- 设计薄抽象层
- 把“可切换”保留在代码边界，不强求首版 UI 做满

### 4. Do Not Make Vector Search The Product Center

当前不建议把检索系统做成首版核心体验。

原因：

- 学习闭环核心是“按学习单元理解、提问、验证、复习”。
- 当前单元内解释和问答，直接基于绑定原文更可靠。
- 项目级搜索和混合检索更适合在 V1.1 以后增强。

## Initial Data And Runtime Shape

建议至少落地以下基础设施：

- `PostgreSQL`
- `Redis`
- `MinIO`
- `API service`
- `Worker service`
- `Web app`

建议本地开发以 `Docker Compose` 管理这些服务。

## Phase Alignment

### Phase 0

先确定并初始化：

- `Next.js`
- `FastAPI`
- `PostgreSQL`
- `Redis`
- `MinIO`
- monorepo 目录骨架

### Phase 1

优先打通：

- 学习项目 CRUD
- 文件上传 / 文本粘贴
- 资料持久化
- 资料处理状态

### Phase 2

新增：

- 文本抽取 worker
- 学习单元切分
- 单元标题和顺序生成

### Phase 3

新增：

- 学习单元阅读页
- 基于单元原文的讲解
- 基于单元范围的问答
- 引用与来源片段展示

### Phase 4

新增：

- 小测生成
- 即时反馈
- 反馈中的原文依据

### Phase 5

新增：

- 复习任务生成
- 间隔规则
- 掌握状态推进

## Final Recommendation

如果当前就开始搭建代码仓库，推荐用下面这套最小可行组合：

- Frontend: `Next.js + TypeScript + Tailwind + TanStack Query`
- Backend: `FastAPI + Pydantic + SQLAlchemy + Alembic`
- Data: `PostgreSQL + pgvector + Full Text Search`
- Infra: `Redis + Celery + MinIO`
- AI: `OpenAI Responses API + Embeddings API`
- Testing: `pytest + Playwright`

这套方案的目标不是“证明 StudyFlow 是一个通用 AI 平台”，而是先把“可导入、可学习、可提问、可测验、可复习”的最短闭环做稳。
