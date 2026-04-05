# F-002 HttpDeductionMessageConsumeHandle HTTP 响应 body 空指针

- **需求**: Front
- **级别**: 高
- **状态**: 待确认修复
- **发现日期**: 2026-04-05

## 问题描述

`HttpDeductionMessageConsumeHandle` 中处理 HTTP 响应时，未对 `body` 和 `resultBody` 做 null 检查：

1. `httpResult.getBody()` 可能返回 null
2. `JSONObject.parseObject(body)` 当 body 为 null 或空字符串时可能返回 null
3. 第 82 行直接调用 `resultBody.getString("code")` 未对 resultBody 做 null 检查

## 具体位置

- `HttpDeductionMessageConsumeHandle.java` :80-82

## 当前代码

```java
if (statusCode == HttpStatus.OK || statusCode == HttpStatus.FOUND) {
    JSONObject resultBody = JSONObject.parseObject(body);
    if (FrontConstants.HTTP_DEFAULT_CODE_SUCCESS.equals(resultBody.getString("code"))) {
        // resultBody 可能为 null
    }
}
```

## 建议修复

增加 null 检查：
```java
if (statusCode == HttpStatus.OK || statusCode == HttpStatus.FOUND) {
    if (StringUtils.isBlank(body)) {
        log.warn("HTTP response body is blank");
        return;
    }
    JSONObject resultBody = JSONObject.parseObject(body);
    if (resultBody == null) {
        log.warn("HTTP response body parse failed");
        return;
    }
    ...
}
```

## 备注

`HttpTransferMessageConsumeHandle` 第 85-86 行存在相同问题。