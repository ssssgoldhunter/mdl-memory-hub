# Notification And Reconciliation

> Status: current, verified-against-source
> Last updated: 2026-05-12

This page is the entry point for B/D/银行实收通知 and 清结算联调口径.

## Current Baseline

- B/D/银行实收仍保留 `RocketMQ -> front` 通知结构。
- Current front consumers for B/D/银行实收 already call 清结算 `resultNotifyApi`.
- 普通消费/充值等通知链仍兼容 `notifyUrl` 回调。
- 提现结果通知继续沿用现有回调链路。

## Topic List

| Business | Topic | DTO |
|----------|-------|-----|
| 扣款 | `mq_http_catering_deduction` | `ConsumeNotifyDto` |
| 划付 | `mq_http_catering_transfer` | `TransferNotifyDto` |
| 提现 | `mq_http_catering_withDraw` | `WithDrawNotifyDto` |
| 银行实收 | `mq_http_catering_actual_receipt` | `ActualReceiptNotifyDto` |

## Actual Receipt

- 银行实收入金充值类型使用 `IC`。
- 不明来款执行“银行渠道上账”成功后，按实收入金处理。
- Current source has two send paths:
  - `PlatformRechargeJobService`
  - `UnknownTransServiceImpl.processBankChannelEntry(...)`
- Both paths assemble notification with `transType=IC`.

## Source Pointers

- Notification object list:
  - `docs/NOTIFICATION_OBJECTS_FOR_RECON.md`
- Front DTOs:
  - `../mdl/fund-catering-front/fund-catering-front-api/src/main/java/com/chinaums/erp/slhy/catering/front/message/`
- Front consumers:
  - `../mdl/fund-catering-front/fund-catering-front-service/src/main/java/com/chinaums/erp/slhy/catering/front/handle/impl/message/`
- Actual receipt task sender:
  - `../mdl/fund-catering-task/src/main/java/com/chinaums/erp/slhy/catering/task/job/zx/PlatformRechargeJobService.java`
- Unknown remittance sender:
  - `../mdl/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/service/impl/zx/UnknownTransServiceImpl.java`

## Related Docs

- `docs/NOTIFICATION_OBJECTS_FOR_RECON.md`
- `docs/superpowers/mdl-supply-chain-abcd/00-current-baseline.md`
- `docs/superpowers/mdl-supply-chain-abcd/01-code-state.md`
- `requirements/overview.md`
