# D-003 接口阶段包含预下单，已确认属于设计口径

- **需求**: D
- **级别**: 重要
- **状态**: 已确认非缺陷
- **发现日期**: 2026-04-03
- **结论**: 需求D两阶段口径已修正为“第一阶段落地明细数据 + 转账数据，第二阶段 task 处理账户变动”

## 背景说明

02 接口 `batchAddTransferTi02Data` 内部调了 `processSingleTransferTi`，后者执行了 `transTransferService.transTransferTiPre`，在接口阶段完成转账数据预创建。

## 具体位置

- `TransTransferTiBatchBusinessServiceImpl.java` :779-799

## 当前行为

接口受理 → 落明细 + 落转账表 → **调前置预下单** → status=S

## 确认结论

当前实现符合已确认的两阶段设计：

- 第一阶段：接口落地明细数据 + 转账数据
- 第二阶段：task 执行账户变动、冻结落库、状态收口、通知

因此这条不是代码缺陷，而是旧文档口径过时。应修正文档，不应删除接口阶段的预下单逻辑。
