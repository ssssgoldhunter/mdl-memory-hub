# E Batch Deduction Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 新增 `E` 批量扣款能力，接口阶段先落明细和交易骨架，task 阶段逐条执行，单条执行入口内部按 `useFrozen` 分普通扣款与冻结扣款。

**Architecture:** `E` 参考当前 `B` 的扣款业务语义和 `D02` 的执行模型，但代码链路与单笔 `B` 完全隔离。实现上拆成三块：批量明细 service、预落地专用 liteflow、task 扫描明细并调用单条执行入口。接口阶段固定先由 business service 落 `trans_deduction_batch_detail`，再由 `chainDeductionPre` 的 node 落 `trans_consume_t + trans_consume_sub_t + trans_consume_sub_rec_t` 三类交易骨架，不做真实账务动作；真实账务在单条执行阶段完成。

**Tech Stack:** Spring Boot, LiteFlow, MyBatis-Plus, OpenFeign, XXL-Job, Redis 分布式锁, Maven

---

## File Structure

### 现有文件

- `bwcj/fund-catering-consume/fund-catering-consume-api/src/main/java/com/chinaums/erp/slhy/catering/consume/api/TransConsumeApi.java`
  - 当前消费/扣款对外接口定义，需要新增 `E` 的批量受理与单条执行接口。
- `bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/controller/TransConsumeController.java`
  - `TransConsumeApi` 实现，需接入 `E` 的接口。
- `bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/service/impl/TransConsumeServiceImpl.java`
  - 当前单笔 `B` 入口和锁卡逻辑所在位置，只作为参考，不应承接 `E` 的对外入口或执行入口。
- `bwcj/fund-catering-consume/fund-catering-consume-service/src/main/resources/liteflow/consume.el.xml`
  - 需要新增 `E` 的预落地专用 chain。
- `bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/flow/component/trans/deduction/DeductionTrans.java`
  - 当前单笔 `B` 既做交易骨架落地又做正式交易，`E` 只能参考其字段口径和落表结构，不能直接复用同一执行节点。
- `bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/flow/component/trans/deduction/DeductionTransPack.java`
  - 当前单笔 `B` 的 pack，可参考其基础校验和 `D/DC/DR` 语义设置。
- `bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/flow/component/trans/deduction/DeductionTransAfter.java`
  - `B useFrozen=false` 的真实账务实现。
- `bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/flow/component/trans/deduction/DeductionTransBAfter.java`
  - `B useFrozen=true` 的真实账务实现。
- `bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/domain/TransDeductionBatchDetail.java`
  - `E` 的明细表对象，已具备 `useFrozen`、`orgFrozenTransNo` 等字段，后续需要补状态查询与执行使用。
- `bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/service/TransDeductionBatchDetailService.java`
  - 明细表 service，后续需新增待处理分页查询、状态更新、按 `transNo` 查询等能力。
- `bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/service/impl/TransDeductionBatchDetailServiceImpl.java`
  - 明细表 service 实现。
- `bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/mapper/TransDeductionBatchDetailMapper.xml`
  - 需要补 `E` 的待处理分页查询 SQL 或按状态查询支持。
- `bwcj/fund-catering-task/src/main/java/com/chinaums/erp/slhy/catering/task/job/TransferTi02TaskJobService.java`
  - `D02` 的 task 参考实现，`E` 可参考其“任务级锁 + 分页扫明细 + 单入口执行”模型。
- `bwcj/fund-catering-task/src/main/java/com/chinaums/erp/slhy/catering/task/constant/TaskConstants.java`
  - 需新增 `E` 专用 task lock key。

### 预计新增文件

- `bwcj/fund-catering-consume/fund-catering-consume-api/src/main/java/com/chinaums/erp/slhy/catering/consume/request/ConsumeDeductionBatchPreRequest.java`
  - `E` 批量受理请求对象，承载批量明细入参。
- `bwcj/fund-catering-consume/fund-catering-consume-api/src/main/java/com/chinaums/erp/slhy/catering/consume/request/ConsumeDeductionBatchItemRequest.java`
  - 批量受理单条明细请求对象。
