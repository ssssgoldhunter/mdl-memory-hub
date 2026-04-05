# E-011 预落地失败后明细卡在 status=F，无重试机制

- **需求**: E
- **级别**: 重要
- **状态**: 已修复
- **发现日期**: 2026-04-04

## 问题描述

`batchPreCreate` 循环中先调 `saveDeductionDetail` 落明细，再调 LiteFlow 预落交易骨架。如果 LiteFlow 失败，`markPreCreateFailed` 将明细标为 `F`。

但调用方重试整个批量请求时，`saveDeductionDetail` 内的 transNo 去重检查会拒绝已存在的记录（包括状态为 `F` 的），导致该条明细永久卡住，无恢复路径。

## 具体位置

- `TransDeductionBatchBusinessServiceImpl.java` :64-81 (batchPreCreate 循环)
- `TransDeductionBatchBusinessServiceImpl.java` :122-124 (去重检查)

## 当前代码

```java
// 去重检查不区分 F 状态
if (transDeductionBatchDetailService.lambdaQuery()
        .eq(TransDeductionBatchDetail::getTransNo, item.getTransNo())
        .count() > 0) {
    throw new BaseException(...);  // 已存在即拒绝，包括 F 状态
}
```

## 建议修复

方案一：去重检查排除 `status=F` 的记录，允许重试覆盖。方案二：提供独立的明细状态重置接口。
