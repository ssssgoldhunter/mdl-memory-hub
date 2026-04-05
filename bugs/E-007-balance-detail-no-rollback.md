# E-007 余额已改但明细未落，无回滚

- **需求**: E
- **级别**: 严重
- **状态**: 不修复
- **发现日期**: 2026-04-04

## 问题描述

`syncUpdateSubAccounts` 内部先调 `baseAccountServiceApi.batchChangeAccount` 更新子账户余额，再调 `transAccountApi.batchChangeAccountDetail` 落账户变动明细。两步跨服务 RPC 调用无事务包裹。

如果第一步成功（余额已改）但第二步失败（明细未落），异常触发 `failAfterAccountOrContextError` 标记明细失败，但余额已变更且无对应流水记录。

## 具体位置

- `DeductionBatchExecuteServiceImpl.java` :550-619 (syncUpdateSubAccounts)

## 当前代码

```java
// 第一步：改余额
DefaultResult<Integer> batchAccountResult = baseAccountServiceApi.batchChangeAccount(batchReq);
// 第二步：落明细
DefaultResult<Integer> detailResult = transAccountApi.batchChangeAccountDetail(detailBatchReq);
// 两步之间无事务保障
```

## 建议修复

考虑将余额变更和明细写入合并为同一个对端接口调用，或在第二步失败时增加补偿逻辑（冲正第一步）。
