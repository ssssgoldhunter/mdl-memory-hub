# D-007 batchAddTransferTi02Data 异步执行无事务保障

- **需求**: D
- **级别**: 重要
- **状态**: 待确认修复
- **发现日期**: 2026-04-03

## 问题描述

`batchAddTransferTi02Data` 内部异步提交到 `taskExecutor`，调用者先拿到"成功"响应，但数据尚未持久化。如果异步线程全部失败，数据丢失且无补偿。

## 具体位置

- `TransTransferTiBatchBusinessServiceImpl.java` 约 :340（`batchAddTransferTiDataInternal` 中异步提交）

## 建议修复

评估是否需要将核心持久化改为同步，或加补偿机制保证数据最终一致。
