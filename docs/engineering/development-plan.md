# Development Plan

## Purpose

本文档把 StudyFlow 的产品范围、架构边界和技术选型转成一份可执行的开发计划。

与 [implementation-plan.md](D:/Code/VSCode_Project/StudyFlow/docs/engineering/implementation-plan.md) 相比，这份文档更关注：

- 开发先后顺序
- 每阶段的明确输出
- 模块依赖关系
- 验收口径
- 开发过程中的范围控制

当前目标不是做一套完整学习平台，而是以最短路径验证以下闭环：

1. 导入资料
2. 切分学习单元
3. 基于原文讲解和问答
4. 完成小测
5. 生成复习任务

## Planning Assumptions

- 仓库当前仍处于前代码阶段。
- 首版技术栈采用 [technology-selection.md](D:/Code/VSCode_Project/StudyFlow/docs/engineering/technology-selection.md) 中的推荐方案。
- 首版用户聚焦自学型知识工作者和考证型学习者。
- 首版资料类型优先支持 `PDF`、`Markdown` 和纯文本。
- 首版不引入全网搜索、知识图谱、复杂长期计划和精确掌握度评分。

## Development Principles

- 先打通端到端闭环，再补丰富能力。
- 先做稳定状态流转，再做更智能的策略。
- 先做单项目单用户路径，再考虑更复杂的账户、协作和权限。
- 先把“基于原文”做扎实，再扩展搜索和跨资料能力。
- 每个阶段都必须有可演示、可验收的产出，而不是只完成底层准备。

## Delivery Structure

建议按以下三层组织开发任务：

### 1. Foundation

负责：

- 仓库结构
- 基础环境
- 数据模型
- API 基础能力
- 异步任务基础设施

### 2. Core Learning Loop

负责：

- 导入资料
- 学习单元切分
- 学习空间
- AI 讲解与问答
- 小测和反馈
- 复习任务

### 3. Hardening

负责：

- 测试
- 监控
- 错误处理
- 文档完善
- MVP 演示准备

## Phase 0: Project Initialization

### Goal

建立可以开始正式开发的工程骨架。

### Main Deliverables

- 建立目录结构：
  - `apps/web`
  - `apps/api`
  - `packages/domain`
  - `packages/ingestion`
  - `packages/ai`
  - `packages/scheduler`
- 初始化前端项目和后端项目
- 建立基础开发命令和 README 说明
- 确认数据库、缓存、对象存储和 worker 服务的本地开发形态
- 建立基础环境变量模板

### Exit Criteria

- 团队可以在本地启动 web、api 和基础依赖服务
- 代码仓库结构与文档约定一致
- 后续阶段不再需要反复讨论首版技术栈

## Phase 1: Project And Material Foundation

### Goal

让用户能够创建学习项目并导入资料，系统能够记录资料状态。

### Main Deliverables

- 学习项目数据模型和 CRUD API
- 学习项目列表页和详情入口
- 资料上传接口和文本粘贴导入接口
- 学习资料对象持久化
- 资料处理状态字段和状态展示
- 原始文件存储策略

### Suggested Task Breakdown

- `packages/domain`
  - 定义 `LearningProject`、`LearningMaterial`、状态枚举
- `apps/api`
  - 提供项目和资料基础 API
- `apps/web`
  - 提供项目创建、列表查看、资料导入页面
- `packages/ingestion`
  - 接收资料、写入原始记录、提交后续处理任务

### Exit Criteria

- 用户可在 1 分钟内创建学习项目并导入一份资料
- 用户能看到资料名称、导入时间和处理状态
- 失败导入不会导致项目不可继续使用

## Phase 2: Learning Unit Pipeline

### Goal

把导入资料转化为可浏览的学习单元。

### Main Deliverables

- 文本抽取流程
- 学习单元切分逻辑
- 单元标题和排序生成
- 学习单元与原文片段绑定
- 项目内学习单元列表

### Suggested Task Breakdown

- `packages/ingestion`
  - 文本抽取器
  - 学习单元切分器
  - 切分结果持久化
- `packages/domain`
  - 定义 `LearningUnit`
  - 定义材料到单元的关系
- `apps/api`
  - 提供学习单元读取接口
- `apps/web`
  - 提供项目内学习单元列表和进入入口

### Exit Criteria

- 用户导入资料后可以看到一组基本可读的学习单元
- 每个学习单元都能回溯到对应原文内容
- 单元顺序足以支持首版按顺序学习

## Phase 3: Learning Session And AI Assistance

### Goal

打通“阅读原文 + AI 讲解 + 单元内问答”的学习空间。

### Main Deliverables

- 学习单元阅读页
- 基于当前单元原文的“解释这一段”或“讲解本单元”
- 基于当前单元和项目范围的问答
- 回答中的引用来源和原文片段
- 流式响应体验

### Suggested Task Breakdown

- `packages/ai`
  - 讲解 prompt
  - 问答 prompt
  - 引用格式约束
  - provider 接口和 OpenAI 实现
