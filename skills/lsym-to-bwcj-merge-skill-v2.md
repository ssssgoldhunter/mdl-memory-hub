# lsym 到 bwcj 代码合并技能

## 概述

本文档记录从 lsym 项目合并代码到 bwcj 项目的方法和经验。

## 项目信息

| 项目 | 路径 | 说明 |
|------|------|------|
| lsym_prod | `/Users/limeng/workspaces/IdeaProjects_lsym_uat/slhy` | 源项目 |
| bwcj | `/Users/limeng/workspaces/IdeaProjects_mdl_dep/bwcj` | 目标项目 |

## 合并约束

### 绝对禁止
1. **databatch 项目**：不能修改任何内容
2. **report 非 catering 部分**：不能修改

### 保留 bwcj 版本
1. 项目配置文件（application-local.yml 等）
2. pom.xml 配置
3. databatch 相关引用

## 合并流程

### 1. 准备工作
```bash
# 创建合并分支
git checkout bwcj_prod
git checkout -b bwcj_lsym_merge_{日期}

# 确认源项目路径正确
ls /Users/limeng/workspaces/IdeaProjects_lsym_uat/slhy/
```

### 2. 分析差异
```bash
# 分析模块差异
diff -rq /Users/limeng/workspaces/IdeaProjects_lsym_uat/slhy/{模块路径} ./{模块路径}

# 检查提交历史
cd /Users/limeng/workspaces/IdeaProjects_lsym_uat/slhy
git log lsym_prod --format="%h %ad %s" --date=short --after="2025-12-15" -- "{模块路径}"

cd /Users/limeng/workspaces/IdeaProjects_mdl_dep/bwcj
git log bwcj_prod --format="%h %ad %s" --date=short --after="2025-12-15" -- "{模块路径}"
```

### 3. 合并决策矩阵

| lsym 12/15后修改 | bwcj 12/15后修改 | 处理方式 |
|-----------------|-----------------|---------|
| 否 | 否 | 使用 lsym 版本 |
| 是 | 否 | 使用 lsym 版本 |
| 否 | 是 | 保留 bwcj 版本 |
| 是 | 是 | 需要手动合并 |

### 4. 执行合并
```bash
# 新增文件
cp /Users/limeng/workspaces/IdeaProjects_lsym_uat/slhy/{文件路径} ./{文件路径}

# 修改文件（确认安全后）
cp /Users/limeng/workspaces/IdeaProjects_lsym_uat/slhy/{文件路径} ./{文件路径}

# 删除文件（确认无引用后）
rm ./{文件路径}
```

### 5. 验证
```bash
# 编译检查
mvn compile -q

# 不要为了编译通过乱改代码！
```

## 经验教训

1. **必须验证源代码存在**：在声称"从 lsym 同步"前，必须在 lsym_prod 项目中验证代码存在
2. **不要虚构功能**：不能根据假设添加代码
3. **检查引用**：删除文件前必须检查全局引用
4. **不要为了编译通过乱改**：宁愿有报错，也要仔细分析原因

## 操作记录

### 2026-03-22 新合并任务

#### starter-redis 模块
- 状态：✅ 已完成
- 提交：dc0a2c963
- 文件变更：
  - 新增：RedissonIdUtils.java, RedissonIdUtilsConfig.java
  - 修改：RedisConfigProperties.java, RedissonAutoConfiguration.java, RedisService.java
  - lsym 12/15后提交：3个（退款4级表修复、门店信息回写优化、redis及分表提交）
  - bwcj 12/15后提交：无
  - 决策：使用 lsym 版本

#### common-core 模块
- 状态：✅ 已完成
- lsym 新增文件：13个 ✅ 已添加
  - AlertMessageTypeEnums.java
  - BillDataUtil.java
  - AsyncMultiSheetExport.java, SheetConfig.java
  - OrderUtilConfig.java
  - RedissonIdKey.java
  - InnerSqlException.java, MerchantErrCode.java, MerchantErrorException.java
  - OuterHttpException.java, PlatformErrCode.java, PlatformErrorException.java
  - JsonListTypeHandler.java
- bwcj 独有文件：23个（databatch 相关，保留）
- 重叠文件：已手动合并 ✅
  - DefaultResult.java: 合并 lsym front* 字段 + bwcj fail(message, code, model) 方法
  - ApiConstants.java: 合并 lsym 常量 + bwcj RESULT_DEALING 常量
  - DateUtils.java: 合并 lsym convertToDateTimeFormat() + bwcj parseTimestamp/formatTimestamp/addDays 方法
  - ValidateMsgConstant.java: 合并 lsym 验证消息 + bwcj 门店验证消息
  - CommonConstants.java: 使用 lsym 版本
  - ValidateConstant.java: 使用 lsym 版本

#### fund-catering-base 模块
- 状态：✅ 已完成
- 提交：2ce8ce9b4
- 新增文件：39个（来自 lsym）
  - API: BasBankInfoServiceApi, BasCardAccountManagedApi, NpkStoreServiceFacadeApi
  - Request/Response: 账户变动、提现白名单等
  - Controller/Service: 银行信息、账户管理、白名单等
- 修改文件：80个
- 保留文件：WsBankServiceApi.java（databatch 网商银行相关）
- lsym 12/15后提交：20+个
- bwcj 12/15后提交：无

#### fund-catering-consume 模块
- 状态：✅ 已完成
- 提交：b2dc7e42f
- 变更文件：559个
- 主要变更：账户变动核心逻辑重构、授信功能、白名单增强
- lsym 12/15后提交：20+个
- bwcj 12/15后提交：无

#### fund-catering-task 模块
- 状态：✅ 已完成
- 提交：9a0bbbc45
- 变更文件：39个
- 主要变更：定时任务、日切任务优化
- lsym 12/15后提交：多个
- bwcj 12/15后提交：无

#### fund-catering-web 模块
- 状态：✅ 已完成
- 提交：9a0bbbc45
- 变更文件：112个
- 主要变更：Web服务接口、授信功能
- lsym 12/15后提交：多个
- bwcj 12/15后提交：无

#### fund-catering-report 模块 (catering path)
- 状态：✅ 已完成
- 提交：9a0bbbc45
- 变更文件：约300个
- 主要变更：报表服务、账单、交易流水
- lsym 12/15后提交：多个
- bwcj 12/15后提交：无

#### fund-catering-front 模块
- 状态：✅ 已完成
- 提交：无变更
- 说明：lsym 和 bwcj 版本一致，无需合并

---

## 合并完成总结

**合并完成日期**: 2026-03-22

**合并提交记录**:
1. dc0a2c963 - starter-redis 模块
2. 43e335856 - common-core 模块
3. 2ce8ce9b4 - fund-catering-base 模块
4. b2dc7e42f - fund-catering-consume 模块
5. 9a0bbbc45 - fund-catering-task/web/report 模块

**保留的 bwcj 独有代码**:
1. ApiConstants.RESULT_DEALING - databatch 人工补单使用
2. DefaultResult.fail(message, code, model) - databatch 使用
3. DateUtils.parseTimestamp/formatTimestamp/addDays - databatch 使用
4. ValidateMsgConstant 门店验证消息 - databatch 使用
5. WsBankServiceApi.java - databatch 网商银行相关

**注意事项**:
1. 未进行 mvn compile 验证（网络原因无法连接私有仓库）
2. 建议在有网络环境下进行完整编译验证
3. databatch 相关代码已保留，不应影响现有功能