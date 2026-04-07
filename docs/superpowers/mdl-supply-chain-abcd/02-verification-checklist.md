# 验证清单

更新时间：2026-03-31 10:18:00 CST

## 1. 仍待外部条件

- `A` 清结算查询 API
- 真实回调地址 / 通知联调环境
- `bas_param_t` 配套项补齐：
  - `PARAM_PAY_TRANS_TYPE` 追加 `DC`
  - `PARAM_REC_TRANS_TYPE` 追加 `DR`

## 2. 仍待手测

### A

- `needRuleCheck=false`
- `needRuleCheck=true`
- `withDrawRuleCheck` 是否正确进入
- 清结算查询未接入前，只验证链路、开关和日志，不验证真实拦截

### C

- 冻结
- 部分解冻
- 全额解冻
- 重复解冻
- 非法 `orgTransNo`
- 账户侧 `04.frozenAmt` 不足时是否正确拦截
- 原冻结剩余额度足够但账户冻结金额不足时是否正确拦截

### B

#### `useFrozen=false`

- 前置 `04` 可用余额校验
- 正式交易前冻结金额占用
- 账户变动前解冻
- 收付款卡同步账户更新
- 主交易类型是否为 `D`
- 账户变动明细类型是否为：
  - 付款方 `DC`
  - 收款方 `DR`

#### `useFrozen=true`

- 原冻结剩余额度校验
- 账户当前冻结金额校验
- 账户总冻结金额与当前原冻结剩余额度的一致性保护
- 交易成功后是否生成本次 `D`
- 是否正确回写原始 `F`
- 是否同步更新收付款卡
- 是否正确减少付款卡 `balance/frozenAmt`
- 是否不再额外走普通冻结 / 解冻动作

### D

- `TRANSFER_MODE_SWITCH=01`
- `TRANSFER_MODE_SWITCH=02`
- `02` 是否只在划付域生效，不影响原转账链
- `02` task 是否直接扫描明细表
- `02` 是否逐条调用 `processDetail02`
- `02 + isFrozen=false`
- `02 + isFrozen=true`
- `02 + useFrozen=true`
- `02` 收款卡上账成功后是否立即写冻结
- `02` 通知是否在 task 成功点发送

### front

- `B` 通知成功
- `D` 通知成功
- 通知失败后的 1 小时重发

## 3. 已废弃的旧验证口径

- 不再验证：
  - `D02` 通过 `data-batch` 启动
  - `D02` 通过 `processDebit02 + processCredit02` 两接口串行执行
  - `B` 使用主交易类型 `P`
  - `B` 使用账户变动明细类型 `PC/PR`
  - `B` 收款侧异步上账
  - `D` 通过原转账链触发

## 4. 仍待补自动化测试

- `A` LiteFlow 节点开关
- `C` 部分解冻 + 账户冻结金额校验
- `B useFrozen=false` 冻结 / 解冻链路
- `B useFrozen=true` 原冻结池消费与 `D` 生成
- `B` 主交易类型 `D` 与账户明细类型 `DC/DR`
- `D02` task 明细扫描、`processDetail02`、`isFrozen` 联动
