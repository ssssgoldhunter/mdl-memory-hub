# D-008 D02 task 中 processDetail 未捕获异常，一条失败中止整个扫描

- **需求**: D
- **级别**: 重要
- **状态**: 已修复
- **发现日期**: 2026-04-05

## 问题描述

`TransferTi02TaskJobService.processDetail` 调用 `transTransferTiBatchBusinessApi.processDetail02(transNo)` 但未捕获异常。如果单条执行抛出 `BaseException`，异常会冒泡到 `scanAndProcessPendingDetails` 的循环外，中止整个扫描，剩余未处理记录要等下一轮调度。

对比 E 批量的 `DeductionBatchTaskJobService` 每条都做了 try 处理。

## 具体位置

- `TransferTi02TaskJobService.java` :128-146

## 当前代码

```java
private void processDetail(TransTransferTiBatchDetailQueryRes detail) {
    // ... 各种校验 ...
    DefaultResult<Boolean> result = transTransferTiBatchBusinessApi.processDetail02(detail.getTransNo());
    // 无 try-catch，异常会中断整个扫描循环
}
```

## 建议修复

在 `scanAndProcessPendingDetails` 的循环中为每条记录加 try-catch，单条失败不影响后续。
