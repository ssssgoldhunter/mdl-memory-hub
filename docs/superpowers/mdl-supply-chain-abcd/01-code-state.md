# 当前代码状态

更新时间：2026-04-10 15:30:00 CST

## 1. 已落地

### A

- `scWithdraw` 已增加 `needRuleCheck`
- 提现 `LiteFlow` 已增加 `withDrawRuleCheck`
- 节点当前是清结算查询占位

### C

- 原冻结继续使用业务主流水
- 解冻只认 `orgTransNo`
- 当前代码已支持部分解冻
- 当前代码已支持给 `B useFrozen=true` 提供部分冻结扣款能力
- `B/C` 已收口为同一套冻结池语义：
  - 公共冻结池负责生成本次 `UF/D` 冻结流水
  - 公共冻结池负责回写原始冻结 `F`
  - 账户余额 / `frozenAmt` / 冻结金额变动明细仍在各自业务核心代码中
- 解冻前已补账户侧 `04.frozenAmt` 校验
- 解冻链已复用现有 `accountCheck + slot.refreshCardSubAccount(...)`

### B

- Web 入口：
  - `/api/ums/catering/trans/scDeduction`
- 核心 deduction 链独立
- 复用原 `TransSlot`
- 主交易类型已改为 `D`
- 账户变动明细类型已改为：
  - 付款方 `DC`
  - 收款方 `DR`
- `useFrozen=false`
  - 已补前置余额校验
  - 已补前置 `04` 冻结金额占用
  - 已补账户变动前解冻
  - 已改为同步更新收付款卡
- `useFrozen=true`
  - 直接消费原冻结额度
  - 不再额外走普通冻结 / 解冻动作
  - 已生成 `D` 并回写原始 `F`
  - 已改为同步更新收付款卡
- `B` service 入口已改为先锁付款卡、收款卡
- `B` 两个 after 已改成同步账户变更，不再走收款异步上账
- 扣款成功后仍投递到 `front` deduction topic
- `front` 扣款消费者已改成清结算 API `TODO` 占位

### D

- `TRANSFER_MODE_SWITCH` 已接入
- `02` 接口阶段走：
  - `pre`
  - `batchAddTransferTiData`
- `02` task 侧已新增专用启动入口
- `02` task 侧改为直接扫描明细表
- `02` task 外部只调单接口：
  - `processDetail02`
- `01` 原有上下账保持不动
- `useFrozen/isFrozen` 元数据已透传到 task
- `isFrozen=true` 已调整为“收款卡上账成功后立即写冻结”
- 成功后仍投递到 `front` transfer topic
- `front` 划付消费者已改成清结算 API `TODO` 占位
- `D` 已明确收回划付域，不再继续改原转账链

### front

- `B/D/银行实收` 的 `RocketMQ` 通知结构仍保留
- 扣款、划付、银行实收消费者已登记清结算 API `TODO`
- `B/D/银行实收` 不再以 `notifyUrl + HTTP consumer` 作为当前有效实现口径

### E

- Web 入口：
  - `/consume/transDeductionBatch/batchPreCreate`
  - `/consume/transDeductionBatch/queryPendingDetailPage`
  - `/consume/transDeductionBatch/processDeductionDetail02`
- LiteFlow 预落地链 `chainDeductionPre` 已独立
  - `DeductionBatchPreCreatePack`：校验 + slot 设置
  - `DeductionBatchPreCreate`：落四表交易骨架（status=P）
- 批量受理 `TransDeductionBatchBusinessServiceImpl`：
  - 批量入参校验 + transNo 去重
  - 先落 `trans_deduction_batch_detail`（status=I）
  - 再逐条调 LiteFlow 预落交易骨架
- 单条执行 `DeductionBatchExecuteServiceImpl`（933行）：
  - `processDeductionDetail02(transNo)`
  - task 阶段锁付款卡 + 收款卡（30s）
  - 根据 `useFrozen` 分流普通扣款 / 冻结扣款
  - `useFrozen=N`：前置冻结 → 投递到 `front` → `front` 消费者待接清结算 API → 解冻 → 同步更新收付款卡
  - `useFrozen=Y`：校验原冻结剩余额度 → 投递到 `front` → `front` 消费者待接清结算 API → 消费原冻结额度 → 同步更新收付款卡
  - 收付款账户变更明细类型 `DC/DR`
  - 冻结变动明细复用 `createFrozenDetail(...)`
  - 状态机完整：I → P → S/F
- Task `DeductionBatchTaskJobService`：
  - XXL-Job `deductionBatchTaskJobService`
  - 任务级 Redis 锁（300s）
  - 分页扫 `status in (I, P)` 的明细
  - 逐条调 `processDeductionDetail02`
- `FlowChainEnums` 已新增 `CHAIN_TRANS_DEDUCTION_PRE`
- 测试已通过

## 2. 已清理的错误实现

- `A` 通过 `front` 查本地失败扣款再拦截提现
- `DeductionTransSlot`
- `D02 registerMode02Detail / batchRegisterMode02Detail`
- `D02` 启动放在 `data-batch`
- 原转账链带入 `D` 模式语义
- `B` 使用消费主交易类型 `P`
- `B` 使用消费账户明细类型 `PC/PR`

## 3. 当前仍是占位而非真实业务完成

### A 清结算查询

- `withDrawRuleCheck` 现在只有 `TODO`
- 只是把链路和开关先站住

### 测试

- 只做了聚焦编译
- 还没补自动化测试

## 4. 已验证

已执行：

```bash
mvn -Dmaven.repo.local=/tmp/codex-m2 -pl fund-catering-consume/fund-catering-consume-service,common-core -am -DskipTests compile
```

结果：

- `BUILD SUCCESS`

另：

- 当前确认 `D02` 启动在 `fund-catering-task`
- `fund-catering-data-batch` 不属于本次 `D02` 需求范围
