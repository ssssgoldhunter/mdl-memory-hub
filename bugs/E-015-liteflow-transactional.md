# E-015 @Transactional 在 LiteFlow NodeComponent 上可能不生效

- **需求**: E
- **级别**: 重要
- **状态**: 不修复
- **发现日期**: 2026-04-04

## 问题描述

`DeductionBatchPreCreate.process()` 方法上标注了 `@Transactional(rollbackFor = Exception.class)`。LiteFlow 通过自身框架调用 NodeComponent，可能不走 Spring AOP 代理链，导致 `@Transactional` 静默失效。

如果事务不生效，`createConsumeSkeleton` 内四步 save 操作（三张表）不是原子的，任一步失败会导致部分写入。

## 具体位置

- `DeductionBatchPreCreate.java` :51

## 当前代码

```java
@Transactional(rollbackFor = Exception.class)
@Override
public void process() {
    // 四步 save 操作
}
```

## 结论

当前按业务口径保持现状，不修复。

原因：
- 这里涉及两个场景下的操作，不作为当前一轮统一改造项处理
- 现阶段先维持现有调用方式，后续如出现明确事务边界问题再专项处理

## 建议修复

验证 LiteFlow 是否支持 Spring 事务代理。如果不支持，将事务逻辑下沉到 Service 层方法中。
