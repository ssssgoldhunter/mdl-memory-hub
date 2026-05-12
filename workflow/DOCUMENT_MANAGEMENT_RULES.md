# 工作偏好：文档和记忆体管理规则

> **重要规则**: 除非用户直接要求，所有 md 文档和记忆体内容都应存放在 mdl-memory-hub 记忆体项目中

## 📋 核心规则

### 文档存储规则

| 类型 | 存储位置 | 说明 |
|------|----------|------|
| **md 文档** | `mdl-memory-hub/` | 所有 Markdown 文档都放在记忆体项目 |
| **记忆体内容** | `mdl-memory-hub/` | AI 助手的学习内容、项目知识等 |
| **项目文档** | `mdl-memory-hub/docs/` | 项目的核心设计文档 |
| **快速参考** | `mdl-memory-hub/business-flows/` | 业务流程快速参考 |
| **技术文档** | `mdl-memory-hub/architecture/` | 架构设计文档 |
| **工作流程** | `mdl-memory-hub/workflow/` | 工作流程和偏好设置 |
| **主题索引** | `mdl-memory-hub/topics/` | 高频问题域的当前口径和源码入口 |
| **LLM入口** | `mdl-memory-hub/llms.txt` | AI/Agent 首读导航 |

### 不放在记忆体的内容

| 类型 | 存储位置 | 说明 |
|------|----------|------|
| **源代码** | `mdl/` 项目目录 | 代码文件放在项目中 |
| **配置文件** | `mdl/` 项目目录 | 配置文件放在项目中 |
| **资源文件** | `mdl/` 项目目录 | 静态资源放在项目中 |
| **编译产物** | `mdl/` 项目目录 | 编译生成的文件 |

## 🎯 使用场景

### 多台电脑协作

1. **电脑 A** 工作
   - 拉取最新的 mdl-memory-hub 仓库
   - 查看记忆体中的项目信息
   - 继续工作

2. **电脑 B** 工作
   - 拉取最新的 mdl-memory-hub 仓库
   - 获取电脑 A 更新的记忆体内容
   - 继续工作

3. **同步流程**
   ```
   工作 → 更新记忆体 → git push
   ↓
   切换电脑 → git pull → 获取最新记忆体
   ↓
   继续工作
   ```

## 📝 记忆体更新时机

AI 助手会在以下情况更新记忆体：

- ✅ 发现重要的项目信息
- ✅ 学习新的业务流程
- ✅ 记录技术决策
- ✅ 总结常见问题解决方案
- ✅ 更新文档版本
- ✅ 用户明确要求保存内容

## 🏷️ 文档状态标记

用于帮助 AI 区分当前口径、历史资料和待复核内容。新建或更新重要文档时，建议在标题后或开头补充状态说明。

| 状态 | 含义 | 使用方式 |
|------|------|----------|
| `current` | 当前有效口径 | 可作为当前讨论和开发依据，但关键结论仍需按需查源码 |
| `verified-against-source` | 已对照源码 | 适合主题页、审计报告、源码映射 |
| `historical` | 历史资料 | 保留追溯价值，不应直接作为当前实现依据 |
| `needs-source-check` | 待源码复核 | 使用前必须查当前源码、POM 或配置 |

### 主题页维护规则

- 高频、跨目录、会反复被问到的问题放入 `topics/`。
- 主题页只收敛当前结论、关键源码入口和相关文档链接。
- 历史过程仍放在 `conversation-logs/`、`requirements/` 或 `docs/` 中。
- 如果主题页结论和源码不一致，以源码为准，并在主题页顶部标记 `needs-source-check`。

## 🔧 操作流程

### 保存文档到记忆体

```powershell
cd D:\workspaces\IdeaProjects_mdl_dep\mdl-memory-hub

# 编辑或添加文档后查看状态
git status --short

# 提交到 Git
git add .
git commit -m "docs: add new document"
git push
```

### 从记忆体获取信息

```powershell
cd D:\workspaces\IdeaProjects_mdl_dep\mdl-memory-hub

# 拉取最新内容
git pull

# 查看文档
Get-Content docs/TRANSACTION_QUICK_REFERENCE.md -Encoding UTF8
```

## 📂 记忆体目录结构

```
mdl-memory-hub/
├── CLAUDE.md                # AI 工作配置
├── llms.txt                 # LLM/Agent 入口导航
├── README.md                # 入口说明
├── architecture/            # 技术架构文档
├── bugs/                    # 问题记录与修复备忘
├── business-flows/          # 业务流程文档
├── conversation-logs/       # 历史会话记录
├── docs/                    # 所有设计文档和参考文档
├── modules/                 # 模块说明文档
├── requirements/            # 需求文档
├── skills/                  # 项目专项技能说明
├── topics/                  # 高频主题入口与当前口径
├── technical-decisions/     # 技术决策记录
└── workflow/                # 工作流程和偏好
    ├── DOCUMENT_MANAGEMENT_RULES.md
    ├── PROJECT_MEMORY.md
    └── MEMORY_CODE_AUDIT_2026-04-27.md
```

## 🔄 与项目的关系

```
mdl/                           mdl-memory-hub/
├── fund-catering-*           ├── docs/           (从旧项目迁移并持续维护的文档)
│   ├── fund-catering-        ├── architecture/    (技术架构)
│   ├── fund-catering-front/   ├── business-flows/  (业务流程)
│   └── ...                   ├── workflow/       (工作偏好)
└── ...                       └── ...
```

## 🚫 例外情况

只有用户明确要求时，才将内容放在其他位置：

- 用户要求：`把这个文档放在项目目录下`
- 用户要求：`在项目中创建这个文件`
- 特殊需求：某些文档必须和代码放在一起

---

**生效日期**: 2026-03-03
**更新原因**: 支持多台电脑协作，统一文档管理
**GitHub**: https://github.com/ssssgoldhunter/mdl-memory-hub
