# E-003 明细状态更新无CAS

- **需求**: E
- **级别**: 重要
- **状态**: 待确认修复
- **发现日期**: 2026-04-03

## 问题描述

`updateDetailStatus` 直接按 `detailId` 更新，没有在 WHERE 中加 `status = expected` 条件。并发 task 轮次可同时通过 status 检查并各自推进状态。

## 具体位置

- `DeductionBatchExecuteServiceImpl.java` :694-713

## 当前代码

```java
private void updateDetailStatus(Long detailId, String frontStatus, String dcStatus, String drStatus, String status) {
    TransDeductionBatchDetailReq updateReq = new TransDeductionBatchDetailReq();
    updateReq.setDetailId(detailId);
    // ... set 各字段
    DefaultResult<Boolean> result = transDeductionBatchDetailService.update(updateReq);
    // update 无 WHERE status=expected 条件
}
```

## 建议修复

在 SQL 更新级别加乐观锁 WHERE 条件，如 `WHERE detail_id = ? AND status = ?`，更新失败则抛并发冲突异常。
