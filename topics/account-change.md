# Account Change

> Status: current, verified-against-source
> Last updated: 2026-05-12

This page is the entry point for account balance updates, MAC/CAS, and account change detail consistency.

## Current Account Update Entry Points

The current stable API group is `BaseAccountServiceApi`:

- `batchChangeAccount` — generic batch account change
- `batchChangeAccountForRecharge` — recharge
- `batchChangeAccountForRefundRecharge` — refund recharge
- `batchChangeAccountForConsume` — consume/deduction
- `batchChangeAccountForRefundConsume` — consume refund

Consume main transaction paths are mostly connected to these APIs.

## High-Risk Areas

- Multiple updates to the same card/sub-account in one flow.
- Missing `BaseSlot.refreshCardSubAccount(...)` before a later MAC/CAS update.
- Mixed two-phase flow: account balance update in base, detail write in consume.
- Task module old paths that still call `updateCardSubAccount`.
- Recall/compensation flows that are not ordinary transaction after-handlers.

## Investigation Checklist

1. Identify transaction type.
2. Locate LiteFlow chain in:
   - `../mdl/fund-catering-consume/fund-catering-consume-service/src/main/resources/liteflow/consume.el.xml`
   - `../mdl/fund-catering-consume/fund-catering-consume-service/src/main/resources/liteflow/query.el.xml`
3. Find the `After` component or task after-service.
4. Check which account update entry is used.
5. Check whether account snapshot is refreshed before balance update.
6. Check whether detail and balance can diverge if one write fails.

## Current Task Module Watch List

- `PaWithDrawUpdateStatusAfterService`
  - Still an older path; prioritize if task account consistency is in scope.
- `TransferRecallServiceImpl`
  - Recall path; do not refactor as a normal transaction after-handler without reviewing compensation semantics.
- `ZxWithDrawUpdateStatusAfterService`
  - Uses batch account update but still has separated detail writing.
- `AccountEntryAfterService`
  - Async entry path remains two-phase.

## Source Pointers

- Current map:
  - `docs/ACCOUNT_CHANGE_SOURCE_MAP.md`
- Base account API:
  - `../mdl/fund-catering-base/fund-catering-base-api/src/main/java/com/chinaums/erp/slhy/catering/base/api/BaseAccountServiceApi.java`
- Consume slot refresh:
  - `../mdl/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/flow/slot/BaseSlot.java`
- Core constants:
  - `../mdl/common-core/src/main/java/com/chinaums/slhy/common/catering/constant/CommonConstants.java`

## Related Docs

- `docs/ACCOUNT_CHANGE_SOURCE_MAP.md`
- `technical-decisions/MAC_CONCURRENCY_FIX.md`
- `workflow/PROJECT_MEMORY.md`
