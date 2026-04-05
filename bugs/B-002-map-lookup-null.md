# B-002 map lookup 返回 null 无检查

- **需求**: B
- **级别**: 重要
- **状态**: 已修复
- **发现日期**: 2026-04-04

## 问题描述

`DeductionTrans.createSubConsume` 从 `slot.getBusinessInfos()` 和 `slot.getCompayInfoMaps()` 取值时无 null 检查。如果收款卡的业务信息或公司信息未加载到 slot 中，后续直接调用 `busInfo.getSecondOrgCode()` 等方法将 NPE。

此时主交易记录已落库，无法回滚。

## 具体位置

- `DeductionTrans.java` :231-233

## 当前代码

```java
BasBusinessInfoQueryRes busInfo = slot.getBusinessInfos().get(subTransVo.getReceiveCardCode());
BasCompanyInfoQueryRes recCompanyInfo = slot.getCompayInfoMaps().get(subTransVo.getReceiveCardCode());
// 无 null 检查，直接使用 busInfo/recCompanyInfo
```

## 建议修复

增加 null 检查，未找到时抛明确业务异常。
