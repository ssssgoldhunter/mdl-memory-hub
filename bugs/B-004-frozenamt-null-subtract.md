# B-004 frozenAmt 为 null 时 subtract 抛 NPE

- **需求**: B
- **级别**: 重要
- **状态**: 已修复
- **发现日期**: 2026-04-04

## 问题描述

`DeductionTransBAfter.createFrozenChangeDetail` 中直接调用 `paySubCardAccount.getFrozenAmt().subtract(transAmt)`，未检查 `getFrozenAmt()` 是否为 null。数据库映射的 BigInteger 字段可能为 null。

NPE 发生在 front 交易成功后、冻结变动明细写入时，导致状态不一致。

## 具体位置

- `DeductionTransBAfter.java` :307

## 当前代码

```java
request.setFrozenAmt(paySubCardAccount.getFrozenAmt().subtract(transAmt));
```

## 建议修复

```java
BigInteger frozenAmt = paySubCardAccount.getFrozenAmt();
if (frozenAmt == null) {
    throw new BaseException(..., "冻结金额不能为空");
}
request.setFrozenAmt(frozenAmt.subtract(transAmt));
```