- `bwcj/fund-catering-consume/fund-catering-consume-api/src/main/java/com/chinaums/erp/slhy/catering/consume/request/ConsumeDeductionPreRequest.java`
  - 预落地 liteflow 使用的单条扣款预建骨架请求对象。
- `bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/flow/component/trans/deduction/pre/DeductionTransPrePack.java`
  - `E` 预落地 chain 的 pack。
- `bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/flow/component/trans/deduction/pre/DeductionTransPre.java`
  - `E` 预落地 chain 的核心节点，只落交易骨架不做真实执行。
- `bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/service/TransDeductionBatchBusinessService.java`
  - `E` 的批量业务 service 接口。
- `bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/service/impl/TransDeductionBatchBusinessServiceImpl.java`
  - `E` 批量业务服务实现，负责批量受理、待处理分页、单条执行入口。
- `bwcj/fund-catering-task/src/main/java/com/chinaums/erp/slhy/catering/task/job/DeductionBatchTaskJobService.java`
  - `E` 的 task job，分页扫明细并调用单条执行接口。

### 测试文件

- `bwcj/fund-catering-consume/fund-catering-consume-service/src/test/java/.../TransDeductionBatchBusinessServiceImplTest.java`
  - `E` 预落地和单条执行分流测试。
- `bwcj/fund-catering-task/src/test/java/.../DeductionBatchTaskJobServiceTest.java`
  - `E` task 扫描和跳过逻辑测试。

---

### Task 1: 固定 E 的数据模型与状态查询能力

**Files:**
- Modify: `bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/domain/TransDeductionBatchDetail.java`
- Modify: `bwcj/fund-catering-consume/fund-catering-consume-api/src/main/java/com/chinaums/erp/slhy/catering/consume/request/TransDeductionBatchDetailReq.java`
- Modify: `bwcj/fund-catering-consume/fund-catering-consume-api/src/main/java/com/chinaums/erp/slhy/catering/consume/response/TransDeductionBatchDetailQueryRes.java`
- Modify: `bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/mapper/TransDeductionBatchDetailMapper.xml`
- Modify: `bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/service/TransDeductionBatchDetailService.java`
- Modify: `bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/service/impl/TransDeductionBatchDetailServiceImpl.java`
- Test: `bwcj/fund-catering-consume/fund-catering-consume-service/src/test/java/.../TransDeductionBatchDetailServiceImplTest.java`

- [ ] **Step 1: 写明细模型测试用例**

覆盖这些规则：
- `transNo` 必填
- `useFrozen` 默认 `N`
- `useFrozen = Y` 时 `orgFrozenTransNo` 必填
- `frontStatus/dcStatus/drStatus/status` 初始状态按设计设置

- [ ] **Step 2: 运行测试确认当前实现缺少 E 的状态查询能力**

Run: `mvn -Dmaven.repo.local=/tmp/codex-m2 -pl fund-catering-consume/fund-catering-consume-service -Dtest=TransDeductionBatchDetailServiceImplTest test`
Expected: FAIL，缺少待处理分页查询、按 `transNo` 查询或状态更新方法

- [ ] **Step 3: 完善明细表 service 能力**

新增这些能力：
- 按 `transNo` 查询单条明细
- 分页查询待处理明细
- 按 `transNo` 更新 `frontStatus/dcStatus/drStatus/status/remark`
- 批量受理时统一初始化状态

- [ ] **Step 4: 运行测试验证通过**

Run: `mvn -Dmaven.repo.local=/tmp/codex-m2 -pl fund-catering-consume/fund-catering-consume-service -Dtest=TransDeductionBatchDetailServiceImplTest test`
Expected: PASS

- [ ] **Step 5: 提交**

