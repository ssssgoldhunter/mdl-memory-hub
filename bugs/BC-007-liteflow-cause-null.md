# BC-007 liteflowResponse.getCause() 为 null 时异常处理不当

- **需求**: B+C
- **级别**: 高
- **状态**: 待确认修复
- **发现日期**: 2026-04-05

## 问题描述

`TransFrozenServiceImpl` 的 `transAccountFrozen` 和 `transAccountUFrozen` 方法中，catch 块调用 `liteflowResponse.getCause().getMessage()`，但 `getCause()` 可能为 null，导致 NPE。

## 具体位置

- `TransFrozenServiceImpl.java` :60-65
- `TransFrozenServiceImpl.java` :142-148

## 当前代码

```java
} catch (Exception e) {
    log.error("transAccountFrozen error: {}", e.getMessage(), e);
    if (liteflowResponse != null && !liteflowResponse.isSuccess()) {
        throw new BaseException(ApiConstants.RESULT_FAIL, 
            liteflowResponse.getCause().getMessage());  // getCause() 可能为 null
    }
    ...
}
```

## 建议修复

增加 getCause() 的 null 检查：
```java
Throwable cause = liteflowResponse.getCause();
String errorMsg = cause != null ? cause.getMessage() : "流程执行失败";
throw new BaseException(ApiConstants.RESULT_FAIL, errorMsg);
```