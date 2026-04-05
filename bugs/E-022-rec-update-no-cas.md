# E-022 rootRec/childRec 状态更新用 updateById 无状态守卫

- **需求**: E
- **级别**: 一般
- **状态**: 不修复
- **发现日期**: 2026-04-05

## 问题描述

`updateRootRecStatus` 和 `updateChildRecStatus` 调用 `transConsumeSubRecTService.update(req)`，底层是 MyBatis-Plus 的 `updateById`，只按 `recId` 匹配，无当前状态条件。并发场景下可被覆盖。

与 E-020 类似但影响面更小（rec 状态更新在双卡锁保护内）。

## 具体位置

- `DeductionBatchExecuteServiceImpl.java` :801-848

## 建议修复

低优先级。当前有 Redis 双卡锁保护，实际并发风险较低。可后续统一补 CAS。
