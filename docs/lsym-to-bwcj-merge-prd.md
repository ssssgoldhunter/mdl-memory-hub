# lsym 到 bwcj 代码合并需求文档 (PRD)

## 项目背景

两个项目基于同一框架代码衍生，项目代码结构相似，现在需要将 lsym 项目的代码合并到 bwcj 项目。

## 项目信息

| 项目 | 名称 | 路径 |
|------|------|------|
| 源项目 | lsym | `/Users/limeng/workspaces/IdeaProjects_lsym_uat/slhy` |
| 目标项目 | bwcj | `/Users/limeng/workspaces/IdeaProjects_mdl_dep/bwcj` |

## 合并范围

合并的业务代码为 **catering 供应链代码**，涉及以下模块：

| 模块 | 差异文件数 |
|------|------------|
| common-core | 299 |
| starter-redis | 29 |
| fund-catering-base | 120 (api + service) |
| fund-catering-consume | 558 (api + service) |
| fund-catering-task | 39 |
| fund-catering-report (仅 catering) | 306 |
| fund-catering-web | 112 |
| fund-catering-front | 0 |
| **总计** | **1463** |

## 合并约束

### 禁止操作
1. **databatch 项目及 databatch 清结算内容**：一点都不要修改！
2. **report 模块**：只能调整 catering 下的代码
3. **无效代码**：只有空格、特殊符号、回车的不需要合并
4. **不要为了 mvn install 乱改代码**

### 合并规则
1. **配置文件**：项目配置基于 bwcj（application-local 等）
2. **pom.xml**：基于 bwcj，注意路径变动
3. **冲突处理**：需要先比较两边代码提交记录
4. **历史对比**：bwcj 的很多供应链代码是 12月15日之前的老代码，需要注意比较历史提交

## 合并流程

1. 分析文件差异（新增/修改/删除）
2. 检查两边提交历史
3. 决定合并策略
4. 执行合并
5. 验证编译

## 重要提醒

- bwcj_prod 基线日期：2025-12-15
- 不要假设不存在的提交、不存在的代码
- 宁愿有报错，也要仔细分析原因