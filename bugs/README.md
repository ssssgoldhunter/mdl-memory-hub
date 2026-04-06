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
| E-6 | E | 严重 | frozenResult 返回 null 时 NPE（三处） | 已修复 | [E-006](./E-006-frozen-result-npe.md) |
| E-7 | E | 严重 | 余额已改但明细未落，无回滚 | 不修复 | [E-007](./E-007-balance-detail-no-rollback.md) |
| E-8 | E | 严重 | 冻结两步写非原子，第二步失败无补偿 | 不修复 | [E-008](./E-008-frozen-two-step-no-rollback.md) |
| E-9 | E | 严重 | task 分页扫描可能重复拾取 status=P 的明细 | 已修复 | [E-009](./E-009-task-scan-refetch-p.md) |
| E-10 | E | 重要 | 失败处理中状态更新可能部分成功 | 已修复 | [E-010](./E-010-fail-handler-partial-status.md) |
| E-11 | E | 重要 | 预落地失败后明细卡在F，无重试机制 | 已修复 | [E-011](./E-011-failed-detail-no-retry.md) |
| E-12 | E | 重要 | transNo 存在性检查与插入之间无DB唯一约束 | 不修复 | [E-012](./E-012-transno-toctou-race.md) |
| E-13 | E | 重要 | 分页字段名不匹配（current vs pageNum） | 已确认非缺陷 | [E-013](./E-013-pagination-field-mismatch.md) |
| E-14 | E | 重要 | handleReceiveSide 只刷新不验证 | 已修复 | [E-014](./E-014-receive-side-no-verify.md) |
| E-15 | E | 重要 | @Transactional 在 LiteFlow NodeComponent 上可能不生效 | 不修复 | [E-015](./E-015-liteflow-transactional.md) |
| E-16 | E | 一般 | transTime substring 无长度校验 | 已修复 | [E-016](./E-016-transtime-substring-validation.md) |
| E-17 | E | 一般 | amount 为 null 或零/负数时校验不充分 | 已修复 | [E-017](./E-017-amount-validation-gap.md) |
| B-1 | B | 严重 | subTransList 无空检查导致部分写入 | 已修复 | [B-001](./B-001-subtranslist-npe.md) |
| B-2 | B | 重要 | map lookup 返回 null 无检查 | 已修复 | [B-002](./B-002-map-lookup-null.md) |
| B-3 | B | 重要 | 子交易 org/operator 字段使用了收款方业务信息 | 不修复 | [B-003](./B-003-wrong-org-fields.md) |
| B-4 | B | 重要 | frozenAmt 为 null 时 subtract NPE | 已修复 | [B-004](./B-004-frozenamt-null-subtract.md) |
| B-5 | B | 一般 | 冗余代码：无效 re-put 和冗余刷新 | 已修复 | [B-005](./B-005-redundant-code-issues.md) |
| BC-5 | B+C | 重要 | FrozenPoolHelper getFrozenAmt 为 null 时 NPE | 已修复 | [BC-005](./BC-005-frozenamt-null-in-helper.md) |
| E-18 | E+B+C | 严重 | FrozenPoolHelper.resolveAllowFlag getFrozenAmt NPE（BC-005 遗漏） | 已修复 | [E-018](./E-018-resolve-allow-flag-npe.md) |
| E-19 | E | 严重 | E 批量扣款执行成功/失败后无通知 | 已修复 | [E-019](./E-019-no-notification.md) |
| E-20 | E | 重要 | updateConsumeRecAccChangeStatusByRecId 无 CAS 保护 | 不修复 | [E-020](./E-020-rec-status-no-cas.md) |
| E-21 | E | 重要 | resetFailedDeductionDetail 设置的字段被 update 静默丢弃 | 不修复 | [E-021](./E-021-reset-fields-discarded.md) |
| E-22 | E | 一般 | rootRec/childRec 状态更新无 CAS 守卫 | 不修复 | [E-022](./E-022-rec-update-no-cas.md) |
| D-8 | D | 重要 | D02 task processDetail 未捕获异常，一条失败中止扫描 | 已修复 | [D-008](./D-008-task-exception-aborts-scan.md) |
| D-9 | D | 一般 | D02 task 明细在扫描和执行中各查一次 | 不修复 | [D-009](./D-009-detail-double-query.md) |
| D-10 | D | 严重 | markDetailFailed 未设置 status 导致循环失败重试 | 已修复 | [D-010](./D-010-mark-detail-failed-no-status.md) |
| D-11 | D | 高 | processDetail 未处理返回值，锁失败时不触发失败标记 | 待确认修复 | [D-011](./D-011-process-detail-no-result-check.md) |
| E-23 | E | 高 | sendDeductionNotify 未检查 consume 是否为 null | 待确认修复 | [E-023](./E-023-notify-consume-null.md) |
| B-6 | B | 高 | transDeduction 缺少 MAC 幂等校验 | 待确认修复 | [B-006](./B-006-deduction-no-mac-idempotent.md) |
| BC-6 | B+C | 高 | FrozenTrans/UnFrozenTrans findFirst().get() 未处理空 | 待确认修复 | [BC-006](./BC-006-findfirst-no-empty-check.md) |
| BC-7 | B+C | 高 | liteflowResponse.getCause() 为 null 时异常处理不当 | 待确认修复 | [BC-007](./BC-007-liteflow-cause-null.md) |
| F-1 | Front | 高 | HttpDeductionMessageConsumeHandle properties NPE | 已修复 | [F-001](./F-001-deduction-properties-npe.md) |
| F-2 | Front | 高 | HTTP 响应 body 空指针 | 已修复 | [F-002](./F-002-http-response-body-npe.md) |
| F-3 | Front | 高 | MessageSendServiceImpl messageTypeTopic NPE | 已修复 | [F-003](./F-003-message-topic-npe.md) |
