# D-009 D02 task 中明细在扫描和执行中各查一次

- **需求**: D
- **级别**: 一般
- **状态**: 不修复
- **发现日期**: 2026-04-05

## 问题描述

`TransferTi02TaskJobService.processDetail` 从分页查询拿到完整 `TransTransferTiBatchDetailQueryRes` 对象，但只取 `transNo` 传给 `processDetail02`。`processDetail02` 内部再按 `transNo` 重新查询一次明细。

每条记录被查两次，双倍 DB 负载。

## 具体位置

- `TransferTi02TaskJobService.java` :143 (只传 transNo)
- `TransTransferTiBatchBusinessServiceImpl.java` :2940 (内部再查)

## 建议修复

低优先级。可在后续优化时考虑将已查到的 detail 对象直接传入 `processDetail02`，避免重复查询。
