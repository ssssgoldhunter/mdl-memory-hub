# D-001 冻结主流水号唯一性未校验

- **需求**: D
- **级别**: 严重
- **状态**: 已修复
- **发现日期**: 2026-04-03
- **违反边界**: 需求D规则19 "D必须校验冻结主流水号唯一性"

## 问题描述

`createTransferFrozenIfNecessary()` 直接构造 `TransFrozenTReq` 并调用 `transFrozenTService.save()`，没有先查 `trans_frozen_t` 中是否已存在 `trans_no = detail.getTransNo()` 的记录。如果 `processDetail02` 被重试（上账成功但冻结写入后超时），会写入重复的 F 记录。

## 具体位置

- `TransTransferTiBatchBusinessServiceImpl.java` :3095-3176 `createTransferFrozenIfNecessary()`
- 关键行 :3145 `transFrozenTService.save(frozenReq)` — 无查重

## 当前代码

```java
// :3118-3145
TransFrozenTReq frozenReq = new TransFrozenTReq();
// ... 设置各字段
frozenReq.setTransNo(detail.getTransNo());  // :3127
// 没有先查 trans_frozen_t 是否已存在该 trans_no
DefaultResult<String> saveResult = transFrozenTService.save(frozenReq);
```

## 建议修复

写入前先查 `trans_frozen_t` by trans_no，已存在则跳过创建。
