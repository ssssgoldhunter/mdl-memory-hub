# E-021 resetFailedDeductionDetail 设置的字段被 update 方法静默丢弃

- **需求**: E
- **级别**: 重要
- **状态**: 不修复
- **发现日期**: 2026-04-05

## 问题描述

`resetFailedDeductionDetail` 为重试场景填充了 `batchNo`、`transAmt`、`payCardCode`、`receiveCardCode`、`useFrozen` 等字段，但 `TransDeductionBatchDetailServiceImpl.update()` 只处理 `frontStatus`、`dcStatus`、`drStatus`、`status`、`remark`。

如果调用方重试时传入了不同的金额或卡号，这些变更会被静默丢弃，明细重置为 `I` 但仍按旧的金额和卡号执行。

## 具体位置

- `TransDeductionBatchBusinessServiceImpl.java` :199-230 (resetFailedDeductionDetail)
- `TransDeductionBatchDetailServiceImpl.java` :45-77 (update 只处理部分字段)

## 当前代码

```java
// resetFailedDeductionDetail 设置了很多字段
updateReq.setTransAmt(new BigInteger(item.getAmount()).longValue());
updateReq.setPayCardCode(item.getPayCardCode());
// ...

// 但 update() 只处理 status 相关字段
wrapper.set(TransDeductionBatchDetail::getFrontStatus, req.getFrontStatus());
wrapper.set(TransDeductionBatchDetail::getStatus, req.getStatus());
// transAmt, payCardCode 等被忽略
```

## 建议修复

方案一：在 update 方法中补齐对金额、卡号等字段的更新。方案二：如果重试不允许修改这些字段，在 resetFailedDeductionDetail 入口校验并拒绝不一致的参数。
