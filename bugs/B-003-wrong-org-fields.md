# B-003 子交易记录 org/operator 字段使用了收款方业务信息

- **需求**: B
- **级别**: 重要
- **状态**: 不修复
- **发现日期**: 2026-04-04

## 问题描述

`DeductionTrans.createSubConsume` 中，子交易的 `orgCode`、`operatorCode`、`firstOrgCode`、`secondOrgCode` 取自 `busInfo`，而 `busInfo` 是按 `receiveCardCode` 从 map 中获取的收款方业务信息。

但对比 `buildFrontConsumeReq`（:321）使用的是付款方（payCardCode）的业务信息来设 org/operator。扣款子交易的归属组织应为付款方，而非收款方。

## 具体位置

- `DeductionTrans.java` :231, 257-260

## 当前代码

```java
// line 231: 取的是收款方的业务信息
BasBusinessInfoQueryRes busInfo = slot.getBusinessInfos().get(subTransVo.getReceiveCardCode());
// line 257-260: 子交易记录用了收款方的 org
consumeSubDto.setOrgCode(busInfo.getSecondOrgCode());
consumeSubDto.setOperatorCode(busInfo.getOperatorCode());
```

## 处理说明

当前按业务口径保持现状，不继续调整该字段归属，不作为本轮缺陷修复项。
