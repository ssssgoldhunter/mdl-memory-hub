# E-012 transNo 存在性检查与插入之间无 DB 唯一约束

- **需求**: E
- **级别**: 重要
- **状态**: 不修复
- **发现日期**: 2026-04-04

## 问题描述

`saveDeductionDetail` 先查询 transNo 是否已存在，再插入。两步之间无数据库唯一约束保护。并发请求同一 transNo 时，两个线程都可能通过存在性检查，导致重复插入。

`validateTransNoDuplicateInBatch` 只防止单次批量请求内重复，不防并发请求。

## 具体位置

- `TransDeductionBatchBusinessServiceImpl.java` :121-154

## 当前代码

```java
if (transDeductionBatchDetailService.lambdaQuery()
        .eq(TransDeductionBatchDetail::getTransNo, item.getTransNo())
        .count() > 0) {
    throw new BaseException(...);
}
// ... 竞态窗口 ...
DefaultResult<Long> detailResult = transDeductionBatchDetailService.save(detailReq);
```

## 建议修复

在 `trans_deduction_batch_detail` 表上对 `trans_no` 添加唯一索引，用 DB 约束兜底。
