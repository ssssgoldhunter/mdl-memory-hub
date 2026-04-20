# BC-006 FrozenTrans/UnFrozenTrans findFirst().get() 未处理空

- **需求**: B+C
- **级别**: 高
- **状态**: 已修复
- **发现日期**: 2026-04-05

## 问题描述

`FrozenTrans` 和 `UnFrozenTrans` 中使用 `findFirst().get()` 获取 Optional 结果，未处理 Optional.empty() 情况，可能抛出 `NoSuchElementException`。

## 具体位置

- `FrozenTrans.java` :51-52
- `UnFrozenTrans.java` :71-72, :75

## 当前代码

```java
// FrozenTrans.java:51-52
TransConsumeSubRecTQueryRes firstRec = slot.getTransConsumeSubRecs().stream()
    .filter(rec -> ...)
    .findFirst().get();  // 未处理 empty

// UnFrozenTrans.java:71-72
TransConsumeSubRecTQueryRes firstRec = slot.getTransConsumeSubRecs().stream()
    .filter(rec -> ...)
    .findFirst().get();  // 未处理 empty
```

## 建议修复

使用 `findFirst().orElseThrow()` 或在 filter 前检查 isEmpty：
```java
Optional<TransConsumeSubRecTQueryRes> firstRec = slot.getTransConsumeSubRecs().stream()
    .filter(rec -> ...)
    .findFirst();
if (firstRec.isEmpty()) {
    throw new BaseException(...);
}
```
