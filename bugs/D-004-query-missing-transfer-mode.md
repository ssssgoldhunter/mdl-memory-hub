# D-004 02待处理查询缺 transferMode 过滤

- **需求**: D
- **级别**: 重要
- **状态**: 不修复
- **发现日期**: 2026-04-03

## 问题描述

`queryMode02PendingDetailPage` 只按 `batchNo=null` + `status=S` 过滤，没有 `transferMode` 条件。如果 01 和 02 同时运行，02 task 可能拾取 01 的记录。

## 具体位置

- `TransTransferTiBatchBusinessServiceImpl.java` :523-524

## 当前代码

```java
request.setBatchNo(null);
request.setStatus(CommonConstants.CONSUME_STATUS_S);
return transTransferTiBatchDetailService.selectPage(request);
```

## 结论

业务确认该查询不以 `transferMode` 作为必要边界条件，本问题不作为缺陷修复。

当前查询已通过 `status/toStatus/tiStatus` 约束待处理记录，满足现有业务使用口径。
