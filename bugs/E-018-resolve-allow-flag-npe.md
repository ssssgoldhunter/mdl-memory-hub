# E-018 FrozenPoolHelper.resolveAllowFlag 中 getFrozenAmt() 无 null 检查

- **需求**: E+B+C
- **级别**: 严重
- **状态**: 已修复
- **发现日期**: 2026-04-05

## 问题描述

BC-005 修复了 `getRemainFrozenAmt()` 中的 `frozenAmt` null 守卫，但同文件 `resolveAllowFlag()` 中仍直接调用 `originalFrozen.getFrozenAmt().compareTo(afterUsedAmt)`，无 null 检查。

`loadAvailableOriginalFrozen` 只校验 `status` 和 `allowFlag`，不保证 `frozenAmt` 非 null。

## 具体位置

- `FrozenPoolHelper.java` :81

## 当前代码

```java
private boolean resolveAllowFlag(TransFrozenTQueryRes originalFrozen, BigInteger afterUsedAmt) {
    return originalFrozen.getFrozenAmt().compareTo(afterUsedAmt) == 0;
    // getFrozenAmt() 可能为 null
}
```

## 建议修复

```java
private boolean resolveAllowFlag(TransFrozenTQueryRes originalFrozen, BigInteger afterUsedAmt) {
    BigInteger frozenAmt = originalFrozen.getFrozenAmt();
    if (frozenAmt == null) {
        frozenAmt = BigInteger.ZERO;
    }
    return frozenAmt.compareTo(afterUsedAmt) == 0;
}
```
