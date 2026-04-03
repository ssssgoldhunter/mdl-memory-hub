# E-004 重试路径冻结/解冻可能重复

- **需求**: E
- **级别**: 重要
- **状态**: 待确认修复
- **发现日期**: 2026-04-03

## 问题描述

普通扣款模式下，`createFrozenBeforeTrade` 在每次执行时都创建冻结。如果 task 重试一条之前已创建了冻结但前置调用失败的明细，会再次创建冻结，后续解冻逻辑可能只解冻第二次的冻结。

## 具体位置

- `DeductionBatchExecuteServiceImpl.java` :167 `createFrozenBeforeTrade()`
- :291-316 `handleFrontFailed()` 中 `releaseFrozenBeforeAccountChange()`

## 建议修复

在 `createFrozenBeforeTrade` 前加判断，检查是否已存在本次交易的冻结记录。
