# lsym 项目合并到 bwcj 项目代码合并技能

## 概述

本文档记录从 lsym 项目合并代码到 bwcj 项目的方法和经验。

## 合并流程

### 1. 准备工作
- 确认源项目: lsym (项目1)
- 确认目标项目: bwcj (项目2)
- 创建合并分支: `bwcj_lsym_commit_{日期}`

- 埮始.bwcjignore 文件排除 target 目录、减少扫描时间

- 临时移除 lsym 项目的 target 目录

### 2. 文件分类处理

#### 新增文件 (B类)
- lsym 有， bwcj 没有的新文件
- **安全**: 直接提交
- 命令: `git add {文件路径}`

#### 删除文件 (A类)
- bwcj 有但 lsym 没有的文件
- **需确认**:
  1. 检查是否有引用: `grep -r "类名" --include="*.java" .`
  2. 无引用则安全删除

  3. 有引用则需进一步分析

#### 修改文件 (C类) - 重点
- 两个项目都有的文件，差异需要逐个 review
- **按风险等级分类**:
  | 删除行数 | 风险 | 处理方式 |
  |--------|------|------|
  | >200 | 🔴 高 | 逐个对比 Java 代码 |
  | 100-200 | 🟡 中 | 检查字段是否有使用 |
  | <100 | 🟢 低 | 可直接使用 lsym 版本 |

### 3. Mapper XML 文件处理

#### 检查方法
```bash
# 查看差异统计
git diff --numstat bwcj_prod -- "**Mapper.xml" | awk '{print $2"\t"$1"\t"$3}' | sort -rn

# 查看具体差异
git diff bwcj_prod -- "xxxMapper.xml"
```

#### 关键检查点
1. **被删除的字段** → 检查 Java 代码中是否存在该字段
2. **被删除的 SQL** → 检查是否有调用
3. **新增的字段** → 确认 Java 代码中有对应属性

#### 示例: GdWarningMsgMapper.xml
- **问题**: lsym 版本删除了 `msg_type` 字段
- **检查**: Java 代码中 `GdWarningMsg.java` 和 `GdWarningMsgReq.java` 都有 `msgType` 属性
- **解决**: 保留 `msg_type` 字段映射

### 4. 配置文件处理
- **pom.xml**: 通常保留目标项目版本（避免依赖冲突）
- **bootstrap.yml**: 保留目标项目版本（环境配置）
- **logback-spring.xml**: 根据需要选择（自定义 vs 简化）
- **application-local.yml**: ⚠️ **绝对不能删除**，这是本地启动必需的配置文件！

### 5. 提交规范
```bash
git commit -m "$(cat <<'EOF'
feat: limeng lsym merge - {模块名}合并 ({文件数}个)

从 lsym 项目迁移的{描述}：
- {模块1}: {文件数}个文件
- {模块2}: {文件数}个文件

特殊处理:
- {特殊情况说明}

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
EOF
)"
```

## 常用命令

```bash
# 查看模块提交历史
git log bwcj_prod --format="%h %ad %s" --date=short -- "模块路径" | head -10

# 按模块统计修改文件
git status --short | grep "模块名" | wc -l

# 挌删除行数排序
git diff --numstat | awk '$2>100 {count++} END {print count, $0}' | sort -rn

# 检查字段使用
grep -rn "字段名" --include="*.java" . | grep -v "target/"

# 检查 Mapper XML 差异
git diff --numstat bwcj_prod -- "**Mapper.xml"
```

### 6. LiteFlow 配置文件处理
- 检查新增的链路和组件
- **关键检查**: 确认 Java 代码中有对应的 `@Component("组件名")` 注解
- 示例: `consumeTransRefundSplitSwitch` → 检查 `ConsumeTransRefundSplitSwitch.java`

### 7. 一边修改一边查询的方法
- 发现差异后立即检查 Java 代码
- 确认安全后立即 `git add`
- 按模块批量处理，提高效率

## 经验总结

