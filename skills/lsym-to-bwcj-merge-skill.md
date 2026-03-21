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
| common-core | 26 | ✅ 有（3月2日） | 需仔细核对3月提交 |

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
| /Users/limeng/workspaces/IdeaProjects_mdl_dep/ | 当前工作目录 |
| /Users/limeng/workspaces/IdeaProjects_bwcj_uat/ | bwcj UAT 环境 |
| /Users/limeng/workspaces/IdeaProjects_lsym_dep/ | lsym 项目 |
| /Users/limeng/workspaces/IdeaProjects_lsym_uat/ | lsym UAT 环境 |

### 常见问题处理

| 问题 | 解决方案 |
|------|---------|
| bwcj 新代码被删除 | 从 bwcj_prod 恢复：`git checkout bwcj_prod -- "文件路径"` |
| 删除类有引用 | 检查引用是否在合并范围内，如在范围内则恢复 |
| 依赖文件缺失 | 恢复文件时检查 import，确保依赖文件也存在 |
