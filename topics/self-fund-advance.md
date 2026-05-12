# Self Fund Advance

> Status: current, verified-against-source
> Last updated: 2026-05-12

This page is the current entry point for 自有资金池垫资.

## Current Business Split

1. 资金充值
   - 沿用当前充值通知和 task 补偿链路。
   - 通过银行电子账号/银行卡号匹配企业和 `cardCode` 后走充值。
   - 自有资金池阶段 1 的 `03` 是平台/中信侧入金类型口径，不是新增独立交易。
   - 普通入金最终走 `TransConsumeApi.rechargeTrans(...)`。
   - 未显式设置 `IC` 时，`ConsumeRechargeRequest.transType` 默认是 `C`。

2. 清结算垫资
   - 清结算服务调用 consume Feign 内部转账接口。
   - 银行侧资金动作由银行完成，系统侧同步内部账户记账。
   - Consume Feign 接口：`/consume/trans/transTransferInner`。
   - 入参金额单位：分，整数字符串。
   - 接口交接文档：`docs/SELF_FUND_ADVANCE_CONSUME_API.md`。

3. 垫资扣款
   - 使用新的平台收款账号完成平台自有资金账户收款。
   - Consume Feign 接口：`/consume/trans/transPlatformReceive`。
   - `transPlatformDeduction` 当前复用平台收款。
   - 入参金额单位：分，整数字符串。
   - 接口交接文档：`docs/SELF_FUND_ADVANCE_CONSUME_API.md`。

## Platform Receive / Pay Baseline

- 平台付款交易类型：`MC`
- 平台收款交易类型：`MR`
- Web 入口：
  - `/scPlatformPay`
  - `/scPlatformReceive`
  - `/scPlatformDeduction`
- Consume 入口：
  - `/consume/trans/transPlatformPay`
  - `/consume/trans/transPlatformReceive`
  - `/consume/trans/transPlatformDeduction`
- LiteFlow 链：
  - `chainPlatformPay`
  - `chainPlatformReceive`
- Front 入口：
  - `/front/trans/platformPay`
  - `/front/trans/platformReceive`
- 中信业务用途：
  - `2041`：平台付款
  - `2042`：平台收款

## Required Configuration

1. `bas_param_t`
   - `SELF_FUND_ACCOUNT_CONFIG`
   - JSON fields:
     - `cardcode`
     - `ebankid`
     - `registerType`, default `12`
     - `dealType`
     - `fundTp`

2. `bas_card_bin_business_t`
   - Self fund card BIN needs `MC` for platform pay.
   - Business payer card BIN needs `MR` for platform receive/deduction.
   - If missing, platform chain fails with "没有找到卡bin业务信息".

## Source Pointers

- Consume Feign:
  - `../mdl/fund-catering-consume/fund-catering-consume-api/src/main/java/com/chinaums/erp/slhy/catering/consume/api/TransConsumeApi.java`
- Platform receive/pay request:
  - `../mdl/fund-catering-consume/fund-catering-consume-api/src/main/java/com/chinaums/erp/slhy/catering/consume/request/ConsumeDeductionTransRequest.java`
- Internal transfer request:
  - `../mdl/fund-catering-consume/fund-catering-consume-api/src/main/java/com/chinaums/erp/slhy/catering/consume/request/ConsumeTransferRequest.java`
- Platform chain implementation:
  - `../mdl/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/flow/component/trans/platform/`
- Internal transfer implementation:
  - `../mdl/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/flow/component/trans/transfer/manage/`
- Platform config parser:
  - `../mdl/fund-catering-consume/fund-catering-consume-common/src/main/java/com/chinaums/erp/slhy/catering/consume/common/config/SelfFundAccountConfig.java`

## Related Docs

- `docs/SELF_FUND_ADVANCE_CONSUME_API.md`
- `workflow/PROJECT_MEMORY.md`
- `requirements/overview.md`
- `docs/NOTIFICATION_OBJECTS_FOR_RECON.md`