```bash
git add bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/domain/TransDeductionBatchDetail.java \
  bwcj/fund-catering-consume/fund-catering-consume-api/src/main/java/com/chinaums/erp/slhy/catering/consume/request/TransDeductionBatchDetailReq.java \
  bwcj/fund-catering-consume/fund-catering-consume-api/src/main/java/com/chinaums/erp/slhy/catering/consume/response/TransDeductionBatchDetailQueryRes.java \
  bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/mapper/TransDeductionBatchDetailMapper.xml \
  bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/service/TransDeductionBatchDetailService.java \
  bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/service/impl/TransDeductionBatchDetailServiceImpl.java
git commit -m "feat: add e batch deduction detail state support"
```

### Task 2: 建立 E 的批量受理请求模型与对外接口

**Files:**
- Create: `bwcj/fund-catering-consume/fund-catering-consume-api/src/main/java/com/chinaums/erp/slhy/catering/consume/request/ConsumeDeductionBatchItemRequest.java`
- Create: `bwcj/fund-catering-consume/fund-catering-consume-api/src/main/java/com/chinaums/erp/slhy/catering/consume/request/ConsumeDeductionBatchPreRequest.java`
- Modify: `bwcj/fund-catering-consume/fund-catering-consume-api/src/main/java/com/chinaums/erp/slhy/catering/consume/api/TransConsumeApi.java`
- Modify: `bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/controller/TransConsumeController.java`
- Create: `bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/service/TransDeductionBatchBusinessService.java`
- Create: `bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/service/impl/TransDeductionBatchBusinessServiceImpl.java`
- Test: `bwcj/fund-catering-consume/fund-catering-consume-service/src/test/java/.../TransDeductionBatchBusinessServiceImplTest.java`

- [ ] **Step 1: 写受理接口测试**

覆盖：
- 每条 `transNo` 必传
- `transNo` 不自动生成
- `useFrozen` 未传时默认 `N`
- `useFrozen = Y` 且未传 `orgFrozenTransNo` 时失败

- [ ] **Step 2: 运行测试确认接口和 service 尚不存在**

Run: `mvn -Dmaven.repo.local=/tmp/codex-m2 -pl fund-catering-consume/fund-catering-consume-service -Dtest=TransDeductionBatchBusinessServiceImplTest test`
Expected: FAIL，缺少批量受理接口/实现

- [ ] **Step 3: 新增 E 的批量受理 API 和 service 壳子**

至少补齐：
- `TransConsumeApi` 新接口：批量预落地受理
- `TransConsumeController` 对应实现
- `TransDeductionBatchBusinessService` 接口
- `TransDeductionBatchBusinessServiceImpl` 基础实现骨架

- [ ] **Step 4: 运行测试验证接口校验通过**

Run: `mvn -Dmaven.repo.local=/tmp/codex-m2 -pl fund-catering-consume/fund-catering-consume-service -Dtest=TransDeductionBatchBusinessServiceImplTest test`
Expected: PASS

- [ ] **Step 5: 提交**

```bash
git add bwcj/fund-catering-consume/fund-catering-consume-api/src/main/java/com/chinaums/erp/slhy/catering/consume/request/ConsumeDeductionBatchItemRequest.java \
  bwcj/fund-catering-consume/fund-catering-consume-api/src/main/java/com/chinaums/erp/slhy/catering/consume/request/ConsumeDeductionBatchPreRequest.java \
  bwcj/fund-catering-consume/fund-catering-consume-api/src/main/java/com/chinaums/erp/slhy/catering/consume/api/TransConsumeApi.java \
  bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/controller/TransConsumeController.java \
  bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/service/TransDeductionBatchBusinessService.java \
  bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/service/impl/TransDeductionBatchBusinessServiceImpl.java
git commit -m "feat: add e batch deduction pre accept api"
```

### Task 3: 抽取 E 专用预落地 liteflow

**Files:**
- Create: `bwcj/fund-catering-consume/fund-catering-consume-api/src/main/java/com/chinaums/erp/slhy/catering/consume/request/ConsumeDeductionPreRequest.java`
- Create: `bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/flow/component/trans/deduction/pre/DeductionTransPrePack.java`
- Create: `bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/flow/component/trans/deduction/pre/DeductionTransPre.java`
- Modify: `bwcj/fund-catering-consume/fund-catering-consume-service/src/main/resources/liteflow/consume.el.xml`
- Modify: `bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/flow/component/trans/deduction/DeductionTrans.java`
- Test: `bwcj/fund-catering-consume/fund-catering-consume-service/src/test/java/.../DeductionTransPreTest.java`

