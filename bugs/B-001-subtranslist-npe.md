# B-001 subTransList 无空检查导致部分写入

- **需求**: B
- **级别**: 严重
- **状态**: 已修复
- **发现日期**: 2026-04-04

## 问题描述

`DeductionTrans.createSubConsume` 直接调用 `deductionTransVo.getSubTransList().get(0)`，未检查 subTransList 是否为 null 或空。如果上游未正确填充，将抛 NPE。

此时 `createMainConsume` 已成功持久化主交易记录，NPE 导致无法回滚，产生孤立的主交易记录。

## 具体位置

- `DeductionTrans.java` :225

## 当前代码

```java
SubTransVo subTransVo = deductionTransVo.getSubTransList().get(0);
```

## 建议修复

```java
List<SubTransVo> subTransList = deductionTransVo.getSubTransList();
if (subTransList == null || subTransList.isEmpty()) {
    throw new BaseException(..., "子交易列表不能为空");
}
SubTransVo subTransVo = subTransList.get(0);
```
