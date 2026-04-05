# D-010 D02 task markDetailFailed 未设置 status 导致循环失败重试

- **需求**: D
- **级别**: 严重
- **状态**: 已修复
- **发现日期**: 2026-04-05

## 问题描述

`TransferTi02TaskJobService.markDetailFailed` 只设置了 `toStatus` 和 `tiStatus` 为 `F`，未设置主状态 `status`。

`updateDetailStatus` API 校验要求 `status` 不能为空，否则返回 false。导致失败状态未更新成功。

下次 task 执行时，第 139 行检查 `status == S` 会再次尝试处理已失败的明细，导致循环失败重试。

## 具体位置

- `TransferTi02TaskJobService.java` :155-169 (markDetailFailed)
- `TransTransferTiBatchBusinessServiceImpl.java` :949-952 (status 校验)

## 修复

在 `markDetailFailed` 中添加 `req.setStatus(CommonConstants.CONSUME_STATUS_F)`。