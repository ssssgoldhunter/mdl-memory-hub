# BC-005 FrozenPoolHelper getFrozenAmt 为 null 时 NPE

- **需求**: B+C
- **级别**: 重要
- **状态**: 已修复
- **发现日期**: 2026-04-04

## 问题描述

`FrozenPoolHelper.getRemainFrozenAmt` 调用 `originalFrozen.getFrozenAmt().subtract(getUsedAmt(originalFrozen))`，未对 `getFrozenAmt()` 做 null 检查。

同文件 `getUsedAmt` 对 `freezableAmt` 做了 null 处理，但 `frozenAmt` 没有。从 `validateOldFrozenDetail` 和 `consumeOldFrozenAmount` 调用，NPE 将导致冻结扣款流程中断。

## 具体位置

- `FrozenPoolHelper.java` :69

## 当前代码

```java
public BigInteger getRemainFrozenAmt(TransFrozenTQueryRes originalFrozen) {
    return originalFrozen.getFrozenAmt().subtract(getUsedAmt(originalFrozen));
}
```

## 建议修复

```java
public BigInteger getRemainFrozenAmt(TransFrozenTQueryRes originalFrozen) {
    BigInteger frozenAmt = originalFrozen.getFrozenAmt();
    if (frozenAmt == null) {
        frozenAmt = BigInteger.ZERO;
    }
    return frozenAmt.subtract(getUsedAmt(originalFrozen));
}
```
