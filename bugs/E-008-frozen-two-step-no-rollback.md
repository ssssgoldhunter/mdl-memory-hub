# E-008 冻结两步写非原子，第二步失败无补偿

- **需求**: E
- **级别**: 严重
- **状态**: 不修复
- **发现日期**: 2026-04-04

## 问题描述

`consumeOldFrozenAmount` 先调 `transFrozenTService.save` 新增冻结扣款记录，再调 `transFrozenTService.updateStatusByTransNoAndCardCode` 更新原始冻结记录。两步非原子操作。

如果第一步成功（新冻结记录已落）但第二步失败（原始冻结未扣减），异常触发失败处理但无补偿逻辑删除已保存的新冻结记录。冻结池额度跟踪不一致。

## 具体位置

- `DeductionBatchExecuteServiceImpl.java` :475-518 (consumeOldFrozenAmount)

## 当前代码

```java
DefaultResult<String> saveResult = transFrozenTService.save(deductionFrozenReq);  // 第一步
// ...
DefaultResult<Boolean> updateResult = transFrozenTService.updateStatusByTransNoAndCardCode(updateReq);  // 第二步
// 第二步失败后无补偿删除第一步的记录
```

## 处理说明

当前按业务口径保持现状，不做跨库回滚补偿。

- 冻结相关操作涉及两个库
- 异常时交易保持进行中 `P`
- 冻结金额不释放
- 后续由补偿/重试继续推进
