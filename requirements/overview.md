# 需求总览

更新时间：2026-04-20 14:35:00 CST

> 当前实现边界以 `docs/superpowers/mdl-supply-chain-abcd/00-current-baseline.md` 为准。  
> 旧讨论中的模糊概念、过渡方案、已废弃方案，不再作为当前开发依据。

## 大计划归属

- 大计划名称：
  - `mdl定制`
- 当前 `A/B/C/D/front` 统一归属 `mdl定制` 下的供应链改造计划
- 后续扩展中的 `E` 批量扣款，也统一归属该计划

## 当前需求

### 需求 A

- 名称：提现接口及逻辑改造
- 核心点：
  - `needRuleCheck` 已作为对外提现入参
  - 提现 `LiteFlow` 已新增 `withDrawRuleCheck` 节点
  - 当前节点中清结算查询仍为 `TODO`
  - `front` 只承担提现结果通知，不承担提现前查询校验

### 需求 B

- 名称：新增扣款接口
- 核心点：
  - 对外入口复用消费体系，代码接口路径为 `/consume/trans/transDeduction`
  - 核心交易链与普通消费已分开
  - 复用原 `TransSlot`，不再单独分叉 slot
  - 主交易类型已改为 `D`
  - 账户变动明细类型已改为 `DC/DR`
  - 仅支持 `04`
  - 支持 `useFrozen=true/false`
  - `useFrozen=false`：前置冻结、账户变动前解冻、同步更新收付款卡
  - `useFrozen=true`：直接消费原冻结额度、同步更新收付款卡
  - 扣款成功后仍通过 `RocketMQ` 投递到 `front`
  - `front` 消费者当前已不走 `URL` 回调
  - `front` 扣款消费者当前已直接调用清结算接口 `resultNotifyApi`

### 需求 C

- 名称：原冻结交易改造
- 核心点：
  - 继续作为原冻结额度池 + 独立冻结/解冻能力
  - 解冻只使用 `orgTransNo`
  - 冻结扣款使用 `orgFrozenTransNo`
  - 当前代码口径支持按剩余冻结额度做部分解冻
  - 当前代码口径支持按剩余冻结额度做部分冻结扣款
  - `B` 继续复用其内部额度池能力
  - `C` 不走异步通知模型，不走清结算通知

### 需求 D

- 名称：划付改造
- 核心点：
  - 保留 `01`，新增 `02 task` 模式
  - 参数表开关为 `TRANSFER_MODE_SWITCH`
  - `02` 不依赖批次任务启动
  - `02` 接口阶段先预落交易骨架 / 预上账数据，再落划付明细
  - `02` 参考 `01` 的划付明细落地方式，但执行模型独立
  - `02` 真正执行由 `fund-catering-task` 扫明细后逐条调 `processDetail02`
  - `isFrozen=true` 时，收款卡上账成功后写新的原冻结 `F`
  - `D` 全部属于划付域，不属于原转账链
  - 划付成功后仍通过 `RocketMQ` 投递到 `front`
  - `front` 消费者当前已不走 `URL` 回调
  - `front` 划付消费者当前已直接调用清结算接口 `resultNotifyApi`

### 需求 E

- 名称：批量扣款
- 当前状态：
  - 已完成当前实现口径
  - 测试已通过
- 核心点：
  - 业务语义复用 `B`，代码链路与单笔 `B` 完全隔离
  - 执行模型参考 `D02`
  - 接口阶段先落明细，再预落交易骨架（`chainDeductionPre`）
  - `transNo` 必传，由调用方提供
  - task 扫明细表逐条执行（`DeductionBatchTaskJobService`）
  - 单条执行入口 `processDeductionDetail02` 内部根据 `useFrozen` 分普通扣款与冻结扣款
  - 冻结扣款依赖 `orgFrozenTransNo`，复用 `FrozenPoolHelper`
  - 接口阶段不锁卡，task 单条执行前锁付款卡和收款卡

## 当前状态

1. `A`
   - 已完成主链接入
   - 待清结算查询 API
2. `C`
   - 已完成当前实现口径
3. `B`
   - 已完成当前实现口径
4. `D`
   - 已完成当前实现口径
5. `E`
   - 已完成当前实现口径
   - 测试已通过
6. `front`
   - `B/D/银行实收` 仍保留 `RocketMQ` 通知结构
   - `B/D/银行实收` 的 `front` 消费者当前已直接调用清结算接口
   - 普通消费/充值等通知链仍兼容 `notifyUrl` 回调
   - 只有提现结果通知继续沿用现有回调链路
7. 自动化测试与联调
   - 未完成

## 当前开发依据

- 当前 superpower 模块：
  - `docs/superpowers/mdl-supply-chain-abcd/`
- 设计历史：
  - `2026-03-26-abcd-front-design-confirmed.md`
- `E` 设计稿：
  - `docs/superpowers/specs/2026-03-31-e-batch-deduction-design.md`
- `E` 实现计划：
  - `docs/plans/2026-03-31-e-batch-deduction-implementation-plan.md`
- 当前实现基线：
  - `docs/superpowers/mdl-supply-chain-abcd/00-current-baseline.md`
  - `docs/superpowers/mdl-supply-chain-abcd/01-code-state.md`
