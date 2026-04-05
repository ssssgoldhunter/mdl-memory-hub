# E-020 updateConsumeRecAccChangeStatusByRecId 无 expectedStatus CAS 保护

- **需求**: E
- **级别**: 重要
- **状态**: 不修复
- **发现日期**: 2026-04-05

## 问题描述

`beginAccountChange` 和 `markExecuteSuccess` 调用 `updateConsumeRecAccChangeStatusByRecId` 更新 rec 的 `acc_change_status`，底层 SQL 只做 `WHERE rec_id = ?`，无当前状态校验。

对比 `updateDetailStatus` 已有 `expectedStatus` 做 CAS 保护，但 rec 路径没有。如果并发重试同时通过明细的状态检查，两个线程会交叉覆盖 rec 状态。

## 具体位置

- `DeductionBatchExecuteServiceImpl.java` :354 (beginAccountChange)
- `DeductionBatchExecuteServiceImpl.java` :392 (markExecuteSuccess)

## 当前代码

```java
TransConsumeSubRecTReq childRecReq = new TransConsumeSubRecTReq();
childRecReq.setRecId(context.childRec.getRecId());
childRecReq.setAccChangeStatus(CommonConstants.CONSUME_SUB_REC_ACC_CHANGE_STATUS_01);
int updateCount = transConsumeSubRecTService.updateConsumeRecAccChangeStatusByRecId(childRecReq);
// 无 expectedStatus 条件
```

## 建议修复

在 `updateConsumeRecAccChangeStatusByRecId` 的 SQL 中增加 `AND acc_change_status = #{expectedStatus}` 条件。
