# B-006 B 扣款 transDeduction 缺少 MAC 幂等校验

- **需求**: B
- **级别**: 高
- **状态**: 待确认修复
- **发现日期**: 2026-04-05

## 问题描述

`TransConsumeServiceImpl.transDeduction` 方法缺少像 `transConsumeFree` 和 `transConsume` 方法中的 MAC 幂等校验逻辑。

如果同一笔扣款请求重复提交，可能导致重复扣款。

## 具体位置

- `TransConsumeServiceImpl.java` :454-555 (transDeduction)

## 对比

`transConsumeFree` 和 `transConsume` 方法都有：
```java
// MAC 幂等校验
if (StringUtils.isNotBlank(request.getMac())) {
    ConsumeTransVo cacheVo = macCache.get(request.getMac());
    if (cacheVo != null) {
        // 返回已存在的交易
    }
}
```

但 `transDeduction` 方法没有此校验。

## 建议修复

参考 transConsumeFree 添加 MAC 幂等校验逻辑。