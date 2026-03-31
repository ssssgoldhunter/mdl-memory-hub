# 需求 D 讨论日志

更新时间：2026-03-31 10:20:00 CST

> 说明：本文档保留讨论过程。  
> 其中关于 `D02` 两接口执行、非 task 启动、`oldFrozenTransNo` 等旧口径，已经被后续代码与基线文档纠偏覆盖。

## 2026-03-26 10:35:00 CST

### 讨论背景

在 `需求 C/B` 已形成基础结论后，继续分析 `需求 D` 的新划付模式，重点确认 `02` 模式与 `C/B` 的复用关系。

### 已确认事实

1. `需求 D` 的 `02` 模式不是走外部银行交易，而是内部上下账
2. `02` 模式下，接口阶段已经会落：
   - 转账表数据
   - 划付流水表数据
3. 接口阶段尚未发生账户变动，实际划付由 task 执行

### 用户确认的关键边界

- `02` 模式沿用 `01` 的划付明细新增接口
- `02` 模式下 `batchNo` 不需要强校验
- task 才是实际执行划付的人
- `isFrozen=true` 时，整笔划付金额都需要冻结，不做比例配置

### 已确认的冻结模型

- `02 + isFrozen=true` 时，一笔划付成功对应一笔独立原冻结 `F`
- 不跨多笔划付做合并冻结池
- 后续冻结代扣、解冻都根据原流水号一一对应

### 已确认的复用链路

- `需求 D` 生产标准 `F`
- `需求 B` 通过 `D` 记录消耗这笔 `F`
- `需求 C` 通过 `UF` 释放剩余额度

示例：

- 划付 `100`
- 冻结 `100`
- 后续 `B` 扣款 `20`
- 再由 `C` 释放剩余 `80`

### 已确认的失败处理方向

- `trans_transfer_ti_batch_detail` 的三个状态字段可用于承载不同阶段：
  - `to_status`：付款侧出金状态
  - `ti_status`：收款侧入金状态
  - `status`：整笔汇总状态
- 当前先按“自动失败不整笔自动重试”处理
- 特别是 `to_status = S`、`ti_status = F` 时，先记失败，后续通过补偿或人工处理

### 已发现的现有可复用底层能力

- `TransTransferTiBatchBusinessController / Service`
  - `retryPreOrder`
  - `processCredit`
  - `processDebit`
  - `updateDetailStatus`
- `TransAccountController`
  - 已有后管交易表补偿修复机制相关接口

### 当前建议

- 后期增加一个面向 Web 的手动补偿入口
- 该入口内部编排现有底层能力完成补偿处理

### 已确认的状态机与冻结流水号规则

- `trans_transfer_ti_batch_detail` 的三状态字段最终用于承载 `02` 模式不同阶段：
  - `to_status`：付款侧出金状态
  - `ti_status`：收款侧入金状态
  - `status`：整笔汇总状态
- 初始态：
  - `I / I / I`
- 开始付款：
  - `P / I / P`
- 付款成功、开始收款：
  - `S / P / P`
- 全成功：
  - `S / S / S`
- 付款失败：
  - `F / I / F`
- 付款成功、收款失败：
  - `S / F / F`

### 已确认的冻结落账规则

- `isFrozen=true` 时，task 成功后必须落：
  - `trans_frozen_t.F`
  - `trans_acct_frozen_change_detail_t.F`
  - 收款账户 `frozenAmt` 增加
- 采用拆分写法，不走统一黑盒入口
- `F.trans_no` 直接使用划付流水号
- 后续 `B/C` 的 `oldFrozenTransNo` 就是该划付流水号
