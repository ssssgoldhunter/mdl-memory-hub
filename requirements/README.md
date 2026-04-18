# 需求记录目录

该目录用于沉淀和持续维护需求讨论结果，避免会话中断后需求背景丢失。

## 目录约定

- `overview.md`
  - 记录当前阶段所有需求的依赖关系、优先级和推进顺序
- `req-*/summary.md`
  - 记录单个需求的目标、边界、当前结论
- `req-*/discussion-log.md`
  - 记录按时间追加的讨论过程、关键决策、未决问题
- `req-*/open-questions.md`
  - 记录仍待确认的问题
- `req-*/design.md`
  - 记录确认后的设计方案

## 当前需求清单

- `docs/superpowers/mdl-supply-chain-abcd/`
  - 当前 `mdl` 供应链 A/B/C/D/front 续聊主入口
- `2026-03-27-abcd-front-implementation-baseline.md`
  - 当前 A/B/C/D/front 实现基线，优先级高于旧确认稿

### 需求状态（2026-04-18 更新）

| 需求 | 名称 | 状态 |
|------|------|------|
| A | 提现接口及规则改造 | 已完成主链（清结算查询 TODO） |
| B | 新增扣款接口 | ✅ 已完成 |
| C | 原冻结交易改造 | ✅ 已完成 |
| D | 划付模式改造 | ✅ 已完成 |
| E | 批量扣款 | ✅ 已完成（测试通过） |
| front | B/D/银行实收通知 | 已完成（清结算通知 TODO） |

### 需求详情目录

- `req-b-deduction-api/` — 扣款接口详情
- `req-c-frozen-refactor/` — 冻结交易改造详情
- `req-d-transfer-mode-refactor/` — 划付模式改造详情

## 维护规则

- 每次需求讨论后，先更新对应 `discussion-log.md`
- 讨论形成稳定结论后，回写 `summary.md`
- 设计定稿后补充 `design.md`
- 如果一个需求会影响其他需求，要同步更新 `overview.md`