1. **先看提交历史**: 了解目标模块最后修改时间，判断是否需要保留
2. **Java 代码优先**: 先合并 Java 代码，再处理 XML 和配置
3. **XML 后处理**: Mapper XML 需要对比 Java 代码确认字段是否存在
4. **配置文件保留**: pom.xml、bootstrap.yml 通常保留目标项目版本
5. **高风险文件**: 删除>100行的文件需要逐个检查
6. **临时提交**: 如果有最近的临时提交，需要单独校验
7. **边查边改**: 发现差异立即检查 Java 代码，确认后立即暂存
8. **LiteFlow 配置**: 检查组件是否存在，避免运行时错误
9. **批量处理**: 只有新增的文件可以批量添加，有删除的需逐个确认
10. **application-local.yml**: ⚠️ 绝对不能删除！这是本地启动必需的配置文件

## 教训记录

### application-local.yml 误删事件
- **问题**: 之前将 application-local.yml 当作废弃文件删除
- **原因**: 没有意识到这是本地启动必需的配置文件
- **影响**: 项目无法在本地环境启动
- **解决**: 从 bwcj_prod 恢复所有 application-local.yml 文件
- **教训**: 配置文件删除前必须确认是否影响项目启动

## 实战案例

### fund-catering-base 模块
- Java 代码: 67个文件，直接使用 lsym 版本
- Mapper XML: 9个，其中 GdWarningMsgMapper.xml 需保留 msg_type 字段
- 配置文件: 保留 bwcj_prod 版本

### fund-catering-consume 模块
- Java 代码: 152个文件，直接使用 lsym 版本
- Mapper XML: 13个，全部检查 Java 字段后使用 lsym 版本
- LiteFlow 配置: 1个，检查组件后使用 lsym 版本
- 配置文件: 保留 bwcj_prod 版本

### fund-catering-task 模块 ✅ 已完成
- Java 代码: 31个文件，使用 lsym 版本
- 配置文件: 使用 bwcj_prod 版本（pom.xml、bootstrap.yml、logback-spring.xml）
- 最后修改: 2025-12-22，无3月提交

### fund-catering-front 模块 ✅ 已完成
- Java 代码: 32个文件，使用 lsym 版本
- 配置文件: 使用 bwcj_prod 版本（4个pom.xml、bootstrap.yml、logback-spring.xml）
- 删除文件: WebUtils.java（无引用）
- 特殊检查: BankPagedListResultDto.java 3月2日 Revert 已包含，版本一致
- 最后修改: 2025-12-11，无3月提交

### starter-modules 模块 ✅ 已完成
- Java 代码: 3个文件，使用 lsym 版本
- 新增文件: 2个（RedissonIdUtils.java、RedissonIdUtilsConfig.java）
- 重要: RedissonIdUtils 被 consume 模块依赖，必须合并
- 最后修改: 2025-12-03，无3月提交

## 任务进度 (2026-03-21)

### ✅ 已完成模块
| 模块 | 文件数 | 处理方式 |
|------|--------|----------|
| fund-catering-base | 67+9 | Java/Mapper XML 使用 lsym，配置用 bwcj |
| fund-catering-consume | 152+13 | Java/Mapper XML 使用 lsym，配置用 bwcj |
| fund-catering-task | 33 | Java 使用 lsym，配置用 bwcj |
| fund-catering-front | 33 | Java 使用 lsym，配置用 bwcj |
| starter-modules | 5 | 全部使用 lsym（含依赖） |
| fund-catering-report | 94 | 已审查，中风险文件无问题 |
| fund-catering-web | 73 | 已审查，已恢复 bwcj 新代码 |
| common-core | 26 | 已审查，中风险文件无问题 |

### ✅ 审查完成 (2026-03-21)

#### 发现的问题及修复

**问题1：bwcj 新代码被删除（8个文件）**

| 文件 | bwcj 提交内容 | 处理结果 |
|------|--------------|----------|
| ZtBatchDateController.java | 6.3接口、手动触发定时任务、busOrdNo | ✅ 已恢复 |
| TzMonthBatchDateController.java | 6.7/6.8卡号解密、公司间调账 | ✅ 已恢复 |
| ZtDataController.java | busOrdNo、分账冻结/扣款 | ✅ 已恢复 |
| TestBatchNotifyMainController.java | 6.7/6.8接口 | ✅ 已恢复 |
| BaseBillDetailResponse.java | 新增 busOrdNo 字段 | ✅ 已恢复 |
| LedgerFreezeDebitResponse.java | 新增 busOrdNo 字段 | ✅ 已恢复 |
| SettleInAccResponse.java | 6.1/6.3接口 | ✅ 已恢复 |
| StoreJobService.java | 定时任务 | ✅ 已恢复 |

