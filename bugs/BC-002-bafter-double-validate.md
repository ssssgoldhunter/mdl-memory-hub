# BC-002 BAfter 二次校验冻结池，前置成功后可能抛异常导致不一致

- **需求**: B+C
- **级别**: 严重
- **状态**: 待确认修复
- **发现日期**: 2026-04-03
- **违反边界**: 需求B规则18

## 问题描述

`DeductionTransBAfter.process()` 在 `updateResult()` 将交易状态改成 S 后，再调用 `validateOldFrozenDetail` 校验冻结池。如果此时原冻结已被并发消耗完，校验抛异常，但交易已被标记为成功——状态不一致。

## 具体位置

- `DeductionTransBAfter.java` :82-92

## 当前代码

```java
// :82 先更新交易结果为 S
updateResult(slot, frontResult);
if (!ApiConstants.RESULT_SUCCESS.equals(slot.getCode())) {
    return;
}

// :87-88 刷新账户
slot.refreshCardSubAccount(deductionTransVo.getPayCardCode(), baseAccountServiceApi);
slot.refreshCardSubAccount(deductionTransVo.getReceiveCardCode(), baseAccountServiceApi);

// :91 二次校验冻结池 — 如果这里抛异常，交易已经是 S 了
oldFrozen = deductionFrozenPoolSupport.validateOldFrozenDetail(deductionTransVo, paySub);
deductionFrozenPoolSupport.consumeOldFrozenAmount(slot, paySub, oldFrozen);
```

## 建议修复

冻结池校验+消耗应原子化，或在 `updateResult` 之前完成冻结池操作。
