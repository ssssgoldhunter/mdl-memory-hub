# Topic Index

> Status: current
> Last updated: 2026-05-12

This directory is the LLM-facing topic layer for `mdl-memory-hub`.

Topic pages collect the current working context for one problem area and point to source files, design docs, requirements, bugs, and historical notes. They are not a replacement for source code. Use them to find the right starting point quickly, then verify current behavior in `../mdl/`.

## Current Topics

| Topic | File | Use When |
|------|------|----------|
| Self fund advance | `self-fund-advance.md` | 自有资金池、垫资、03 入金、平台收款/付款、清结算调用 consume Feign |
| Notification and reconciliation | `notification-recon.md` | B/D/银行实收通知、清结算通知、RocketMQ 到 front 的通知对象 |
| Account change | `account-change.md` | 账户余额更新、MAC/CAS、账户变动明细、task 旧路径排查 |

## How To Maintain

- Add one topic page when a subject appears repeatedly across requirements, bugs, conversation logs, and source references.
- Keep topic pages short enough to scan.
- Put stable conclusions near the top.
- Put historical links near the bottom.
- Mark uncertain or old claims with `needs-source-check` or `historical`.
- If a topic becomes high-frequency, add a pointer from `workflow/PROJECT_MEMORY.md`.
