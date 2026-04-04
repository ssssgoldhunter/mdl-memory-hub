# AF-002 Front 通知 catch Exception 吞掉错误

- **需求**: Front
- **级别**: 重要
- **状态**: 已修复
- **发现日期**: 2026-04-03

## 问题描述

所有 `HttpXxxMessageConsumeHandle` 的 `processMessage()` 方法 catch Exception 后只记日志，然后正常返回。RocketMQ 视为消费成功不会重投。如果 JSON 解析失败等瞬时错误，消息丢失。

## 具体位置

- `HttpConsumeMessageConsumeHandle.java` :98-100
- `HttpTransferMessageConsumeHandle.java` :91-93
- `HttpWithDrawMessageConsumeHandle.java` :98-100
- `HttpRechargeMessageConsumeHandle.java` :98-100

## 当前代码

```java
catch (Exception e) {
    log.error("processMessage error", e);
}
// 正常返回，MQ 认为消费成功
```

## 建议修复

catch 中返回 `RECONSUME_LATER`，让 MQ 重投消息。或在 catch 中区分可重试异常和不可重试异常。
