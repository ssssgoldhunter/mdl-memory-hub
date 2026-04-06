# F-003 MessageSendServiceImpl messageTypeTopic 空指针

- **需求**: Front
- **级别**: 高
- **状态**: 已修复
- **发现日期**: 2026-04-05

## 问题描述

`MessageSendServiceImpl.sendRendTopicMessageDelay` 方法中，当 `messageTypeTopic == null` 时只打印日志但未 return，第 146 行直接调用 `request.getMessageTypeTopic().getTopic()` 会抛出 NPE。

## 具体位置

- `MessageSendServiceImpl.java` :142-146

## 当前代码

```java
@Override
public void sendRendTopicMessageDelay (BasMessageRequest request){
    if (request.getMessageTypeTopic() == null) {
        log.info("sendTopicMessageDelay: Topic: {}, BusinessKey: {}, MsgId: {}, Content: {}, topic empty",
                request.getTopic(), request.getBusinessKey(), request.getMsgId(), request.getBusinessKey());
        // 未 return，继续执行
    }
    request.setTopic(request.getMessageTypeTopic().getTopic());  // NPE
```

## 建议修复

在日志输出后增加 return：
```java
if (request.getMessageTypeTopic() == null) {
    log.info("sendTopicMessageDelay: Topic: {}, ... topic empty", ...);
    return;  // 增加此行
}
```