- `apps/api`
  - 讲解接口
  - 问答接口
  - SSE 输出
- `apps/web`
  - 学习空间页面
  - 原文、讲解、问答交互
- `packages/domain`
  - 学习行为事件记录

### Exit Criteria

- 用户能在当前学习单元页面直接获得基于原文的解释
- 用户能连续追问而不频繁跳出当前学习语境
- AI 输出中能展示可核验的来源信息

## Phase 4: Quiz And Feedback

### Goal

为已学习单元增加即时验证能力。

### Main Deliverables

- 单元级小测生成
- 单选、判断和简答中的最小可用题型组合
- 用户答题提交
- 正误反馈和简要解释
- 反馈中的原文依据

### Suggested Task Breakdown

- `packages/ai`
  - 出题 prompt
  - 题目结构化输出
  - 简答反馈生成
- `packages/domain`
  - 定义 `Quiz`、`QuizItem`、提交结果结构
- `apps/api`
  - 生成小测接口
  - 提交答案接口
- `apps/web`
  - 小测入口、答题表单、反馈视图

### Exit Criteria

- 用户能在单元学习后发起一次小测
- 用户提交后能立即获得反馈
- 小测反馈可回溯到原文依据

## Phase 5: Review Loop

### Goal

让学习结果真正回流到后续任务中，形成闭环。

### Main Deliverables

- 复习任务生成规则
- 基于答题表现的间隔调整
- 待复习任务列表
- 掌握状态流转
- 复习完成后的状态推进

### Suggested Task Breakdown

- `packages/scheduler`
  - 复习任务生成逻辑
  - 基础间隔规则
- `packages/domain`
  - `MasteryState`
  - 状态流转规则
- `apps/api`
  - 任务列表和任务完成接口
- `apps/web`
  - 今日任务或待复习列表页面

### Exit Criteria

- 用户完成小测后能看到后续复习任务
- 用户在第二天或后续进入系统时能看到待复习内容
- 掌握状态能随学习和复习行为推进

## Cross-Phase Workstreams

这些工作横跨多个阶段，不应等到最后才补：

### API Contract

- 统一请求和响应结构
- 统一错误结构
- 长任务状态返回格式

### Data Migration

- 所有核心实体的数据库迁移都要可重复执行
- 结构演进要通过迁移管理，而不是手工改表

### Observability

- API 和 worker 都需要结构化日志
- 长任务必须可定位到项目、资料和单元
- AI 调用失败要能区分 provider 问题和业务输入问题

### Testing

- 每个阶段结束前都至少补齐对应的关键路径测试
- 优先保障主闭环测试，而不是外围页面覆盖率

## Recommended Milestones

### Milestone A: Importable

完成 Phase 0 到 Phase 1。

标志：

- 用户能创建项目并导入资料
- 系统能记录处理状态

### Milestone B: Learnable

完成 Phase 2。

标志：

- 资料能转为可浏览学习单元

### Milestone C: Explainable

完成 Phase 3。

标志：

- 用户能基于原文获得讲解和问答

### Milestone D: Verifiable

完成 Phase 4。

标志：

- 用户能完成一次轻量小测并拿到反馈

### Milestone E: Loop Closed

完成 Phase 5。

标志：

- 学习结果能自动转成复习任务

## Acceptance Checklist

在进入 MVP 可演示状态前，应确认：

- 用户可以独立完成一次“导入 -> 学习 -> 提问 -> 小测 -> 复习任务”的流程
- 所有 AI 讲解和问答都尽量绑定原文片段
- 小测反馈可以解释正误，而不是只返回对错
- 复习任务不是静态占位，而是真正由学习结果驱动
- 系统没有为了扩展性而提前引入明显超出 MVP 的复杂度

## Scope Guardrails

开发过程中如果出现以下倾向，应暂停扩展并回到 PRD 做收缩：

- 想在主链路里加入全局资料搜索
- 想自动建立高可信知识依赖图
- 想做复杂长期学习计划系统
- 想把掌握度建模成精确数值评分
- 想同时兼容多个差异极大的学习场景
- 想过早做重型模型市场和复杂凭证管理

## Open Decisions To Resolve During Development

这些问题会影响实现细节，但不应阻塞 Phase 0 启动：

1. 首版 PDF 文本抽取是否只采用单一抽取器，还是保留可切换抽取策略。
2. 学习单元默认粒度优先按标题切分，还是标题加长度混合切分。
3. 小测简答题首版是否做自动评分，还是只给参考答案和解释。
4. 复习任务首版展示为“今日任务”还是“待复习列表”。
5. 学习项目是否首版就支持目标字段的富文本表达。

## Suggested Immediate Next Step

如果要从文档进入实际搭建，推荐按以下顺序执行：

1. 完成 Phase 0，搭起目录骨架和基础运行环境。
2. 直接推进 Phase 1 和 Phase 2，优先拿到“导入后可学习”能力。
3. 再推进 Phase 3 到 Phase 5，完成最短学习闭环。

在这个顺序下，团队会更早看到真实产品，而不是长期停留在基础设施建设阶段。