**依赖文件恢复（2个）**：
- PageResultResp.java
- PageResultDetailsResp.java

**提交记录**: `a54b87419 fix: 恢复 bwcj 12月15日后新增的文件 (10个)`

#### 中风险文件检查结果

| 文件 | bwcj 12月15日后提交 | 检查结果 |
|------|-------------------|----------|
| DefaultResult.java | Revert JSON序列化配置 | 🟢 删除的方法无引用，安全 |
| PagedListResultDto.java | Revert JSON序列化配置 | 🟢 构造函数调整，安全 |
| ApiConstants.java | 人工补单开发 | 🟢 新增常量，安全 |
| CommonConstants.java | 履历相关修改 | 🟢 格式调整+新增，安全 |
| BaseAccountServiceApi.java | fix: 修改sql | 🟢 删除方法无调用，安全 |
| BaseAccountFacadeController.java | fix: 修改sql | 🟢 删除方法无调用，安全 |

#### 删除文件全局引用检查结果

**无引用，安全删除**：
- SettlementMethodEnum
- SettlementModeEnum
- WsBankServiceApi
- WsChannelConfig 等 WS 相关类 (6个)
- StoreJobService
- StoreController
- PageResultResp、PageResultDetailsResp（后因依赖恢复）

### ⏳ 待处理模块
| 模块 | 文件数 | bwcj 3月提交 | 注意事项 |
|------|--------|--------------|----------|
| fund-catering-report | 94 | ✅ 有（3月11-18日） | 需仔细核对3月提交 |
| fund-catering-web | 73 | ✅ 有（3月4-5日） | 需仔细核对3月提交 |

### ✅ common-core 模块已完成 (2026-03-21)

#### 修复内容
| 文件 | 问题 | 处理结果 |
|------|------|----------|
| ContractConstants.java | 合并时常量值错误 | ✅ 使用 lsym 版本覆盖 |
| CreateNoUtil.java | 缺少 createContractNo 方法 | ✅ 添加方法 |
| Constant.java | 缺少 VOID_PAYMENT 相关常量 | ✅ 添加常量 |
| QueryOrderResponse.java | 缺少 realStatus 字段 | ✅ 添加字段 |
| ExcelUtil.java | 缺少必填字段和正则验证逻辑 | ✅ 添加验证逻辑 |

#### ContractConstants.java 冲突分析
- **CONTRACT_TYPE 冲突**:
  - lsym: TYPE_0="0"(常规), TYPE_1="1"(不放款)
  - bwcj 原: TYPE_1="1"(常规合同), TYPE_2="2"(不放款合同)
  - 代码逻辑使用 TYPE_1 表示"不放款"，故 bwcj 版本错误
- **CONTRACT_STATUS_F 冲突**:
  - lsym: "F" = 放款失败
  - bwcj 原: "F" = 失效
  - 代码逻辑用于"放款失败"场景，故 bwcj 版本错误
- **历史分析**: bwcj_prod 原始版本没有这些常量，是合并时错误添加

### ✅ fund-catering-base 模块已完成 (2026-03-21)

#### 修复内容
| 文件 | 问题 | 处理结果 |
|------|------|----------|
| AccountServiceImpl.java | 缺少银行返回报错信息优化 | ✅ 使用 lsym 版本覆盖 |
| BasContractInfoReq.java | - | 🟢 bwcj 已有 updateBy/updateTime 字段 |
| GdWarningMsgReq.java | - | 🟢 bwcj 已有 msgType 字段 |
| GdWarningMsg.java | - | 🟢 bwcj 已有 msgType 字段 |

#### AccountServiceImpl.java 更新说明
- lsym 在 2026-03-18 更新：账户类对接银行返回报错信息优化
- 将银行返回的错误信息（code, message, frontCode, frontMessage, frontTransSsn）透传给前端

