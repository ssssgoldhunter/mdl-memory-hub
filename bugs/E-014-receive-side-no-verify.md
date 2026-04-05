# E-014 handleReceiveSide 只刷新不验证

- **需求**: E
- **级别**: 重要
- **状态**: 已修复
- **发现日期**: 2026-04-04

## 问题描述

`handleReceiveSide` 只调 `refreshReceiveSubAccount(context)` 刷新收款方子账户，然后返回。刷新后的数据未被使用、未做验证。`syncUpdateSubAccounts` 已在之前同步更新了收付款双方，此处刷新结果丢弃。

可能是预留的验证位未实现，或属于无效 RPC 调用。

## 具体位置

- `DeductionBatchExecuteServiceImpl.java` :393-395

## 当前代码

```java
private void handleReceiveSide(ExecutionContext context) {
    refreshReceiveSubAccount(context);
}
```

## 建议修复

如果需要验证收款到账，补充验证逻辑。如果不需要，移除该方法和调用。
