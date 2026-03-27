# 验证清单

更新时间：2026-03-27 14:25:00 CST

## 1. 仍待外部条件

- `A` 清结算查询 API
- 真实回调地址 / 通知联调环境

## 2. 仍待手测

### A

- `needRuleCheck=false`
- `needRuleCheck=true`
- `withDrawRuleCheck` 是否正确进入

### C

- 冻结
- 部分解冻
- 全额解冻
- 重复解冻
- 非法 `orgTransNo`

### B

- 普通扣款
- `useFrozen=true`
- 部分冻结扣款
- 冻结额度全部用尽后的行为

### D

- `TRANSFER_MODE_SWITCH=01`
- `TRANSFER_MODE_SWITCH=02`
- `02 + useFrozen=true`
- `02 + isFrozen=true`
- `02` task 跑完全链路

### front

- `B` 通知成功
- `D` 通知成功
- 通知失败后的 24 小时重发

## 3. 仍待补自动化测试

- `A` LiteFlow 节点开关
- `C` 部分解冻
- `B` 部分冻结扣款
- `D02` task 元数据透传与冻结联动
