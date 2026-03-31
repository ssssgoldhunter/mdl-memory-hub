# 历史索引

更新时间：2026-03-31 10:18:00 CST

## 1. 当前应优先看的文档

1. `docs/superpowers/mdl-supply-chain-abcd/00-current-baseline.md`
2. `docs/superpowers/mdl-supply-chain-abcd/01-code-state.md`
3. `docs/superpowers/mdl-supply-chain-abcd/02-verification-checklist.md`

## 2. 历史设计稿

- `requirements/2026-03-26-abcd-front-design-confirmed.md`
  - 作用：保留最初确认稿
  - 注意：不再直接作为当前实现依据

- `requirements/2026-03-27-abcd-front-implementation-baseline.md`
  - 作用：保留 2026-03-27 中途收口稿
  - 注意：已有多处被 2026-03-31 后续代码和文档纠偏覆盖
  - 当前只用于追溯为什么曾经出现：
    - `B` 被写成 `P`
    - `D02` 被写成两接口执行
    - `D02` 被误挂到非划付域

## 3. 历史对话日志

- `conversation-logs/2026-03-26.md`
  - 作用：保留中途阶段性实现记录
  - 注意：里面包含已经被后续推翻的过渡方案

- `conversation-logs/2026-03-27.md`
  - 作用：保留 3 月 27 日的纠偏记录
  - 注意：后半段关于 `D02` 两接口执行、`data-batch` 等描述已过期

## 4. 相关专题需求文档

- `requirements/req-b-deduction-api/`
- `requirements/req-c-frozen-refactor/`
- `requirements/req-d-transfer-mode-refactor/`

## 4.1 后续扩展设计稿

- `docs/superpowers/specs/2026-03-31-e-batch-deduction-design.md`
  - 作用：`E` 批量扣款需求设计稿
  - 说明：`E` 是基于当前 `B` 扣款语义 + `D02` 执行模型的后续扩展
  - 当前用于后续讨论和实现参考

## 5. 当前使用规则

- 后续续聊时，先看本模块
- 需要追溯为什么变更，再看历史设计稿和旧对话
- 如果本模块与旧文档冲突，以本模块为准
- 如果 `requirements/req-*` 的 `summary.md` 与 `open-questions.md` 冲突，以 `summary.md` 为准
