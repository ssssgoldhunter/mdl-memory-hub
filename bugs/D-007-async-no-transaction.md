# D-007 batchAddTransferTi02Data 异步执行无事务保障

- **需求**: D
- **级别**: 重要
- **状态**: 不修复
- **发现日期**: 2026-04-03

## 问题描述

`batchAddTransferTi02Data` 内部异步提交到 `taskExecutor`，调用者先拿到"成功"响应，但数据尚未持久化。如果异步线程全部失败，数据丢失且无补偿。

## 具体位置

- `TransTransferTiBatchBusinessServiceImpl.java` 约 :340（`batchAddTransferTiDataInternal` 中异步提交）

## 结论

业务确认后续数据处理按单条执行，不要求批次原子事务保障，因此本问题不作为缺陷修复。

是否同步/异步属于处理模型选择，不再按“必须具备事务包裹”的标准要求。