### ✅ fund-catering-consume 模块已完成 (2026-03-21)

#### 修复内容
| 文件 | 问题 | 处理结果 |
|------|------|----------|
| CreditRepaymentDetailApi.java | 缺少下载接口 | ✅ 添加 downLoadRepaymentList 接口 |
| CreditRepaymentDetailController.java | 缺少下载实现 | ✅ 添加方法实现 |
| CreditRepaymentDetailService.java | 缺少方法声明 | ✅ 添加方法声明 |
| CreditRepaymentDetailServiceImpl.java | 缺少方法实现 | ✅ 添加方法实现 |
| CreditRepaymentDetailMapper.java | 缺少 Mapper 方法 | ✅ 添加方法 |
| CreditRepaymentDetailMapper.xml | 缺少 SQL | ✅ 添加 SQL |
| CreditBillDetailController.java | - | 🟢 已恢复 lsym 原版 |
| RefundRechargeTransAfter.java | import 差异 | 🟢 仅 import 差异，忽略 |
| ConsumeTransFrozen.java | import 差异 | 🟢 仅 import 差异，忽略 |
| TransTransferTiBatchBusinessServiceImpl.java | import 差异 | 🟢 仅 import 差异，忽略 |

### ✅ fund-catering-web 模块已完成 (2026-03-22)

#### 提交信息
- **Commit**: 849dcf748
- **文件数**: 82个

#### 合并内容
- controller: 商户号转换逻辑、前端错误码处理
- config: SignAuthInterceptor完整代码、BaseCheckInterceptor动态商户ID
- request/response: 82个文件从lsym复制

#### 特殊处理
- 保留bwcj独有文件: TzMonthBatchDateController、ZtBatchDateController、TestBatchNotifyMainController、ScStoreSyncReq
- WebConstant.java: 添加缺失的C_REMARK常量

### ✅ fund-catering-report 模块已完成 (之前合并)
- 已在之前的合并提交中处理（commit: 6936297a1, cffd78865, 1b6b1e69e）
- 中风险文件已检查，无问题

### ✅ starter-redis 模块已完成 (2026-03-22)
- 无差异，无需处理

---

## 任务完成总结 (2026-03-22)

### 所有模块合并状态

| 模块 | 文件数 | 状态 | 最终提交 |
|------|--------|------|----------|
| common-core | 26 | ✅ 已完成 | 312c44255 |
| fund-catering-base | 67+9 | ✅ 已完成 | 312c44255 |
| fund-catering-consume | 152+13 | ✅ 已完成 | 312c44255 |
| fund-catering-task | 33 | ✅ 已完成 | 之前合并 |
| fund-catering-front | 33 | ✅ 已完成 | 之前合并 |
| starter-modules | 5 | ✅ 已完成 | 之前合并 |
| fund-catering-report | 118 | ✅ 已完成 | 之前合并 |
| fund-catering-web | 82 | ✅ 已完成 | 9db315eff |
| starter-redis | 0 | ✅ 无差异 | - |

**最终统计**: 715 文件修改, +42,685 行, -7,302 行

### 合并分支信息
- **分支名**: `bwcj_lsym_commit_20260320`
- **基线分支**: `bwcj_prod`
- **基线日期**: 2025-12-15

### 关键修复记录
1. bwcj 12月15日后新增代码被删除 → 已恢复 (commit: a54b87419)
2. ContractConstants.java 常量缺失 → ✅ 已从 lsym_prod 恢复 (commit: 312c44255)
3. CreditRepaymentDetail 下载功能缺失 → ✅ 已从 lsym_prod 恢复 (commit: 312c44255)
4. AccountServiceImpl 银行返回报错信息优化 → ✅ 已从 lsym_prod 恢复 (commit: 312c44255)
5. WebConstant.java C_REMARK常量缺失 → 已添加 (commit: 849dcf748)
6. DefaultResult.java front* 字段 → ✅ 已确认存在 (commit: 312c44255)
7. BatchNotifyRequestRequest.java 被误删 → ✅ 已从 bwcj_prod 恢复 (commit: 9db315eff)

### ⚠️ 错误使用 lsym 目录问题 (2026-03-22 发现并修复)

