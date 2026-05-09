# Architecture Draft

## Purpose

本文档定义的是“实现边界”，不是最终技术选型说明。当前仓库还没有代码，因此这里优先回答两个问题：

1. 系统应该分成哪些职责明确的模块。
2. 哪些产品约束必须反映到架构里。

## Architecture Principles

- 先支持短闭环，再扩展复杂学习系统能力。
- 资料解析、AI 能力、复习调度不要耦合成单个黑盒。
- 领域状态流转单独建模，不要散落在页面或 prompt 里。
- 所有 AI 输出都应能追溯到资料上下文。
- 允许首版算法简单，但边界要为后续演进预留空间。

## Suggested System Modules

### 1. Project Workspace

负责：

- 学习项目创建、编辑、删除
- 项目目标与资料归档
- 项目级视图入口

不负责：

- 资料解析细节
- AI 讲解生成逻辑

### 2. Material Ingestion

负责：

- 文件上传或文本粘贴接收
- 文本抽取
- 资料处理状态记录
- 切分前的原始内容归档

不负责：

- 讲解、问答、测验生成

### 3. Learning Unit Segmentation

负责：

- 将资料切分为候选学习单元
- 为单元生成标题与顺序
- 保留单元与原文之间的绑定关系

不负责：

- 生成高可信知识图谱
- 推断复杂长期计划

### 4. Learning Session

负责：

- 学习单元阅读界面
- 讲解触发
- 问答上下文组织
- 学习状态记录

不负责：

- 复习调度算法

### 5. AI Assistance

负责：

- 基于原文的解释生成
- 单元范围问答
- 小测题目生成
- 回答或讲解中的引用约束

不负责：

- 项目管理
- 任务排期

### 6. Quiz And Feedback

负责：

- 单元级题组生成
- 用户答案提交
- 正误反馈、简要解释、原文依据

### 7. Review Scheduler

负责：

- 根据小测和学习结果生成复习任务
- 决定复习时间间隔
- 维护任务状态

### 8. Mastery Tracking

负责：

- 维护掌握状态
- 为学习界面和任务系统提供状态依据

首版建议状态：

- 未开始
- 学习中
- 待复习
- 复习中
- 基本掌握

## Core Entities

### LearningProject

- `id`
- `name`
- `description`
- `goal`
- `createdAt`

### LearningMaterial

- `id`
- `projectId`
- `title`
- `sourceType`
- `processingStatus`
- `rawText`
- `importedAt`

### LearningUnit

- `id`
- `materialId`
- `title`
- `orderIndex`
- `sourceExcerpt`
- `masteryState`

### Quiz

- `id`
- `unitId`
- `generatedAt`

### QuizItem

- `id`
- `quizId`
- `type`
- `question`
- `answerKey`
- `sourceReference`

### ReviewTask

- `id`
- `unitId`
- `scheduledFor`
- `status`
- `triggerReason`

## Primary Flows

### Import To Learning Unit

1. 用户导入资料。
2. 系统抽取文本并记录处理状态。
3. 切分模块生成学习单元。
4. 单元写回项目空间，默认状态为“未开始”。

### Learning Unit To Quiz

1. 用户进入某个学习单元。
2. 系统提供原文视图。
3. 用户触发 AI 讲解或问答。
4. 用户进入小测并提交答案。
5. 系统回写反馈和状态变化。

### Quiz To Review

1. 小测完成后，系统评估表现。
2. 调度模块生成复习任务。
3. 任务进入按天视图。
4. 用户完成复习后，状态继续流转。

## Recommended Initial Code Layout

在没有最终技术栈的前提下，建议先按职责创建目录：

```text
apps/
  web/
  api/
packages/
  domain/
  ingestion/
  ai/
  scheduler/
docs/
  product/
  engineering/
```

如果首版决定采用单体仓库，也建议至少在应用内部保留这几个逻辑模块。

## Important Gotchas

- “基于原文”不是 UI 文案，而是架构要求；必须保留单元与源文本的关联。
- 资料导入成功不等于学习单元可用，处理状态需要拆分。
- 讲解、问答、出题虽然都依赖模型，但不要共用一个无边界的接口。
- 掌握度状态流转应由明确规则驱动，避免页面层直接写死。
- 如果后续接入搜索或知识图谱，应作为新增模块，而不是污染当前 MVP 主链路。
