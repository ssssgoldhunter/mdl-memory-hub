# E-002 markProcessing 缺 frontStatus=P

- **需求**: E
- **级别**: 严重
- **状态**: 待确认修复
- **发现日期**: 2026-04-03
- **违反边界**: 需求E规则16 "开始扣款：frontStatus=P, dcStatus=I, drStatus=I, status=P"

## 问题描述

`markProcessing` 只设 `status=P`，`frontStatus` 传 null 不更新。按状态机，进入处理时应同时推进 `frontStatus=P, status=P`。

## 具体位置

- `DeductionBatchExecuteServiceImpl.java` :690-692

## 当前代码

```java
private void markProcessing(ExecutionContext context) {
    updateDetailStatus(context.detail.getDetailId(), null, null, null, CommonConstants.CONSUME_STATUS_P);
    //                                ^ frontStatus=null，不更新
}
```

## 建议修复

```java
private void markProcessing(ExecutionContext context) {
    updateDetailStatus(context.detail.getDetailId(), 
        CommonConstants.CONSUME_STATUS_P, null, null, CommonConstants.CONSUME_STATUS_P);
}
```
