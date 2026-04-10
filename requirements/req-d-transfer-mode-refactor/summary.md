# 需求 D 摘要

更新时间：2026-03-31 10:10:00 CST

## 需求名称

划付改造

## 业务目标

在保留原 `01` 划付模式的基础上，新增 `02` 定时任务划付模式：

- `01`：沿用老的批量任务模式
- `02`：接口先落划付数据，task 再定时执行内部上下账
- `D` 全部属于“划付域”，不属于原转账链

## 已确认结论

### 1. 模式开关

- 新增划付模式开关：
  - `01`：老模式
  - `02`：定时任务模式
- 开启 `01` 时，沿用原划付逻辑
- 开启 `02` 时，原 `01` 对应的划付任务不执行，由 `fund-catering-task` 中新的定时划付任务执行
- 开关只在划付域生效，不侵入原转账链

### 2. `02` 模式接口阶段

- `02` 模式沿用 `01` 的划付明细新增接口和主要数据结构
- `02` 模式下 `batchNo` 不强校验
- 接口阶段就落：
  - 转账表数据
  - 划付流水表数据
- 接口阶段不自动创建内部批次
- 接口阶段不做：
  - 账户变动
  - 实际内部上下账
  - 冻结金额处理
  - 冻结主记录写入

### 3. `02` 模式 task 阶段

- task 定时分页扫描待处理明细表数据
- task 带任务级 Redis 锁，避免并发重复执行
- 单笔处理时，外部只调用一个 `processDetail02`
- `processDetail02` 内部一次性完成：
  - 执行内部上下账
  - 写账户变动
  - 更新转账表状态、划付状态
  - `isFrozen=true` 时冻结写入
  - 通知发送

`trans_transfer_ti_batch_detail` 三状态字段建议流转：

- 接口新增成功：
  - `to_status = I`
  - `ti_status = I`
  - `status = I`
- task 开始执行付款侧：
  - `to_status = P`
  - `ti_status = I`
  - `status = P`
- 付款成功，开始处理收款侧：
  - `to_status = S`
  - `ti_status = P`
  - `status = P`
- 收款成功：
  - `to_status = S`
  - `ti_status = S`
  - `status = S`
- 付款失败：
  - `to_status = F`
  - `ti_status = I`
  - `status = F`
- 付款成功但收款失败：
  - `to_status = S`
  - `ti_status = F`
  - `status = F`

### 4. isFrozen=false

- 按普通划付完成内部上下账
- 不生成冻结记录
- 不进入 `需求 C/B` 的冻结额度池体系

### 5. isFrozen=true

- 收款卡上账成功后立即生成标准原冻结主记录 `F`
- 这笔 `F` 直接进入 `需求 C` 的原冻结额度池模型
- 同时执行：
  - 写 `trans_acct_frozen_change_detail_t`
  - 增加账户冻结金额
- 账户冻结金额更新继续复用 `createFrozenDetail(...)` 的已有账户更新逻辑
- `F.trans_no` 直接使用划付流水号
- 后续 `需求 B` 的 `orgFrozenTransNo`、`需求 C` 的 `orgTransNo` 都指向该划付流水号

### 6. 与 C/B 的复用关系

- `需求 D` 在 `02 + isFrozen=true` 时，是原冻结来源生产方
- `需求 B` 后续可以基于这笔 `F` 做冻结扣款
- `需求 C` 后续可以基于这笔 `F` 做剩余额度释放

### 7. 多笔划付冻结规则

- 不做账户级合并冻结池
- 一笔划付成功对应一笔独立原冻结 `F`
- 后续冻结扣款和解冻都按原冻结流水号一一对应处理

### 8. 唯一性规则

- `需求 D` 在 `isFrozen=true` 场景下，需要优先校验本次生成的冻结主流水号 `trans_no` 唯一
- 一笔划付只能生成一笔对应的原冻结主记录

### 9. 失败处理与后续补偿方向

- 当前阶段先按“失败后不自动重试整笔”设计
- 尤其在 `to_status = S`、`ti_status = F` 场景下，先记失败，后续走补偿或人工处理
- 后期需要提供 Web 手动补偿入口
- 手动补偿入口建议基于现有底层能力编排实现，而不是直接暴露底层修复接口
- 当前可复用的底层能力包括：
  - `TransTransferTiBatchBusinessService` 中的 `retryPreOrder`、`processDetail02`、`updateDetailStatus`
  - `TransAccountController` 中已有的后管交易表补偿修复机制接口

### 10. 清结算通知与 front 边界

- `需求 D` 不在接口受理成功后立即通知清结算
- 触发时机为：
  - task 实际划付成功后
  - 或补偿处理完成后
- `需求 D` 仍通过 `RocketMQ` 投递到 `front`
- `front` 消费者后续不再走 `URL`
- 当前代码在 `front` 消费者中登记“待清结算 API” `TODO`
- 待清结算接口和报文定义明确后，在 `front` 消费者内直接调用清结算接口
