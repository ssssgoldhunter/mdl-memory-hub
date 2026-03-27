# 当前代码状态

更新时间：2026-03-27 19:05:00 CST

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

### B

- Web 入口：
  - `/api/ums/catering/trans/scDeduction`
- 核心 deduction 链独立
- 复用原 `TransSlot`
- `useFrozen=true` 已走冻结额度管理能力
- 扣款通知已接独立 deduction topic
- `useFrozen=false`
  - 继续走普通扣款语义
- `useFrozen=true`
  - 直接消费原冻结额度
  - 不再额外走普通冻结 / 解冻动作

### D

- `TRANSFER_MODE_SWITCH` 已接入
- `02` 接口阶段走：
  - `pre`
  - `batchAddTransferTiData`
- `02` task 侧已新增专用启动入口
- `02` task 侧已新增：
  - `processDebit02`
  - `processCredit02`
- `01` 原有：
  - `processDebit`
  - `processCredit`
  保持不动
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

另：

- `fund-catering-data-batch` 相关编译已启动
- 当前正在补齐历史未下载依赖
- 截至本次 memory 更新时，尚未出现新的代码编译错误结论
