# BC-001 冻结池无并发保护，双重消耗风险

- **需求**: B+C
- **级别**: 严重
- **状态**: 已修复
- **发现日期**: 2026-04-03
- **违反边界**: 需求B规则18 "同一笔扣款不能重复消耗同一笔原冻结额度"

## 问题描述

`DeductionFrozenPoolSupport.consumeOldFrozenAmount()` 中 `frozenPoolHelper.loadAvailableOriginalFrozen()` 做普通查询（非 `SELECT FOR UPDATE`），两笔并发扣款可同时通过 `validateRemainingAmount` 校验，然后各自写 D 记录、更新原始 F，导致原冻结额度被超卖。

## 具体位置

- `DeductionFrozenPoolSupport.java` :63-116 `consumeOldFrozenAmount()`
- `FrozenPoolHelper.java` :26-50 `loadAvailableOriginalFrozen()` — 查询无行锁

## 当前代码

```java
// FrozenPoolHelper.java :33
DefaultResult<TransFrozenTQueryRes> queryResult = transFrozenTService.selectOne(queryReq);
// 普通查询，无 SELECT FOR UPDATE

// DeductionFrozenPoolSupport.java :103
DefaultResult<String> saveResult = transFrozenTService.save(deductionFrozenReq);
// :112
DefaultResult<Boolean> updateResult = transFrozenTService.updateStatusByTransNoAndCardCode(updateReq);
// validate 和 consume 之间存在并发窗口
```

## 建议修复

在 validate-then-consume 路径加 Redis 锁（scope: orgFrozenTransNo）或对原冻结记录 `SELECT FOR UPDATE`。
