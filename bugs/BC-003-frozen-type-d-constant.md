# BC-003 FROZEN_TYPE_D 硬编码未收敛到 CommonConstants

- **需求**: B+C
- **级别**: 重要
- **状态**: 已修复
- **发现日期**: 2026-04-03

## 问题描述

`FROZEN_TYPE_D = "D"` 在三处各自定义为 `private static final String`。`CommonConstants` 已有 `FROZEN_TYPE_F="F"` 和 `FROZEN_TYPE_UF="UF"`，但缺少 `FROZEN_TYPE_D`。如果值变化需要改多处。

## 具体位置

- `DeductionTransBAfter.java` :50
- `DeductionFrozenPoolSupport.java` :28
- `DeductionBatchExecuteServiceImpl.java` :78

## 当前代码

```java
private static final String FROZEN_TYPE_D = "D";
```

## 建议修复

在 `CommonConstants` 中添加 `FROZEN_TYPE_D = "D"`，三处统一引用。
