# E-001 失败明细未跳过，task重试导致双重扣款

- **需求**: E
- **级别**: 严重
- **状态**: 已修复
- **发现日期**: 2026-04-03
- **违反边界**: 需求E规则16 状态机

## 问题描述

`processDeductionDetail02` 只跳过 `status=S`，不跳过 `status=F`。失败明细在下一轮 task 会被重新执行，再次走前置调用导致双重扣款。

## 具体位置

- `DeductionBatchExecuteServiceImpl.java` :124-127

## 当前代码

```java
if (CommonConstants.CONSUME_STATUS_S.equals(context.detail.getStatus())) {
    return buildSuccessResult();
}
// status=F 的明细不会被跳过，继续执行
```

## 建议修复

加 `status=F` 跳过守卫：
```java
if (CommonConstants.CONSUME_STATUS_S.equals(context.detail.getStatus())
    || CommonConstants.CONSUME_STATUS_F.equals(context.detail.getStatus())) {
    return buildSuccessResult();
}
```
