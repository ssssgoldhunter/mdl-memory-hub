# E-013 分页字段名不匹配（current vs pageNum）

- **需求**: E
- **级别**: 重要
- **状态**: 已确认非缺陷
- **发现日期**: 2026-04-04

## 问题描述

`DeductionBatchTaskJobService` 调用 `request.setPageNum(current)` 设置分页参数，但 `TransDeductionBatchBusinessServiceImpl.queryPendingDetailPage` 读取的是 `request.getCurrent()`。如果 DTO 中 `pageNum` 和 `current` 是不同字段，task 每次都查第一页，导致分页失效。

## 具体位置

- `DeductionBatchTaskJobService.java` :68 (`setPageNum`)
- `TransDeductionBatchBusinessServiceImpl.java` :93 (`getCurrent`)

## 建议修复

确认 DTO 字段名是否一致。如果不一致，统一为一个字段名。
