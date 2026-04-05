# D-011 D02 task processDetail 未处理返回值

- **需求**: D
- **级别**: 高
- **状态**: 待确认修复
- **发现日期**: 2026-04-05

## 问题描述

`TransferTi02TaskJobService.processDetail` 调用 `processDetail02` 后只记录日志，未检查返回结果的 `success` 状态。

如果 `processDetail02` 返回 `success=false`（如锁获取失败返回 `CARD_CODE_BUSY_ERR`），不会抛异常，不会触发 `markDetailFailed`，导致：
- 明细状态保持 `S`，下次继续尝试处理
- 可能导致重复处理或状态不一致

## 具体位置

- `TransferTi02TaskJobService.java` :150-152

## 当前代码

```java
DefaultResult<Boolean> result = transTransferTiBatchBusinessApi.processDetail02(transNo);
log.info("TransferTi02TaskJobService:processDetail02 result, transNo:{}, result:{}",
        transNo, JSONObject.toJSONString(result));
// 未检查 result.isSuccess()
```

## 建议修复

检查返回值，失败时调用 markDetailFailed：
```java
if (!result.isSuccess() || !Boolean.TRUE.equals(result.getModel())) {
    markDetailFailed(detail, result.getMessage());
}
```