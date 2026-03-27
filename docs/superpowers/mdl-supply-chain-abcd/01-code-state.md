# 当前代码状态

更新时间：2026-03-27 14:25:00 CST

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

### B

- Web 入口：
  - `/api/ums/catering/trans/scDeduction`
- 核心 deduction 链独立
- 复用原 `TransSlot`
- `useFrozen=true` 已走冻结额度管理能力
- 扣款通知已接独立 deduction topic

### D

- `TRANSFER_MODE_SWITCH` 已接入
- `01/02` 已按参数表开关分流
- `02` 接口阶段走：
  - `pre`
  - `batchAddTransferTiData`
- `02` task 侧负责真正划付完成
- `useFrozen/isFrozen` 元数据已透传到 task
- 成功后通知已在 task 成功点发送

### front

- `B` 扣款 topic 已具备
- `D` 划付 topic 已具备
- HTTP consumer 已支持 24 小时窗口内重发

## 2. 已清理的错误实现

- `A` 通过 `front` 查本地失败扣款再拦截提现
- `DeductionTransSlot`
- `D02 registerMode02Detail / batchRegisterMode02Detail`

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
mvn -Dmaven.repo.local=/tmp/codex-m2 -pl fund-catering-consume/fund-catering-consume-service,fund-catering-web -am -DskipTests compile
```

结果：

- `BUILD SUCCESS`
