# mdl 供应链 A/B/C/D 专用模块

更新时间：2026-03-31 10:10:00 CST

## 用途

这个模块只服务当前 `mdl` 供应链改造续聊，统一收口以下内容：

- 当前唯一有效的实现基线
- 当前代码状态与边界
- 当前验证与联调清单
- 历史设计与对话日志索引

后续如果继续讨论或开发这组需求，优先从本目录开始，不再先翻散落的 `requirements/`、`conversation-logs/`。

## 所属大计划

- 大计划名称：
  - `mdl定制`
- 当前这组 `A/B/C/D/front` 文档，统一视为 `mdl定制` 下的供应链子计划

## 目录结构

- `00-current-baseline.md`
  - 当前唯一有效的实现/设计基线
- `01-code-state.md`
  - 当前代码已经做到什么，哪些能力已落地
- `02-verification-checklist.md`
  - 当前需要验证什么，哪些仍待外部条件
- `03-history-index.md`
  - 历史设计稿、历史对话日志、相关需求文档索引

## 当前结论

- 当前实现边界以 [`00-current-baseline.md`](./00-current-baseline.md) 为准
- 当前代码状态以 [`01-code-state.md`](./01-code-state.md) 为准
- 当前验证动作以 [`02-verification-checklist.md`](./02-verification-checklist.md) 为准
- 历史资料仅作为追溯，不再作为当前开发依据

## 固定阅读顺序

1. `00-current-baseline.md`
2. `01-code-state.md`
3. `02-verification-checklist.md`
4. 需要追溯时再看 `03-history-index.md` 和 `requirements/req-*`

## 最容易读偏的点

- `D` 全部属于“划付域”，不属于原转账链
- `B` 主交易类型是 `D`，不是 `P`
- `B` 的账户变动明细类型是 `DC/DR`，不是 `PC/PR`
- `B useFrozen=false` 与 `B useFrozen=true` 是两套不同模型，不能混看
