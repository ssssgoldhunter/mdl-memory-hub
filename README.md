# mdl-memory-hub

> mdl（麦当劳）项目记忆体仓库 - 用于保存 AI 助手在 mdl 项目工作过程中的记忆、知识和文档

---

## 📁 目录结构

```
mdl-memory-hub/
├── CLAUDE.md                    # AI 工作配置（必读）
├── README.md                    # 本文件
├── architecture/                # 架构设计文档
├── bugs/                        # 问题记录与修复备忘
├── business-flows/              # 业务流程文档
├── conversation-logs/           # 历史会话记录
├── docs/                        # 核心设计文档和参考文档
├── modules/                     # 模块文档
├── requirements/                # 需求文档
├── skills/                      # 项目专项技能说明
├── technical-decisions/         # 技术决策记录
└── workflow/                    # 工作流程与规范
    ├── DOCUMENT_MANAGEMENT_RULES.md      # 文档管理规则
    └── MEMORY_CODE_AUDIT_2026-04-27.md   # 记忆体与源码校验报告
```

---

## 🎯 项目信息

| 属性 | 值 |
|------|------|
| **项目名称** | mdl (麦当劳餐饮资金体系) |
| **负责人** | 李蒙 (ssssgoldhunter) |
| **主项目路径** | `D:\workspaces\IdeaProjects_mdl_dep\mdl` |
| **记忆库路径** | `D:\workspaces\IdeaProjects_mdl_dep\mdl-memory-hub` |
| **来源项目** | lsym (餐饮资金体系) - 供应链部分相同 |
| **飞书文档** | https://jvn4jogcy6u.feishu.cn |

**详细配置**: 请查看 [`CLAUDE.md`](./CLAUDE.md)

---

## 📚 核心文档速览

### 六大交易流程

| 流程 | 特点 | 快速查看 |
|------|------|----------|
| 消费 | 02膨胀金优先扣款，支持分账 | [快速参考 →](./docs/TRANSACTION_QUICK_REFERENCE.md) |
| 充值 | 支持01现金+02膨胀金赠送 | [快速参考 →](./docs/TRANSACTION_QUICK_REFERENCE.md) |
| 充值退款 | 原路退回，膨胀金收回 | [快速参考 →](./docs/TRANSACTION_QUICK_REFERENCE.md) |
| 提现 | 自动提现+人工审核 | [快速参考 →](./docs/TRANSACTION_QUICK_REFERENCE.md) |
| 转账 | 三层锁机制，支持批量 | [快速参考 →](./docs/TRANSACTION_QUICK_REFERENCE.md) |
| 消费退款 | 按比例/按单退款 | [快速参考 →](./docs/TRANSACTION_QUICK_REFERENCE.md) |

### 技术架构

- **流程编排**: LiteFlow
- **组件类型**: Pack → Check → Trans → After
- **上下文**: TransSlot（交易）、QuerySlot（查询）
- **详情**: [框架结构 →](./architecture/FRAMEWORK_STRUCTURE.md)

---

## 📝 更新记录

| 日期 | 更新内容 |
|------|----------|
| 2026-04-20 | 文档与代码状态同步：修正 B/D/front 清结算通知口径，更新 5 个 bug 状态为“已修复”，补齐需求文档元信息 |
| 2026-04-18 | 记忆体全面复核：BC-001 状态修正（未修复）、需求 B/C/D/E 标记完成、账户变动现状更新、TODO-TRIAGE 复核 |
| 2026-04-03 | 记忆体路径校正：主仓路径统一更新为 `mdl/`，并按当前代码结构修正文档导航 |
| 2026-03-19 | 项目迁移：从 lsym 迁移到 mdl（麦当劳），更新所有路径配置 |
| 2026-03-06 | 模块文档补充：新增 9 个模块文档，包括 base/front/management/task/data-batch/report/check/api/database |
| 2026-03-04 | 文档整理：删除 6 个重复文件，创建 CLAUDE.md |
| 2026-03-03 | 文档迁移：从 slhy/md 迁移到记忆库 |
| 2026-03-02 | 初始化记忆体仓库，补充交易流程详解 v5.5 |

---

## 📧 联系方式

- **GitHub**: https://github.com/ssssgoldhunter