- [ ] **Step 1: 写预落地 chain 测试**

覆盖：
- 只落明细和交易骨架
- 不调正式交易
- 不做冻结/解冻
- 主交易状态初始化为 `P`

- [ ] **Step 2: 运行测试确认当前没有预落地专用 chain**

Run: `mvn -Dmaven.repo.local=/tmp/codex-m2 -pl fund-catering-consume/fund-catering-consume-service -Dtest=DeductionTransPreTest test`
Expected: FAIL，缺少 chain 或节点

- [ ] **Step 3: 抽取只建骨架的能力**

实现要点：
- 从 `DeductionTrans` 中抽取“建交易骨架”能力
- 预落地 chain 只做：
  - 基础校验
  - `trans_consume_t / trans_consume_sub_t / trans_consume_sub_rec_t` 预落地
- 不做：
  - `frontTransConsumeFacadeApi.transConsume(...)`
  - `createFrozenBeforeTrade(...)`
  - 真正账户变更

- [ ] **Step 4: 运行测试验证通过**

Run: `mvn -Dmaven.repo.local=/tmp/codex-m2 -pl fund-catering-consume/fund-catering-consume-service -Dtest=DeductionTransPreTest test`
Expected: PASS

- [ ] **Step 5: 提交**

```bash
git add bwcj/fund-catering-consume/fund-catering-consume-api/src/main/java/com/chinaums/erp/slhy/catering/consume/request/ConsumeDeductionPreRequest.java \
  bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/flow/component/trans/deduction/pre/DeductionTransPrePack.java \
  bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/flow/component/trans/deduction/pre/DeductionTransPre.java \
  bwcj/fund-catering-consume/fund-catering-consume-service/src/main/resources/liteflow/consume.el.xml \
  bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/flow/component/trans/deduction/DeductionTrans.java
git commit -m "feat: add e batch deduction pre-liteflow"
```

### Task 4: 实现 E 的批量预落地受理

**Files:**
- Modify: `bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/service/impl/TransDeductionBatchBusinessServiceImpl.java`
- Modify: `bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/service/TransDeductionBatchBusinessService.java`
- Modify: `bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/controller/TransConsumeController.java`
- Test: `bwcj/fund-catering-consume/fund-catering-consume-service/src/test/java/.../TransDeductionBatchBusinessServiceImplTest.java`

- [ ] **Step 1: 写批量预落地测试**

覆盖：
- 先落 `trans_deduction_batch_detail`
- 再落交易骨架四类表
- `trans_consume_t.status = P`
- `trans_consume_sub_rec_t.recStatus = P`
- `trans_consume_sub_rec_t.accChangeStatus = P`

- [ ] **Step 2: 运行测试确认现状不满足“先明细后交易”**

Run: `mvn -Dmaven.repo.local=/tmp/codex-m2 -pl fund-catering-consume/fund-catering-consume-service -Dtest=TransDeductionBatchBusinessServiceImplTest test`
Expected: FAIL

- [ ] **Step 3: 实现批量受理**

关键实现：
- 循环每条明细
- 先落 `trans_deduction_batch_detail`
- 再调用预落地 chain，由 node 落交易骨架
- 明细和交易骨架都挂同一 `transNo`

- [ ] **Step 4: 运行测试验证通过**

Run: `mvn -Dmaven.repo.local=/tmp/codex-m2 -pl fund-catering-consume/fund-catering-consume-service -Dtest=TransDeductionBatchBusinessServiceImplTest test`
Expected: PASS

- [ ] **Step 5: 提交**

```bash
git add bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/service/impl/TransDeductionBatchBusinessServiceImpl.java \
  bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/service/TransDeductionBatchBusinessService.java \
  bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/controller/TransConsumeController.java
git commit -m "feat: implement e batch deduction pre landing"
```

### Task 5: 新增 E 的单条执行入口

