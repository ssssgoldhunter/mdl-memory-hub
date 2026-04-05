# E-009 task 分页扫描可能重复拾取 status=P 的明细

- **需求**: E
- **级别**: 严重
- **状态**: 已修复
- **发现日期**: 2026-04-04

## 问题描述

`DeductionBatchTaskJobService.scanAndProcessPendingDetails` 查询条件为 `status in (I, P)`。当一条明细从 `I` 变为 `P`（markProcessing）后，下轮分页查询仍会拾取该条记录，可能导致循环卡在第一页。

虽然 `processDeductionDetail02` 内部有 Redis 卡锁保护不会真正双重执行，但任务循环无法推进到后续页。

## 具体位置

- `DeductionBatchTaskJobService.java` :66-95
- `TransDeductionBatchBusinessServiceImpl.java` :101 (查询条件)

## 当前代码

```java
.in(TransDeductionBatchDetail::getStatus, CommonConstants.CONSUME_STATUS_I, CommonConstants.CONSUME_STATUS_P)
```

## 建议修复

方案一：查询条件只查 `status=I`，`P` 状态由单独补偿机制处理。方案二：task 单条处理完后从结果集中排除。
