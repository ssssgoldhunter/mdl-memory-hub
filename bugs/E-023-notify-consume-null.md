# E-023 E 批量扣款通知 sendDeductionNotify 未检查 consume 是否为 null

- **需求**: E
- **级别**: 高
- **状态**: 已修复
- **发现日期**: 2026-04-05

## 问题描述

`sendDeductionNotifyAsync` 方法只检查了 `detail` 和 `payCompanyInfo` 是否为 null，未检查 `consume` 是否为 null。

后续 `sendDeductionNotify` 中直接调用 `context.consume.getConsumeId()` 会抛出 NPE。

## 具体位置

- `DeductionBatchExecuteServiceImpl.java` :1002-1011 (sendDeductionNotifyAsync)
- `DeductionBatchExecuteServiceImpl.java` :1034 (直接使用 consume)

## 当前代码

```java
private void sendDeductionNotifyAsync(ExecutionContext context, String status) {
    if (context == null || context.detail == null || context.payCompanyInfo == null) {
        log.info("E批量扣款通知跳过：上下文不完整，status={}", status);
        return;
    }
    // 未检查 consume
    ...
}

// 后续使用
dto.setBizSeqNo(context.consume.getConsumeId());  // NPE 风险
```

## 建议修复

在 sendDeductionNotifyAsync 检查中增加 `context.consume == null` 判断。
