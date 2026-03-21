# 代码合并变更分析报告

> 日期：2026-03-20
> 项目：bwcj (从另一项目迁移代码)
> 分析方式：git diff --ignore-all-space (过滤空格/回车变化)

---

## 1. 变更统计汇总

### 总体统计

| 类型 | 数量 | 说明 |
|------|------|------|
| **已回退空格变化** | **728** | ✅ 已处理 |
| **有真实代码改动** | **582** | 需要关注 |

### 按模块统计 (真实代码变更)

| 模块 | 变更文件数 |
|------|-----------|
| fund-catering-consume | 177 |
| fund-catering-report | 95 |
| fund-catering-web | 89 |
| fund-catering-base | 86 |
| fund-catering-front | 48 |
| common-core | 48 |
| fund-catering-task | 36 |
| starter-modules | 3 |
| **总计** | **582** |

### 变更类型分类

| 类型 | 数量 | 说明 |
|------|------|------|
| **A类: 删除** | 68 | 项目2原有代码被删除 |
| **B类: 新增** | 250 | 项目1的新代码 ✅ 安全 |
| **C类: 修改** | 264 | ⚠️ 可能冲突 |

---

## 2. A类：项目2原有代码被删除 (68个)

### 高关注度 (>100行删除)

| 删除行数 | 文件 |
|---------|------|
| 605 | `ZtBatchDateController.java` |
| 546 | `ConsumeTransRefundSplitPack.java` |
| 427 | `TzMonthBatchDateController.java` |
| 412 | `ZtDataController.java` |
| 354 | `README_MAC_IMPLEMENTATION.md` |
| 319 | `MybankConstants.java` |
| 212 | `README_BATCH_TRANSFER_IMPLEMENTATION.md` |
| 193 | `WebUtils.java` |
| 164 | `BillDataUtil.java` |
| 161 | `CreditBillDetailRes.java` |
| 152 | `DateValidatorUtil.java` |
| 147 | `StoreController.java` |

### 中等关注度 (50-100行删除)

| 删除行数 | 文件 |
|---------|------|
| 102 | `MybankApiExceptionEnum.java` |
| 91 | `HuaFuStatusEnum.java` |
| 86 | `ScStoreSyncReq.java` |
| 85 | `TestBatchNotifyMainController.java` |
| 84 | `JaxbUtil.java` |
| 75 | `PageResultDetailsResp.java` |
| 73 | `StoreJobService.java` |
| 71 | `WsPayTypeEnums.java` |

### 配置文件删除

| 文件 |
|------|
| `application-local.yml` (base/consume/front/task/web) |
| `bootstrap.yml` (front) |

---

## 3. B类： 项目1新增代码 (250个) ✅ 安全

### 主要新增文件 (>50行)

| 新增行数 | 文件 |
|--------|------|
| 191 | `TransAccountController.java` |
| 138 | `TransAccountApi.java` |
| 126 | `DaySumSubAccountTMapper.xml` |
| 88 | `RechargeTransController.java` |
| 88 | `BasCompanyInfoQueryRes.java` |
| 81 | `DaySumAmtController.java` |
| 76 | `DaySumSubAccountActivityTMapper.xml` |
| 70 | `TransTransferQueryMapper.xml` |
| 63 | `TransPlatformNotifyZxMapper.xml` |
| 61 | `TransTransferQueryServiceImpl.java` |
| 58 | `ConsumeRechargeDetailQueryRes.java` |
| 56 | `ScBusinessAccountResultDetailRes.java` |
| 53 | `TransAcctFrozenChangeDetailTMapper.xml` |
| 53 | `CreditRepaymentDetailMapper.xml` |
| 48 | `BasBindBankLogMapper.xml` |
| 44 | `TransferTransAuth.java` |
| 44 | `TransAcctChangeDetailT.java` |

---

## 4. C类: 可能冲突的文件 (264个) ⚠️

### 高风险 (>200行删除)

| 删除 | 新增 | 文件 |
|------|------|------|
| 1000 | 2221 | `AccountServiceImpl.java` |
| 435 | 891 | `ConsumeTransAfter.java` |
| 428 | 51 | `DaySumAmtServiceImpl.java` |
| 282 | 186 | `TransferRecallJobService.java` |
| 256 | 931 | `ConsumeTransRefundAfter.java` |
| 243 | 242 | `SignAuthInterceptor.java` |
| 227 | 358 | `TransRecallJobService.java` |
| 202 | 687 | `ConsumeTransRefundModel1After.java` |

### 中风险 (100-200行删除)

