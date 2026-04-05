# E-006 frozenResult 返回 null 时 NPE

- **需求**: E
- **级别**: 严重
- **状态**: 已修复
- **发现日期**: 2026-04-04

## 问题描述

`createFrozenBeforeTrade`、`releaseFrozenBeforeAccountChange`、`createFrozenChangeDetail` 三处调用 `transAcctFrozenChangeDetailTService.createFrozenDetail(request)` 后直接调用 `frozenResult.isSuccess()`，未做 null 检查。如果远程调用超时或序列化失败返回 null，将抛出 NPE。

NPE 被 catch 捕获后调用 `failAfterAccountOrContextError`，但此时冻结操作是否已生效不确定，产生资金歧义状态。

## 具体位置

- `DeductionBatchExecuteServiceImpl.java` :251 (createFrozenBeforeTrade)
- `DeductionBatchExecuteServiceImpl.java` :467 (releaseFrozenBeforeAccountChange)
- `DeductionBatchExecuteServiceImpl.java` :543 (createFrozenChangeDetail)

## 当前代码

```java
DefaultResult<Long> frozenResult = transAcctFrozenChangeDetailTService.createFrozenDetail(request);
if (!frozenResult.isSuccess() || ...) {  // frozenResult 为 null 时 NPE
```

## 建议修复

```java
DefaultResult<Long> frozenResult = transAcctFrozenChangeDetailTService.createFrozenDetail(request);
if (frozenResult == null || !frozenResult.isSuccess() || ...) {
```
