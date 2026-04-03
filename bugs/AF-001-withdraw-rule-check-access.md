# AF-001 WithDrawRuleCheck isAccess 不检查 needRuleCheck

- **需求**: A
- **级别**: 重要
- **状态**: 待确认修复
- **发现日期**: 2026-04-03

## 问题描述

`WithDrawRuleCheck.isAccess()` 不检查 `needRuleCheck`，节点仍被进入，只是在 `process()` 中 early return。需求说 `needRuleCheck=false` 时"直接跳过节点"。

## 具体位置

- `WithDrawRuleCheck.java` :34-38

## 当前代码

```java
@Override
public boolean isAccess() {
    TransSlot slot = this.getFirstContextBean();
    return slot != null
            && slot.getWithDrawTransVo() != null
            && CommonConstants.TRANS_TYPE_WD.equals(slot.getTransType());
    // 没有检查 needRuleCheck
}
```

## 建议修复

将 `needRuleCheck` 判断移入 `isAccess()`：
```java
return slot != null
    && slot.getWithDrawTransVo() != null
    && Boolean.TRUE.equals(slot.getWithDrawTransVo().getNeedRuleCheck())
    && CommonConstants.TRANS_TYPE_WD.equals(slot.getTransType());
```
