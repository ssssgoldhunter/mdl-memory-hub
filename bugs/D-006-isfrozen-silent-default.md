# D-006 isFrozen 空值静默默认N

- **需求**: D
- **级别**: 重要
- **状态**: 已修复
- **发现日期**: 2026-04-03

## 问题描述

`normalizeMode02Request` 在 `isFrozen` 为空时静默设为 N，调用方漏传不会报错，直接走不冻结路径。

## 具体位置

- `TransTransferTiBatchBusinessServiceImpl.java` :363-368

## 当前代码

```java
private void normalizeMode02Request(List<ScTransferTiReq> request) {
    for (ScTransferTiReq req : request) {
        if (StringUtils.isBlank(req.getIsFrozen())) {
            req.setIsFrozen(CommonConstants.ALLOW_FLAG_N);
        }
    }
}
```

## 建议修复

改为显式必传校验，`isFrozen` 为空时抛参数异常拒绝请求。
