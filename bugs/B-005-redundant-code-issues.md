# B-005 冗余代码：无效 merchantInfo re-put 和冗余账户刷新

- **需求**: B
- **级别**: 一般
- **状态**: 已修复
- **发现日期**: 2026-04-04

## 问题描述

三处冗余代码：

1. `DeductionTrans:311-313` — merchantInfo 从 map 取出后又以相同 key 放回，属于无意义操作
2. `DeductionTransAfter:72-73` — paySub/recSub 初始获取后未使用，在 refresh 后重新赋值
3. `DeductionFrozenPoolSupport:73` — frozenFlag 判定逻辑与 FrozenPoolHelper 不一致（变量来源不同但语义相同），维护时容易分歧

## 具体位置

- `DeductionTrans.java` :311-313
- `DeductionTransAfter.java` :72-87
- `DeductionFrozenPoolSupport.java` :73

## 建议修复

1. 移除无意义的 re-put 或补充应有逻辑
2. 移除未使用的初始 paySub/recSub 获取
3. 统一 frozenFlag 判定到 FrozenPoolHelper
