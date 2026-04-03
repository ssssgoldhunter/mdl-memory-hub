# D-005 processDetail02 无记录级分布式锁

- **需求**: D
- **级别**: 重要
- **状态**: 待确认修复
- **发现日期**: 2026-04-03

## 问题描述

`processDetail02` 方法内部只有卡级 Redis 锁（`executeSingleDebit02`/`executeSingleCredit02` 内部），没有 `transNo` 级别的锁。并发补偿 + task 轮次可同时进入同一 transNo 的处理。

## 具体位置

- `TransTransferTiBatchBusinessServiceImpl.java` :2935-3035

## 当前代码

```java
public DefaultResult<Boolean> processDetail02(String transNo) {
    // 无 transNo 级 Redis 锁
    TransTransferTiBatchDetailQueryRes detail = getDetailByTransNo(transNo);
    // ...
    if (!debitDone) {
        String debitResult = executeSingleDebit02(detail, transferInfo);  // 内部只有卡锁
    }
}
```

## 建议修复

在方法入口加 `transNo` 级 Redis 锁，如 `process_detail_02_lock:{transNo}`，确保同一 transNo 不会被并发执行。