| 删除 | 新增 | 文件 |
|------|------|------|
| 160 | 159 | `ReportUnidentifiedRemittanceTQueryRes.java` |
| 159 | 42 | `CheckUtil.java` |
| 152 | 0 | `DateValidatorUtil.java` (已删除) |
| 147 | 5 | `StoreController.java` |
| 127 | 577 | `TransRecallServiceImpl.java` |
| 127 | 187 | `PlatformRechargeJobService.java` |
| 112 | 37 | `logback-spring.xml` (base) |
| 110 | 36 | `logback-spring.xml` (web/task/front) |
| 109 | 501 | `BasContractInfoMapper.xml` |
| 99 | 154 | `BillJobService.java` |
| 98 | 158 | `WithDrawTrans.java` |

---

## 5. 需要特别关注的变更

### 5.1 账户变动相关 (符合 PROJECT_MEMORY.md 排查规则)

**检查项**:
- [ ] `ConsumeTransAfter` 大重构是否正确调用 `BaseAccountServiceApi`
- [ ] `TransferTransAfter` 变更是否符合渠道失败处理规则
- [ ] `RefundRechargeTransAfter` 变更是否保持事务一致性
- [ ] `WithDrawTrans` 变更是否正确处理冻结/解冻

### 5.2 删除的文件影响

**需确认**:
1. `ConsumeTransRefundSplitPack.java` 删除 (-546行) - 功能是否已迁移?
2. `WsBankServiceApi.java` 删除 - 是否有替代实现?
3. `MybankConstants.java` 删除 (-319行) - 网商银行相关功能是否废弃?

### 5.3 大规模代码重构
**AccountServiceImpl.java** (+2119/-898):
- 大量代码重构，需确认业务逻辑正确性
- 可能涉及账户开立/查询核心逻辑

---

## 6. 建议下一步行动

1. **逐模块 Review**: 按 fund-catering-consume → fund-catering-base → fund-catering-task 顺序
2. **重点关注 Trans 组件**: 账户变动 API 调用是否统一
3. **验证删除文件**: 确认删除的类是否有引用残留
4. **运行测试**: 确保核心交易流程正常

---

## 7. 回退记录

### 2026-03-20 回退操作
- **已回退文件数**: 728 个 (只有空格/回车变化的文件)
- **剩余修改文件**: 523 个
- **剩余新增文件**: 229 个

### 回退前状态
- 修改文件: 1251 个
- 新增文件: 229 个

### 回退后状态
- 修改文件: 523 个
- 新增文件: 229 个

---

**生成时间**: 2026-03-20
**分析工具**: git diff --ignore-all-space
**回退时间**: 2026-03-20

---

## 2. 删除的文件 (共 72 个)

### common-core 模块删除

**删除的枚举类 (17个)**：
- `BindAndUnbindActionEnum.java`
- `BusiAccountReconciliationEnum.java`
- `ContractPaymentTypeEnum.java`
- `HuaFuStatusEnum.java`
- `ManualOrderAuditStatusEnum.java`
- `ManualOrderCreateStatus.java`
- `MpContractTemplateStateEnum.java`
- `MpStoreContractStateEnum.java`
- `MsgTypeEnums.java`
- `MybankApiExceptionEnum.java`
- `MybankConstants.java` (319行)
- `SettlementMethodEnum.java`
- `SettlementModeEnum.java`
- `TaskStatus.java`
- `WsPayTypeEnums.java`
- `OrganizationTypeEnum.java`
- `StoreSettleTypeEnum.java`
- `StoreStateEnum.java`
- `StoreTypeEnum.java`

**删除的工具类 (6个)**：
- `CDataAdapter.java`
- `FeishuWebHook.java`
- `JaxbUtil.java`
- `DateValidatorUtil.java`
- `CacheConstants.java` (部分)

### fund-catering-base 模块删除

- `WsBankServiceApi.java`
- `GdWarningMsg.java`
- `BasCompanyInfoMapper.java` (旧版本)
- `BasMerchantTMapper.xml`
- `BasAccountBankInfoServiceImpl.java`
- `application-local.yml`
- `bootstrap.yml`

### fund-catering-consume 模块删除

- `CreditBillDetailRes.java`
- `README_BATCH_TRANSFER_IMPLEMENTATION.md`
- `README_MAC_IMPLEMENTATION.md`
- `ConsumeTransRefundSplitPack.java`
- `application-local.yml`

### fund-catering-front 模块删除

- `FrontTransWSFacadeApi.java`
- `BatchCreateReq.java`
- `WsChannelConfig.java`
- `FrontTransWsFacadeController.java`
- `WsTransServiceImpl.java`
- `WsChannelService.java`
- `WsTransService.java`
- `WebUtils.java`
- `application-local.yml`
- `bootstrap.yml`

### fund-catering-report 模块删除

- `DashboardQueryConditionReq.java`
- `ReportUnidentifiedRemittanceTQueryPageReq.java`

---

## 3. 大改动文件 TOP 20 (按删除行数排序)

