# E-019 E 批量扣款执行成功/失败后无通知

- **需求**: E
- **级别**: 严重
- **状态**: 已修复
- **发现日期**: 2026-04-05

## 问题描述

单笔 B 扣款 (`TransConsumeServiceImpl.transDeduction`) 在成功后调 `sendNotifyMessageAsync()` 发送扣款通知。D 划付在 task 执行成功后通过 `sendTransferNotifyMessage()` 发送通知。

但 E 批量扣款 (`DeductionBatchExecuteServiceImpl`) 的 `markExecuteSuccess` 和 `failAfterAccountOrContextError` 中均无任何通知调用。商户依赖异步通知获知扣款结果，E 执行完成后商户不会收到任何回调。

## 具体位置

- `DeductionBatchExecuteServiceImpl.java` markExecuteSuccess 方法 — 缺少成功通知
- `DeductionBatchExecuteServiceImpl.java` failAfterAccountOrContextError 方法 — 缺少失败通知

## 建议修复

在 `markExecuteSuccess` 中补发扣款成功通知（复用 B 的 deduction topic），在 `failAfterAccountOrContextError` 中补发失败通知。
