# D-003 接口阶段包含预下单与文档口径不符

- **需求**: D
- **级别**: 重要
- **状态**: 待确认修复
- **发现日期**: 2026-04-03
- **违反边界**: 需求D规则4 "接口阶段只受理并落数据"

## 问题描述

02 接口 `batchAddTransferTi02Data` 内部调了 `processSingleTransferTi`，后者执行了 `transTransferService.transTransferTiPre`（外部预下单）。需求文档说接口阶段"只受理并落数据"，但实际还做了外部交互。

## 具体位置

- `TransTransferTiBatchBusinessServiceImpl.java` :779-799

## 当前行为

接口受理 → 落明细 + 落转账表 → **调前置预下单** → status=S

## 建议处理

**需确认**: 是改代码去掉接口阶段的预下单（推迟到 task 阶段），还是更新文档口径确认当前行为合理。如果 task 的 `processDetail02` 依赖预下单结果（:2954-2960 查 transferInfo），则改代码需要同步调整 task 阶段。
