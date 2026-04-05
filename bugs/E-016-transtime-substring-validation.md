# E-016 transTime substring 无长度校验

- **需求**: E
- **级别**: 一般
- **状态**: 已修复
- **发现日期**: 2026-04-04

## 问题描述

`buildFrontConsumeReq` 中对 `transTime` 做 `substring(0, 8)` 和 `substring(8)`，假设字符串至少 9 位。如果 `transTime` 格式不符或过短，将抛 `StringIndexOutOfBoundsException`。

`validatePreCreateItem` 只校验非空，不校验格式和长度。

## 具体位置

- `DeductionBatchExecuteServiceImpl.java` :299-300

## 当前代码

```java
request.setTransDate(context.detail.getTransTime().substring(0, 8));
request.setTransTime(context.detail.getTransTime().substring(8));
```

## 建议修复

在 `validatePreCreateItem` 或 `DeductionBatchPreCreatePack` 中增加 transTime 格式校验（长度 >= 14，yyyyMMddHHmmss）。
