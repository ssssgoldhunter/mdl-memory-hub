# BC-004 第二个 subRec 最终状态未更新

- **需求**: B+C
- **级别**: 重要
- **状态**: 待确认修复
- **发现日期**: 2026-04-03

## 问题描述

`DeductionTransBAfter.updateResult()` 只更新 `slot.getTransSubId().get(0)`（rootRec）。`DeductionTrans.createSubConsume` 创建的第二个 subRec 也加到了 `slot.getTransSubId()` 中（index 1），但 BAfter 没有更新它到最终成功状态。

## 具体位置

- `DeductionTransBAfter.java` :101-148 `updateResult()`

## 当前代码

```java
TransConsumeSubRecTReq rootRec = new TransConsumeSubRecTReq();
rootRec.setRecId(slot.getTransSubId().get(0));  // 只更新了 index 0
// ...
updateCount = transConsumeSubRecTService.updateConsumeRecStatusByConsumeId(rootRec);
// slot.getTransSubId().get(1) 未被更新
```

## 建议修复

确认是否 by design。如果不是，需补充 index 1 的 subRec 状态更新。
