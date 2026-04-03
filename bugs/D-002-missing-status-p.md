# D-002 processDetail02 缺少 status=P 中间态

- **需求**: D
- **级别**: 严重
- **状态**: 待确认修复
- **发现日期**: 2026-04-03
- **违反边界**: 需求D规则9 "Task starts payer side: to_status=P, ti_status=I, status=P"

## 问题描述

`processDetail02` 入口只检查了 `status != S` 就抛异常，但没有在开始处理前将 detail 的顶层 status 从 S 推进到 P。实际执行中，`toStatus` 和 `tiStatus` 分别被推进，但**顶层 status 始终停留在 S，从未经过 P**。

## 具体位置

- `TransTransferTiBatchBusinessServiceImpl.java` :2976-2994

## 当前代码

```java
// :2949 检查 status 必须为 S
if (!CommonConstants.CONSUME_STATUS_S.equals(detail.getStatus())) {
    throw new BaseException(..., "流水状态必须为成功状态(S)");
}

// :2979 直接开始下账，没有先把 status 推进到 P
if (!debitDone) {
    String debitResult = executeSingleDebit02(detail, transferInfo);
    // ...
}
```

## 建议修复

在 `executeSingleDebit02` 之前加：
```java
updateDetailStatus(detail.getTransNo(), CommonConstants.CONSUME_STATUS_P, "开始处理");
```
