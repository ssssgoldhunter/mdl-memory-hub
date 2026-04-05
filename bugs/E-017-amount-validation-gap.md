# E-017 amount 为 null 或零/负数时校验不充分

- **需求**: E
- **级别**: 一般
- **状态**: 已修复
- **发现日期**: 2026-04-04

## 问题描述

两处校验存在间隙：

1. `validatePreCreateItem` 只校验 amount 可解析为 BigInteger，不校验正值
2. `DeductionBatchPreCreatePack` 校验 `compareTo(ZERO) <= 0`，但此时 amount 为 null 会 NPE

零或负数金额会通过第一道校验，落入明细表，在 LiteFlow 阶段才被拦截。

## 具体位置

- `TransDeductionBatchBusinessServiceImpl.java` :196-199
- `DeductionBatchPreCreatePack.java` :37

## 当前代码

```java
// validatePreCreateItem - 只校验可解析
new BigInteger(item.getAmount());

// PreCreatePack - amount 为 null 时 NPE
request.getAmount().matches(...)
```

## 建议修复

在 `validatePreCreateItem` 中统一校验 amount 非空、可解析、大于零。
