# E-010 失败处理中状态更新可能部分成功

- **需求**: E
- **级别**: 重要
- **状态**: 已修复
- **发现日期**: 2026-04-04

## 问题描述

`failAfterAccountOrContextError` 依次更新 consumeMainStatus、childRec accChangeStatus、detailStatus。如果 `updateConsumeMainStatus` 抛异常，detailStatus 不会被更新，导致消费主表和明细表状态分裂。

## 具体位置

- `DeductionBatchExecuteServiceImpl.java` :422-446

## 当前代码

```java
// 这些调用可能部分成功部分失败
updateConsumeMainStatus(..., CONSUME_STATUS_F);   // 成功
transConsumeSubRecTService.updateConsumeRecAccChangeStatusByRecId(...);  // 成功
updateDetailStatus(...);  // 如果这行抛异常，前面已提交无法回滚
```

## 建议修复

将失败状态更新包裹在 try-catch 中，确保每步失败不影响后续步骤执行。所有状态更新都应尽力完成。
