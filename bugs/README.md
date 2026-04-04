# Bug 清单

> 按需求边界值 review 发现的问题，状态跟踪

## 状态定义

| 状态 | 含义 |
|------|------|
| 待确认修复 | 已发现问题，等待确认是否修复 |
| 确认修复 | 已确认需要修复，待开发 |
| 修复中 | 正在修复 |
| 已修复 | 修复完成，待验证 |
| 已验证 | 已验证通过 |
| 不修复 | 确认不需要修复（需注明原因） |
| 已确认非缺陷 | 已核对代码与业务口径，确认不是缺陷 |

## 问题索引

| # | 需求 | 级别 | 问题 | 状态 | 文件 |
|---|------|------|------|------|------|
| BC-1 | B+C | 严重 | 冻结池无并发保护，双重消耗风险 | 已修复 | [BC-001](./BC-001-frozen-pool-concurrency.md) |
| BC-2 | B+C | 严重 | BAfter二次校验冻结池，前置成功后可能抛异常导致不一致 | 已修复 | [BC-002](./BC-002-bafter-double-validate.md) |
| BC-3 | B+C | 重要 | FROZEN_TYPE_D 硬编码未收敛到 CommonConstants | 已修复 | [BC-003](./BC-003-frozen-type-d-constant.md) |
| BC-4 | B+C | 重要 | 第二个 subRec 最终状态未更新 | 已修复 | [BC-004](./BC-004-second-subrec-status.md) |
| D-1 | D | 严重 | 冻结主流水号唯一性未校验 | 已修复 | [D-001](./D-001-frozen-trans-no-unique.md) |
| D-2 | D | 严重 | processDetail02 缺少 status=P 中间态 | 已修复 | [D-002](./D-002-missing-status-p.md) |
| D-3 | D | 重要 | 接口阶段包含预下单，已确认属于设计口径 | 已确认非缺陷 | [D-003](./D-003-interface-pre-order.md) |
| D-4 | D | 重要 | 02待处理查询缺 transferMode 过滤 | 不修复 | [D-004](./D-004-query-missing-transfer-mode.md) |
| D-5 | D | 重要 | processDetail02 无记录级分布式锁 | 已修复 | [D-005](./D-005-no-record-level-lock.md) |
| D-6 | D | 重要 | isFrozen 空值静默默认N | 已修复 | [D-006](./D-006-isfrozen-silent-default.md) |
| D-7 | D | 重要 | batchAddTransferTi02Data 异步执行无事务保障 | 不修复 | [D-007](./D-007-async-no-transaction.md) |
| E-1 | E | 严重 | 失败明细未跳过，task重试导致双重扣款 | 已修复 | [E-001](./E-001-failed-detail-retry.md) |
| E-2 | E | 严重 | markProcessing 缺 frontStatus=P | 已修复 | [E-002](./E-002-missing-front-status-p.md) |
| E-3 | E | 重要 | 明细状态更新无CAS | 已修复 | [E-003](./E-003-detail-status-no-cas.md) |
| E-4 | E | 重要 | 重试路径冻结/解冻可能重复 | 已修复 | [E-004](./E-004-retry-frozen-duplicate.md) |
| E-5 | E | 重要 | transSubType="U" 需确认与单笔B一致 | 已确认非缺陷 | [E-005](./E-005-trans-sub-type-u.md) |
| AF-1 | A+Front | 重要 | WithDrawRuleCheck isAccess 不检查 needRuleCheck | 已修复 | [AF-001](./AF-001-withdraw-rule-check-access.md) |
| AF-2 | A+Front | 重要 | Front 通知 catch Exception 吞掉错误 | 已修复 | [AF-002](./AF-002-front-catch-swallow.md) |
| AF-3 | A+Front | 重要 | messageMaxRetries 注入但未使用 | 已修复 | [AF-003](./AF-003-message-max-retries-unused.md) |
