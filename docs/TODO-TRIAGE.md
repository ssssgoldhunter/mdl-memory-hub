# TODO Triage（2026-03-19）

## 目标
对当前仓库中的 TODO 做一次分类清洗，区分：
- 已完成/过时（可删除）
- 待改造（仍有业务价值）
- 风险开关（不能误删）

## 已完成/过时（已清理）
以下 TODO 文案为历史残留，当前代码已具备 Router + Handle 的工厂/策略分发能力，因此删除注释，保留现有逻辑：

- `front-service` 中“此处应该有工厂类 ... handler中实现”共 22 处
- 涉及文件：
  - `slhy/fund-catering/fund-catering-front/fund-catering-front-service/src/main/java/com/chinaums/erp/slhy/catering/front/service/impl/TransQueryServiceImpl.java`
  - `slhy/fund-catering/fund-catering-front/fund-catering-front-service/src/main/java/com/chinaums/erp/slhy/catering/front/service/impl/TransVerificationServiceImpl.java`
  - `slhy/fund-catering/fund-catering-front/fund-catering-front-service/src/main/java/com/chinaums/erp/slhy/catering/front/service/impl/TransConsumeServiceImpl.java`
  - `slhy/fund-catering/fund-catering-front/fund-catering-front-service/src/main/java/com/chinaums/erp/slhy/catering/front/service/impl/AccountServiceImpl.java`
  - `slhy/fund-catering/fund-catering-front/fund-catering-front-service/src/main/java/com/chinaums/erp/slhy/catering/front/service/impl/FileProcessServiceImpl.java`

## 待改造（仍有效）
以下 TODO 仍有真实改造价值，建议保留并转为明确任务单：

- `task` 模块“按照配置通道处理数据”共 17 处
- 涉及方向：
  - 通道配置加载统一（`jobParam` / 参数表）
  - 通道过滤规则落地（`platformCode/operator/flag`）
  - Job 执行模板抽取（减少重复逻辑）
- 典型文件：
  - `slhy/fund-catering/fund-catering-task/src/main/java/com/chinaums/erp/slhy/catering/task/job/WithDrawBatchJobService.java`
  - `slhy/fund-catering/fund-catering-task/src/main/java/com/chinaums/erp/slhy/catering/task/job/AccountEntryJobService.java`
  - `slhy/fund-catering/fund-catering-task/src/main/java/com/chinaums/erp/slhy/catering/task/job/zx/ZxTransNotifyJobService.java`

## 风险开关（禁止直接删除）
以下 TODO 与安全/一致性有关，应先评审再改，不能按“注释清理”处理：

- `base-service` 中“mac值校验，上线再解开”共 11 处
- 重点文件：
  - `slhy/fund-catering/fund-catering-base/fund-catering-base-service/src/main/java/com/chinaums/erp/slhy/catering/base/service/impl/BasCardAccountTServiceImpl.java`
  - `slhy/fund-catering/fund-catering-base/fund-catering-base-service/src/main/java/com/chinaums/erp/slhy/catering/base/service/impl/BaseAccountSecurityServiceImpl.java`
  - `slhy/fund-catering/fund-catering-base/fund-catering-base-service/src/main/java/com/chinaums/erp/slhy/catering/base/service/impl/AccountServiceImpl.java`

## 建议下一步
1. 先立项 `task` 通道配置统一改造（低风险、高收益）。
2. 对 MAC 校验 TODO 建立专项评审，明确开关开启条件与灰度方案。