| 删除 | 新增 | 文件 |
|------|------|------|
| 898 | 2119 | `AccountServiceImpl.java` |
| 426 | 49 | `DaySumAmtServiceImpl.java` |
| 417 | 873 | `ConsumeTransAfter.java` |
| 262 | 166 | `TransferRecallJobService.java` |
| 242 | 241 | `SignAuthInterceptor.java` |
| 210 | 885 | `ConsumeTransRefundAfter.java` |
| 190 | 675 | `ConsumeTransRefundModel1After.java` |
| 131 | 262 | `TransRecallJobService.java` |
| 123 | 573 | `TransRecallServiceImpl.java` |
| 119 | 2 | `CheckUtil.java` |
| 110 | 35 | `logback-spring.xml` (base) |
| 108 | 34 | `logback-spring.xml` (web/task/front) |
| 96 | 5 | `ConsumeOrderUtil.java` |
| 94 | 167 | `ContractAndBillServiceImpl.java` |
| 93 | 485 | `BasContractInfoMapper.xml` |
| 92 | 147 | `BillJobService.java` |
| 66 | 170 | `TransferTransAfter.java` |
| 57 | 769 | `TransTransferTiBatchBusinessServiceImpl.java` |

---

## 4. 核心交易组件变更 (Trans 目录)

**统计**: 40 个文件，新增 4004 行，删除 1765 行

### 关键变更文件

| 文件 | 变更说明 |
|------|----------|
| `ConsumeTransAfter.java` | +873/-417 大重构 |
| `ConsumeTransRefundAfter.java` | +885/-210 大重构 |
| `ConsumeTransRefundModel1After.java` | +675/-190 大重构 |
| `TransferTransAfter.java` | +170/-66 |
| `TransferTrans.java` | +69 行 |
| `TransferTransAuth.java` | +44 行 (新增文件?) |
| `RefundRechargeTransAfter.java` | +349/-302 大改动 |
| `RechargeTransAfter.java` | +252 行 |
| `WithDrawTrans.java` | +130 行 |
| `RechargeTrans.java` | -74 行 (删除逻辑) |
| `ConsumeTransRefundSplitPack.java` | -546 行 (删除文件) |

---

## 5. 需要特别关注的变更

### 5.1 账户变动相关 (符合 PROJECT_MEMORY.md 排查规则)

**检查项**:
- [ ] `ConsumeTransAfter` 大重构是否正确调用 `BaseAccountServiceApi`
- [ ] `TransferTransAfter` 变更是否符合渠道失败处理规则
- [ ] `RefundRechargeTransAfter` 变更是否保持事务一致性
- [ ] `WithDrawTrans` 变更是否正确处理冻结/解冻

### 5.2 删除的文件影响

**需确认**:
1. `ConsumeTransRefundSplitPack.java` 删除 (-546行) - 功能是否已迁移?
2. `WsBankServiceApi.java` 删除 - 是否有替代实现?
3. `MybankConstants.java` 删除 (-319行) - 网商银行相关功能是否废弃?

### 5.3 大规模代码重构

**AccountServiceImpl.java** (+2119/-898):
- 大量代码重构，需确认业务逻辑正确性
- 可能涉及账户开立/查询核心逻辑

---

## 6. 建议下一步行动

1. **逐模块 Review**: 按 fund-catering-consume → fund-catering-base → fund-catering-task 顺序
2. **重点关注 Trans 组件**: 账户变动 API 调用是否统一
3. **验证删除文件**: 确认删除的类是否有引用残留
4. **运行测试**: 确保核心交易流程正常

---

## 7. 只有空格/回车变化的文件 (728个) - 可忽略

> 这些文件只有格式变化（空格、回车、缩进），无实际代码改动，可以 git checkout 恢复

### 按目录统计

| 目录 | 文件数 |
|------|--------|
| fund-catering-base/request | 87 |
| fund-catering-base/mapper | 55 |
| fund-catering-base/response | 52 |
| fund-catering-base/service | 33 |
| fund-catering-base/domain | 33 |
| fund-catering-base/service/impl | 27 |
| fund-catering-front/response | 23 |
| fund-catering-web/request/base | 22 |
| fund-catering-front/request | 20 |
| fund-catering-web/response/base | 19 |
| fund-catering-front/request/zxsaas | 20 |
| fund-catering-front/request/pasass | 18 |
| fund-catering-web/request/trans | 16 |
| fund-catering-web/validate | 12 |
| fund-catering-base/controller | 12 |
| starter-redis/cache | 10 |
| fund-catering-web/request/query | 10 |
| fund-catering-base/api | 10 |
| fund-catering-web/response/query | 9 |
| 其他 | ~240 |

### 7. 回退记录 (2026-03-20)

**已回退**: 728 个只有空格/回车变化的文件

**回退前状态**:
- 修改文件: 1251 个
- 新增文件: 229 个

**回退后状态**:
- 修改文件: 523 个
- 新增文件: 229 个

---

**生成时间**: 2026-03-20
**分析工具**: git diff --ignore-all-space
**回退时间**: 2026-03-20
