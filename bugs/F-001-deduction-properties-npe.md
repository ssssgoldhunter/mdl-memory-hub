# F-001 HttpDeductionMessageConsumeHandle properties 空指针

- **需求**: Front
- **级别**: 高
- **状态**: 已修复
- **发现日期**: 2026-04-05

## 问题描述

`HttpDeductionMessageConsumeHandle` 中调用 `deductionNotifyDto.getProperties().get(FrontConstants.TRACE_ID)`，但 `BaseMessage.properties` 字段未做 null 初始化，如果消息中没有 properties，会抛出 NPE。

## 具体位置

- `HttpDeductionMessageConsumeHandle.java` :53

## 当前代码

```java
baseMessageVo.setMsgId(deductionNotifyDto.getProperties().get(FrontConstants.TRACE_ID));
```

## 建议修复

增加 null 检查：
```java
Map<String, String> properties = deductionNotifyDto.getProperties();
if (properties != null && properties.containsKey(FrontConstants.TRACE_ID)) {
    baseMessageVo.setMsgId(properties.get(FrontConstants.TRACE_ID));
}
```

## 备注

`HttpTransferMessageConsumeHandle` 第 58 行存在相同问题。
