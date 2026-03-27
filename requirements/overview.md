# 需求总览

更新时间：2026-03-27 14:10:00 CST

> 当前实现边界以 `docs/superpowers/mdl-supply-chain-abcd/00-current-baseline.md` 为准。  
> 旧讨论中的模糊概念、过渡方案、已废弃方案，不再作为当前开发依据。

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
  - 对外入口复用消费体系，接口为 `/scDeduction`
  - 核心交易链与普通消费已分开
  - 复用原 `TransSlot`，不再单独分叉 slot
  - 仅支持 `04`
  - 支持 `useFrozen=true/false`
  - 扣款成功后的通知走独立 deduction topic

### 需求 C

- 名称：原冻结交易改造
- 核心点：
  - 继续作为原冻结额度池 + 独立冻结/解冻能力
  - 当前实现本质上是通用冻结额度管理能力
  - 解冻只使用 `orgTransNo`
  - 当前代码口径支持按剩余冻结额度做部分解冻
  - 当前代码口径支持按剩余冻结额度做部分冻结扣款
  - `B/D` 继续复用其内部额度池能力
  - `C` 不走异步通知模型

### 需求 D

- 名称：划付改造
- 核心点：
  - 保留 `01`，新增 `02 task` 模式
  - 参数表开关为 `TRANSFER_MODE_SWITCH`
  - `02` 接口阶段只走 `pre + batchAddTransferTiData`
  - `02` 只复用 `01` 中间那段“批量上传明细”
  - 其余真实划付仍由 task 执行
  - `isFrozen=true` 时，task 成功后写新的原冻结 `F`

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
5. `front`
   - `B/D` 通知与 `24h` 重发已完成
6. 自动化测试与联调
   - 未完成

## 当前开发依据

- 当前 superpower 模块：
  - `docs/superpowers/mdl-supply-chain-abcd/`
- 设计历史：
  - `2026-03-26-abcd-front-design-confirmed.md`
- 当前实现基线：
  - `2026-03-27-abcd-front-implementation-baseline.md`
