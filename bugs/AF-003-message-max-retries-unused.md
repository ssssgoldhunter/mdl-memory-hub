# AF-003 messageMaxRetries 注入但未使用

- **需求**: Front
- **级别**: 重要
- **状态**: 待确认修复
- **发现日期**: 2026-04-03

## 问题描述

所有 `HttpXxxMessageConsumeHandle` 注入了 `@Value("${app.message-retry.max-retries}") int messageMaxRetries`，但该字段只出现在日志中，不在重试判断逻辑中。重试完全依赖 24h 时间窗口，无次数上限。

## 具体位置

- 所有 `HttpXxxMessageConsumeHandle` 的 `messageMaxRetries` 字段声明

## 当前代码

```java
@Value("${app.message-retry.max-retries}") int messageMaxRetries;
// 仅在日志中使用，如：
log.info("retry count: {}, max retries: {}", retryCount, messageMaxRetries);
// 不参与重试终止判断
```

## 建议修复

在重试逻辑中加入次数判断：超过 `messageMaxRetries` 后停止重试并标记失败。或如果确认只用时间窗口，移除该字段。
