# 记忆体与源码交叉校验报告

日期：2026-04-27

## 1. 扫描范围

- 源码项目：`D:\workspaces\IdeaProjects_mdl_dep\mdl`
- 记忆体项目：`D:\workspaces\IdeaProjects_mdl_dep\mdl-memory-hub`
- 路由规则：`C:\Users\ssssg\.codex\memories\rules\project-memory-routing.md`
- 源码重点范围：`mdl/fund-catering-*`
- 记忆体重点范围：`README.md`、`CLAUDE.md`、`workflow/*.md`、`modules/*.md`、`docs/**/*.md`、`business-flows/**/*.md`、`architecture/**/*.md`、`conversation-logs/**/*.md`

## 2. 源码事实基线

### 2.1 顶层目录

源码根目录当前存在：

- `.git`
- `.worktrees`
- `api-modules`
- `auth-service`
- `common-core`
- `crawl-service`
- `deploy`
- `docs`
- `file-service`
- `fund-catering-base`
- `fund-catering-consume`
- `fund-catering-data-batch`
- `fund-catering-front`
- `fund-catering-management`
- `fund-catering-report`
- `fund-catering-task`
- `fund-catering-web`
- `gateway-service`
- `gen-service`
- `libsm4`
- `routing-service`
- `starter-modules`
- `system-service`
- `ui-modules`

### 2.2 Maven/Gradle 模块结构

源码是 Maven 多模块项目，扫描到 `55` 个 `pom.xml`，未发现 Gradle 构建文件。

根 `pom.xml` 当前启用模块：

- `common-core`
- `api-modules`
- `starter-modules`
- `gateway-service`
- `auth-service`
- `system-service`
- `file-service`
- `fund-catering-base`
- `fund-catering-consume`
- `fund-catering-report`
- `fund-catering-task`
- `fund-catering-web`
- `fund-catering-front`
- `fund-catering-data-batch`
- `fund-catering-management`
- `crawl-service`
- `routing-service`

`gen-service` 目录存在，但根 `pom.xml` 中该模块为注释状态，当前未启用。

### 2.3 技术版本

根 `pom.xml` 确认：

- Java：`17`
- Spring Boot：`3.2.4`
- Spring Cloud：`2023.0.1`
- Spring Alibaba：`2023.0.1.0`
- LiteFlow：`2.12.4.1`
- RocketMQ：`5.1.3`
- XXL-Job：`3.1.0`

### 2.4 fund-catering 模块 Java 文件数

| 模块 | Java 文件数 |
|------|-------------|
| `fund-catering-base` | 409 |
| `fund-catering-consume` | 583 |
| `fund-catering-data-batch` | 1015 |
| `fund-catering-front` | 229 |
| `fund-catering-management` | 255 |
| `fund-catering-report` | 663 |
| `fund-catering-task` | 88 |
| `fund-catering-web` | 285 |

### 2.5 核心 API / Service / Controller / Job

按 `fund-catering-*` 扫描：

| 模块 | Controller | Api | Service | JobService |
|------|------------|-----|---------|------------|
| `fund-catering-base` | 23 | 20 | 45 | 0 |
| `fund-catering-consume` | 19 | 19 | 57 | 0 |
| `fund-catering-data-batch` | 34 | 21 | 86 | 0 |
| `fund-catering-front` | 13 | 13 | 17 | 0 |
| `fund-catering-management` | 14 | 12 | 28 | 0 |
| `fund-catering-report` | 38 | 39 | 82 | 0 |
| `fund-catering-task` | 0 | 0 | 48 | 26 |
| `fund-catering-web` | 37 | 0 | 1 | 0 |

已定位关键类/入口：

- 账户服务：`BaseAccountServiceApi`、`AccountServiceFacadeApi`、`AccountChangeBatchService`
- 消费接口：`TransConsumeController`、`TransConsumePaController`、`TransConsumeZxController`、`TransQueryController`
- 任务调度：`AutoWithdrawJobService`、`WithDrawBatchJobService`、`TransferAtJobService`、`TransferRecallJobService`、`DeductionBatchTaskJobService`、`PlatformRechargeJobService`
- 前置消息消费：`HttpConsumeMessageConsumeHandle`、`HttpDeductionMessageConsumeHandle`、`HttpTransferMessageConsumeHandle`、`HttpWithDrawMessageConsumeHandle`、`HttpActualReceiptMessageConsumeHandle`

### 2.6 LiteFlow 链路

`fund-catering-consume-service/src/main/resources/liteflow/consume.el.xml` 和 `query.el.xml` 中存在主要链路，包括：