**Files:**
- Modify: `bwcj/fund-catering-consume/fund-catering-consume-api/src/main/java/com/chinaums/erp/slhy/catering/consume/api/TransConsumeApi.java`
- Modify: `bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/controller/TransConsumeController.java`
- Modify: `bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/service/TransDeductionBatchBusinessService.java`
- Modify: `bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/service/impl/TransDeductionBatchBusinessServiceImpl.java`
- Test: `bwcj/fund-catering-consume/fund-catering-consume-service/src/test/java/.../TransDeductionBatchBusinessServiceImplTest.java`

- [ ] **Step 1: 写单条执行入口测试**

覆盖：
- 只保留一个外部入口
- 入口内部根据 `useFrozen` 分流
- `useFrozen = N` 走普通扣款
- `useFrozen = Y` 走冻结扣款

- [ ] **Step 2: 运行测试确认当前没有 E 单条执行入口**

Run: `mvn -Dmaven.repo.local=/tmp/codex-m2 -pl fund-catering-consume/fund-catering-consume-service -Dtest=TransDeductionBatchBusinessServiceImplTest test`
Expected: FAIL

- [ ] **Step 3: 新增单条执行接口**

建议命名：
- `processDeductionDetail02(String transNo)`

内部职责：
- 查明细
- 查预落交易骨架
- 校验状态
- 根据 `useFrozen` 调用 `B` 的普通/冻结扣款能力
- 同步更新状态

- [ ] **Step 4: 运行测试验证通过**

Run: `mvn -Dmaven.repo.local=/tmp/codex-m2 -pl fund-catering-consume/fund-catering-consume-service -Dtest=TransDeductionBatchBusinessServiceImplTest test`
Expected: PASS

- [ ] **Step 5: 提交**

```bash
git add bwcj/fund-catering-consume/fund-catering-consume-api/src/main/java/com/chinaums/erp/slhy/catering/consume/api/TransConsumeApi.java \
  bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/controller/TransConsumeController.java \
  bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/service/TransDeductionBatchBusinessService.java \
  bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/service/impl/TransDeductionBatchBusinessServiceImpl.java
git commit -m "feat: add e batch deduction detail execution api"
```

### Task 6: 为 E 独立实现普通扣款与冻结扣款执行链

**Files:**
- Create: `bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/service/EdeductionExecuteService.java`
- Create: `bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/service/impl/EdeductionExecuteServiceImpl.java`
- Create: `bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/flow/vo/EDeductionExecuteVo.java`
- Modify: `bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/service/impl/TransDeductionBatchBusinessServiceImpl.java`
- Test: `bwcj/fund-catering-consume/fund-catering-consume-service/src/test/java/.../EDeductionExecuteServiceTest.java`

- [ ] **Step 1: 写 E 独立执行链测试**

覆盖：
- 不改现有单笔 `transDeduction(...)`
- `E useFrozen = N` 走独立普通扣款执行链
- `E useFrozen = Y` 走独立冻结扣款执行链
- `E` 的单条执行锁属于 task 执行锁

- [ ] **Step 2: 运行测试确认当前 E 还没有独立执行链**

Run: `mvn -Dmaven.repo.local=/tmp/codex-m2 -pl fund-catering-consume/fund-catering-consume-service -Dtest=EDeductionExecuteServiceTest test`
Expected: FAIL

- [ ] **Step 3: 独立实现 E 的两条执行链**

建议做法：
- 新建 `E` 专用执行 service
- 在该 service 内部独立实现：
  - 普通扣款执行链
  - 冻结扣款执行链
- 允许参考 `B` 的字段口径、状态口径、账户变更口径
- 不允许直接复用：
  - `transDeduction(...)`
  - `DeductionTransAfter`
  - `DeductionTransBAfter`
  - `DeductionTrans` 主流程节点

- [ ] **Step 4: 运行测试验证通过**

Run: `mvn -Dmaven.repo.local=/tmp/codex-m2 -pl fund-catering-consume/fund-catering-consume-service -Dtest=EDeductionExecuteServiceTest test`
Expected: PASS

