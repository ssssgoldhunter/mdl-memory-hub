# E-005 transSubType="U" 需确认与单笔B一致

- **需求**: E
- **级别**: 重要
- **状态**: 待确认修复
- **发现日期**: 2026-04-03

## 问题描述

`DeductionBatchPreCreatePack` 将 `transSubType` 设为 `TRANS_TYPE_SUB_U`（"U"）。E 批量扣款用的是 D02 模型，需确认该值是否与单笔 B 扣款链 `chainDeduction` 中使用的 `transSubType` 一致。

## 具体位置

- `DeductionBatchPreCreatePack.java` :47

## 当前代码

```java
request.setTransSubType(CommonConstants.TRANS_TYPE_SUB_U);
```

## 建议修复

对比单笔 B 的 `DeductionTransPack` 中 transSubType 赋值，确保 E 与 B 口径一致。