- `chainWithDraw`
- `chainTransfer`
- `chainTransferInner`
- `chainTransferInnerPre`
- `chainTransferTiPre`
- `chainTransferAuth`
- `chainTransferReSendVerification`
- `chainRecharge`
- `chainRefundRecharge`
- `chainConsume`
- `chainConsumeAuth`
- `chainConsumeRefund`
- `chainFrozen`
- `chainUnFrozen`
- `chainDeduction`
- `chainDeductionPre`
- `chainConsumePre`
- `chainConsumePreFinish`
- `chainConsumeClose`
- `chainConsumeCal`
- `chainConsumeCalOrder`
- `chainWithdrawResultQuery`
- `chainRechargeResultQuery`
- `chainFrozenDetailQuery`
- `chainConsumeResultQuery`
- `chainTransferResultQuery`

## 3. 记忆体文档清单

记忆体项目当前包含 `115` 个 Markdown 文档。重点目录：

- 根入口：`README.md`、`CLAUDE.md`
- 工作流：`workflow/PROJECT_MEMORY.md`、`workflow/DOCUMENT_MANAGEMENT_RULES.md`
- 模块说明：`modules/*.md`
- 设计/参考：`docs/**/*.md`
- 业务流程：`business-flows/**/*.md`
- 架构：`architecture/**/*.md`
- 历史记录：`conversation-logs/**/*.md`
- 需求：`requirements/**/*.md`
- 项目技能：`skills/**/*.md`
- 技术决策：`technical-decisions/**/*.md`

## 4. 校验通过项

- 源码项目和记忆体项目路径均存在。
- `project-memory-routing.md` 已有 `mdl` 项目路由规则，指向 `D:\workspaces\IdeaProjects_mdl_dep\mdl-memory-hub`；但仍需补充源码路径到记忆体路径的精确映射。
- `fund-catering-data-batch` 在源码目录和根 `pom.xml` 中均存在且启用，不应被误删或误排除。
- `gen-service` 在源码目录存在，但根 `pom.xml` 注释未启用，应在文档中明确为“目录存在、当前未启用”。
- 主要交易 LiteFlow 链路可在源码 XML 中找到。
- 账户变动当前实现可在 `BaseAccountServiceApi` / `AccountChangeBatchService` 找到。

## 5. 发现的问题

### 5.1 路由规则缺少精确源码路径映射

`C:\Users\ssssg\.codex\memories\rules\project-memory-routing.md` 已说明 `mdl` 使用 `mdl-memory-hub`，但应补充明确规则：

- `D:\workspaces\IdeaProjects_mdl_dep\mdl` 加载 `D:\workspaces\IdeaProjects_mdl_dep\mdl-memory-hub`

### 5.2 入口文档目录结构过期

`README.md` 仍列出不存在目录：

- `api-docs/`
- `knowledge-base/`
- `common-issues/`

当前记忆体实际有：

- `architecture`
- `bugs`
- `business-flows`
- `conversation-logs`
- `docs`
- `modules`
- `requirements`
- `skills`
- `technical-decisions`
- `workflow`

### 5.3 文档管理规则目录结构过期

`workflow/DOCUMENT_MANAGEMENT_RULES.md` 仍列出不存在或已删除内容：

- `docs/MIGRATION_LOG.md`
- `business-flows/TRANSACTION_FLOWS.md`
- `project-overview/PROJECT_INFO.md`
- `workflow/USER_PREFERENCES.md`
- `knowledge-base/`

### 5.4 项目路径仍是旧 Mac 路径

当前入口/参考文档仍存在 `/Users/limeng/workspaces/IdeaProjects_mdl_dep/...`。当前事实应改为 Windows DEP 路径或仓库相对路径：

- `D:\workspaces\IdeaProjects_mdl_dep\mdl`
- `D:\workspaces\IdeaProjects_mdl_dep\mdl-memory-hub`

历史会话日志中的旧路径不应改写原文。

### 5.5 模块文件数过期

| 文档位置 | 现文档 | 源码事实 |
|----------|--------|----------|
| `CLAUDE.md` | base 399、front 214、management 167、task 217、data-batch 100+、report 50+ | base 409、front 229、management 255、task 88、data-batch 1015、report 663 |
| `modules/MODULE_BASE.md` | 399 Java 文件 | 409 |
| `modules/MODULE_FRONT.md` | 214 Java 文件 | 229 |
| `modules/MODULE_MANAGEMENT.md` | 167 Java 文件 | 255 |
| `modules/MODULE_TASK.md` | 217 Java 文件 | 88 |
| `modules/MODULE_DATA_BATCH.md` | 100+ Java 文件 | 1015 |
| `modules/MODULE_REPORT.md` | 50+ Java 文件 | 663 |