- [ ] **Step 5: 提交**

```bash
git add bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/service/EdeductionExecuteService.java \
  bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/service/impl/EdeductionExecuteServiceImpl.java \
  bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/flow/vo/EDeductionExecuteVo.java \
  bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/service/impl/TransDeductionBatchBusinessServiceImpl.java
git commit -m "feat: add isolated execution chain for e deduction"
```

### Task 7: 新增 E 的 task 扫描与任务级锁

**Files:**
- Create: `bwcj/fund-catering-task/src/main/java/com/chinaums/erp/slhy/catering/task/job/DeductionBatchTaskJobService.java`
- Modify: `bwcj/fund-catering-task/src/main/java/com/chinaums/erp/slhy/catering/task/constant/TaskConstants.java`
- Test: `bwcj/fund-catering-task/src/test/java/.../DeductionBatchTaskJobServiceTest.java`

- [ ] **Step 1: 写 task 测试**

覆盖：
- 有任务级锁
- 分页扫明细
- 只处理待处理明细
- 每条只调一个单条执行入口

- [ ] **Step 2: 运行测试确认当前不存在 E task**

Run: `mvn -Dmaven.repo.local=/tmp/codex-m2 -pl fund-catering-task -Dtest=DeductionBatchTaskJobServiceTest test`
Expected: FAIL

- [ ] **Step 3: 实现 E task**

参考 `TransferTi02TaskJobService`：
- 新增 task lock key
- 分页扫描 `trans_deduction_batch_detail`
- 对每条调用 `processDeductionDetail02`

- [ ] **Step 4: 运行测试验证通过**

Run: `mvn -Dmaven.repo.local=/tmp/codex-m2 -pl fund-catering-task -Dtest=DeductionBatchTaskJobServiceTest test`
Expected: PASS

- [ ] **Step 5: 提交**

```bash
git add bwcj/fund-catering-task/src/main/java/com/chinaums/erp/slhy/catering/task/job/DeductionBatchTaskJobService.java \
  bwcj/fund-catering-task/src/main/java/com/chinaums/erp/slhy/catering/task/constant/TaskConstants.java
git commit -m "feat: add e batch deduction task job"
```

### Task 8: 联调状态机与编译验证

**Files:**
- Modify: `bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/service/impl/TransDeductionBatchBusinessServiceImpl.java`
- Modify: `bwcj/fund-catering-task/src/main/java/com/chinaums/erp/slhy/catering/task/job/DeductionBatchTaskJobService.java`
- Test: `bwcj/fund-catering-consume/fund-catering-consume-service/src/test/java/.../TransDeductionBatchBusinessServiceImplTest.java`

- [ ] **Step 1: 写状态流转测试**

覆盖：
- `frontStatus/dcStatus/drStatus/status` 流转
- 锁失败的保留状态
- `useFrozen=N` 与 `useFrozen=Y` 分流状态

- [ ] **Step 2: 运行测试确认当前状态推进未完整覆盖**

Run: `mvn -Dmaven.repo.local=/tmp/codex-m2 -pl fund-catering-consume/fund-catering-consume-service,fund-catering-task -Dtest=TransDeductionBatchBusinessServiceImplTest,DeductionBatchTaskJobServiceTest test`
Expected: FAIL

- [ ] **Step 3: 补齐状态机和错误收口**

重点补齐：
- 明细预落地初始状态
- 执行中状态推进
- 失败 remark
- 锁失败后的可重试语义

- [ ] **Step 4: 运行聚焦编译验证**

Run: `mvn -Dmaven.repo.local=/tmp/codex-m2 -pl fund-catering-consume/fund-catering-consume-service,fund-catering-task -am -DskipTests compile`
Expected: `BUILD SUCCESS`

- [ ] **Step 5: 提交**

```bash
git add bwcj/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/service/impl/TransDeductionBatchBusinessServiceImpl.java \
  bwcj/fund-catering-task/src/main/java/com/chinaums/erp/slhy/catering/task/job/DeductionBatchTaskJobService.java
git commit -m "feat: finalize e batch deduction state machine"
```