**问题**：之前使用错误的 lsym 目录 `/Users/limeng/workspaces/IdeaProjects_lsym_dep/slhy` 进行比较，该目录是旧版本。

**正确的 lsym_prod 路径**：`/Users/limeng/workspaces/IdeaProjects_lsym_uat/slhy`

**导致的问题**：
- 误认为 lsym 不存在 front* 字段、Contract_Type 类等代码
- 错误地删除了这些真实存在的代码，声称是"虚构代码"
- 提交 f738c06bf, 70e3319bb, cbc4fe283, a4a2cb376, 4cf2227b5 都是错误的删除

**已修复 (commit: 312c44255)**：
从正确的 lsym_prod 路径恢复了以下代码：

| 文件 | lsym_prod 提交历史 | 内容 |
|------|-------------------|------|
| DefaultResult.java | 2026-01-04 "中信银行错误透传" | frontCode, frontMessage, frontTransSsn 字段 |
| ContractConstants.java | 2026-03-20 "授信1.21需求" | Contract_Type, Bill_Overdue_Reason 类 |
| AccountServiceImpl.java | 2026-03-18 "账户类对接银行返回报错信息优化" | 17处 setFrontCode 调用 |
| CreditRepaymentDetailMapper.xml | 2026-02-05 "授信-核销管理-下载" | downLoadRepaymentList 方法 |
| CreditRepaymentDetailQueryExportRes.java | 2026-02-05 | 新增导出响应类 |
| CreditRepaymentDetailService/Api/Controller | 2026-02-05 | 下载功能链 |

**教训**：
1. **必须使用正确的源项目路径**：lsym_prod 在 `/Users/limeng/workspaces/IdeaProjects_lsym_uat/slhy`
2. **验证代码存在**：在删除前，必须在正确的源项目中验证代码是否真实存在
3. **不要假设虚构**：看到不认识的代码时，先检查是否是源项目的新功能

### 📋 核对技巧
```bash
# 1. 查看模块完整提交历史（使用 --follow 追踪文件）
git log bwcj_prod --format="%h %ad %an %s" --date=short --follow -- "模块路径"

# 2. 查看12月15日以来的提交
git log bwcj_prod --format="%h %ad %s" --date=short --after="2025-12-15" -- "模块路径"

# 3. 检查具体文件是否在12月15日后有变更
git log bwcj_prod --oneline --after="2025-12-15" -- "文件路径"

# 4. 找出重叠文件（bwcj 12月15日后修改 && 当前合并也修改）
comm -12 <(git log bwcj_prod --format="" --name-only --after="2025-12-15" -- "模块路径" | sort -u) <(git diff bwcj_prod HEAD --name-only | sort -u)

# 5. 全局引用检查
grep -rn "类名" --include="*.java" /Users/limeng/workspaces/ | grep -v "target/" | grep -v ".git/"
```

## 审查要点总结

### 必须执行的检查

1. **重叠文件检查**: bwcj 12月15日后修改的文件，需要对比历史提交
2. **全局引用检查**: 删除类/方法前，必须在所有项目目录搜索引用
3. **依赖检查**: 恢复文件时，检查是否有依赖文件也需要恢复

### 检查范围

| 项目路径 | 说明 |
|---------|------|
| /Users/limeng/workspaces/IdeaProjects_mdl_dep/bwcj | 当前工作目录 |
| /Users/limeng/workspaces/IdeaProjects_bwcj_uat/ | bwcj UAT 环境 |
| **/Users/limeng/workspaces/IdeaProjects_lsym_uat/slhy** | **lsym_prod 源项目（正确路径）** |
| /Users/limeng/workspaces/IdeaProjects_lsym_dep/slhy | lsym 开发环境（可能过时） |

### 常见问题处理

| 问题 | 解决方案 |
|------|---------|
| bwcj 新代码被删除 | 从 bwcj_prod 恢复：`git checkout bwcj_prod -- "文件路径"` |
| 删除类有引用 | 检查引用是否在合并范围内，如在范围内则恢复 |
| 依赖文件缺失 | 恢复文件时检查 import，确保依赖文件也存在 |