### 5.6 技术版本表达不够精确

`CLAUDE.md` 中 LiteFlow 写为“最新版”，且缺少 Spring Cloud / Spring Alibaba / RocketMQ / XXL-Job 等当前源码版本，应按根 `pom.xml` 更新。

### 5.7 Karpathy/AI 编码准则需要同步到项目记忆体

`CLAUDE.md` 已有部分 AI 工作规范，但应补充/强化：

- 少假设，需求不清先说明
- 最小改动，不做无关重构
- 源码优先，文档以源码事实为基线
- 可验证，每次结论要有扫描或命令依据
- 历史设计和当前事实分开，历史正文不改写，只加状态说明

### 5.8 当前参考文档有旧绝对路径

以下当前参考文档存在旧 Mac 绝对路径，应改为当前 Windows 路径或相对路径：

- `docs/ACCOUNT_CHANGE_SOURCE_MAP.md`
- `docs/NOTIFICATION_OBJECTS_FOR_RECON.md`
- `docs/SUPPLY_CHAIN_DESIGN_V5.5.md`
- `docs/TRANSACTION_QUICK_REFERENCE.md`
- `skills/LITEFLOW_SKILLS.md`
- `workflow/PROJECT_MEMORY.md`

### 5.9 历史设计/计划与当前实现混在一起

账户变动历史设计文档中仍有独立 `AccountChangeApi` 方案描述。当前源码事实是扩展 `BaseAccountServiceApi` 并由 `AccountChangeBatchService` 支撑。建议只在文档开头增加状态说明，不改历史正文。

需要处理的文档：

- `docs/superpowers/specs/2026-03-11-account-change-refactor-design.md`
- `docs/superpowers/specs/2026-03-12-account-change-refactor-todo.md`

### 5.10 历史日志包含旧路径和旧项目名

`conversation-logs/**/*.md` 中包含旧路径、旧项目名、已删除目录等历史记录。该类文档属于历史事实，不建议为当前事实改写原文。

## 6. 需要修正的文档列表

建议修正：

- `C:\Users\ssssg\.codex\memories\rules\project-memory-routing.md`
- `README.md`
- `CLAUDE.md`
- `workflow/PROJECT_MEMORY.md`
- `workflow/DOCUMENT_MANAGEMENT_RULES.md`
- `modules/MODULE_BASE.md`
- `modules/MODULE_FRONT.md`
- `modules/MODULE_MANAGEMENT.md`
- `modules/MODULE_TASK.md`
- `modules/MODULE_DATA_BATCH.md`
- `modules/MODULE_REPORT.md`
- `docs/ACCOUNT_CHANGE_SOURCE_MAP.md`
- `docs/NOTIFICATION_OBJECTS_FOR_RECON.md`
- `docs/SUPPLY_CHAIN_DESIGN_V5.5.md`
- `docs/TRANSACTION_QUICK_REFERENCE.md`
- `skills/LITEFLOW_SKILLS.md`
- `docs/superpowers/specs/2026-03-11-account-change-refactor-design.md`
- `docs/superpowers/specs/2026-03-12-account-change-refactor-todo.md`

## 7. 不建议直接修改的历史文档

- `conversation-logs/**/*.md`：历史会话记录保留原文。
- 历史计划正文：不删除旧方案，只加状态说明。
- Java 包名中的 `com.chinaums.erp.slhy...`：这是历史命名空间，不代表仓库目录错误。
- `docs/plans/2026-03-31-e-batch-deduction-implementation-plan.md` 等 bwcj/lsym 迁移计划：属于历史计划，除非追加状态说明，不应重写路径正文。

## 8. 建议修复顺序

1. 补充 Codex 项目记忆路由精确映射。
2. 更新 `README.md` 的目录结构和项目路径。
3. 更新 `CLAUDE.md` 的路径、技术版本、模块统计、AI 编码准则。
4. 更新 `workflow/PROJECT_MEMORY.md` 和 `workflow/DOCUMENT_MANAGEMENT_RULES.md`。
5. 更新 `modules/*.md` 中模块文件数。
6. 修正当前参考文档中的旧绝对路径。
7. 给历史设计/待办文档增加状态说明，不改历史正文。
8. 再次检查 Markdown 链接、路径、模块统计、技术版本和旧路径残留。